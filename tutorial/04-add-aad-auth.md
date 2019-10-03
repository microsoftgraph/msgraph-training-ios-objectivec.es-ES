<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="17ce0-101">En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="17ce0-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="17ce0-102">Esto es necesario para obtener el token de acceso de OAuth necesario para llamar a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="17ce0-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="17ce0-103">Para ello, integrará la [biblioteca de autenticación de Microsoft (MSAL) para iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="17ce0-103">To do this, you will integrate the [Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) into the application.</span></span>

1. <span data-ttu-id="17ce0-104">Cree un nuevo archivo de **lista de propiedades** en el proyecto **GraphTutorial** denominado **AuthSettings. plist**.</span><span class="sxs-lookup"><span data-stu-id="17ce0-104">Create a new **Property List** file in the **GraphTutorial** project named **AuthSettings.plist**.</span></span>
1. <span data-ttu-id="17ce0-105">Agregue los siguientes elementos al archivo en el diccionario **raíz** .</span><span class="sxs-lookup"><span data-stu-id="17ce0-105">Add the following items to the file in the **Root** dictionary.</span></span>

    | <span data-ttu-id="17ce0-106">Key</span><span class="sxs-lookup"><span data-stu-id="17ce0-106">Key</span></span> | <span data-ttu-id="17ce0-107">Tipo</span><span class="sxs-lookup"><span data-stu-id="17ce0-107">Type</span></span> | <span data-ttu-id="17ce0-108">Valor</span><span class="sxs-lookup"><span data-stu-id="17ce0-108">Value</span></span> |
    |-----|------|-------|
    | `AppId` | <span data-ttu-id="17ce0-109">Cadena</span><span class="sxs-lookup"><span data-stu-id="17ce0-109">String</span></span> | <span data-ttu-id="17ce0-110">El identificador de la aplicación del portal de Azure</span><span class="sxs-lookup"><span data-stu-id="17ce0-110">The application ID from the Azure portal</span></span> |
    | `GraphScopes` | <span data-ttu-id="17ce0-111">Matriz</span><span class="sxs-lookup"><span data-stu-id="17ce0-111">Array</span></span> | <span data-ttu-id="17ce0-112">Dos valores de cadena `User.Read` : y`Calendars.Read`</span><span class="sxs-lookup"><span data-stu-id="17ce0-112">Two String values: `User.Read` and `Calendars.Read`</span></span> |

    ![Captura de pantalla del archivo AuthSettings. plist en Xcode](./images/auth-settings.png)

> [!IMPORTANT]
> <span data-ttu-id="17ce0-114">Si usa un control de código fuente como GIT, ahora sería un buen momento para excluir el archivo **AuthSettings. plist** del control de código fuente para evitar la pérdida inadvertida del identificador de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="17ce0-114">If you're using source control such as git, now would be a good time to exclude the **AuthSettings.plist** file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="17ce0-115">Implementar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="17ce0-115">Implement sign-in</span></span>

<span data-ttu-id="17ce0-116">En esta sección, configurará el proyecto para MSAL, creará una clase de administrador de autenticación y actualizará la aplicación para iniciar y cerrar sesión.</span><span class="sxs-lookup"><span data-stu-id="17ce0-116">In this section you will configure the project for MSAL, create an authentication manager class, and update the app to sign in and sign out.</span></span>

### <a name="configure-project-for-msal"></a><span data-ttu-id="17ce0-117">Configurar Project para MSAL</span><span class="sxs-lookup"><span data-stu-id="17ce0-117">Configure project for MSAL</span></span>

1. <span data-ttu-id="17ce0-118">Control haga clic en **info. plist** y seleccione **Abrir como**y, a continuación, **código fuente**.</span><span class="sxs-lookup"><span data-stu-id="17ce0-118">Control click **Info.plist** and select **Open As**, then **Source Code**.</span></span>
1. <span data-ttu-id="17ce0-119">Agregue lo siguiente dentro del `<dict>` elemento.</span><span class="sxs-lookup"><span data-stu-id="17ce0-119">Add the following inside the `<dict>` element.</span></span>

    ```xml
    <key>CFBundleURLTypes</key>
    <array>
      <dict>
        <key>CFBundleURLSchemes</key>
        <array>
          <string>msauth.$(PRODUCT_BUNDLE_IDENTIFIER)</string>
        </array>
      </dict>
    </array>
    <key>LSApplicationQueriesSchemes</key>
    <array>
        <string>msauthv2</string>
        <string>msauthv3</string>
    </array>
    ```

1. <span data-ttu-id="17ce0-120">Abra **AppDelegate. m** y agregue la siguiente instrucción Import en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="17ce0-120">Open **AppDelegate.m** and add the following import statement at the top of the file.</span></span>

    ```objc
    #import <MSAL/MSAL.h>
    ```

1. <span data-ttu-id="17ce0-121">Agregue la siguiente función a la clase `AppDelegate`.</span><span class="sxs-lookup"><span data-stu-id="17ce0-121">Add the following function to the `AppDelegate` class.</span></span>

    ```objc
    - (BOOL)application:(UIApplication *)app
                openURL:(NSURL *)url
                options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
    {
        return [MSALPublicClientApplication handleMSALResponse:url
                                             sourceApplication:options[UIApplicationOpenURLOptionsSourceApplicationKey]];
    }
    ```

### <a name="create-authentication-manager"></a><span data-ttu-id="17ce0-122">Crear el administrador de autenticación</span><span class="sxs-lookup"><span data-stu-id="17ce0-122">Create authentication manager</span></span>

1. <span data-ttu-id="17ce0-123">Cree una nueva **clase táctil de cacao** en el proyecto **GraphTutorial** denominado **AuthenticationManager**.</span><span class="sxs-lookup"><span data-stu-id="17ce0-123">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **AuthenticationManager**.</span></span> <span data-ttu-id="17ce0-124">Elija **NSObject** en el campo **subclase de** .</span><span class="sxs-lookup"><span data-stu-id="17ce0-124">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="17ce0-125">Abra **AuthenticationManager. h** y reemplace el contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="17ce0-125">Open **AuthenticationManager.h** and replace its contents with the following code.</span></span>

    ```objc
    #import <Foundation/Foundation.h>
    #import <MSAL/MSAL.h>

    NS_ASSUME_NONNULL_BEGIN

    typedef void (^GetTokenCompletionBlock)(NSString* _Nullable accessToken, NSError* _Nullable error);

    @interface AuthenticationManager : NSObject

    + (id) instance;
    - (void) getTokenInteractivelyWithCompletionBlock: (GetTokenCompletionBlock)completionBlock;
    - (void) getTokenSilentlyWithCompletionBlock: (GetTokenCompletionBlock)completionBlock;
    - (void) signOut;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. <span data-ttu-id="17ce0-126">Abra **AuthenticationManager. m** y reemplace el contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="17ce0-126">Open **AuthenticationManager.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "AuthenticationManager.h"

    @interface AuthenticationManager()

    @property NSString* appId;
    @property NSArray<NSString*>* graphScopes;
    @property MSALPublicClientApplication* publicClient;

    @end

    @implementation AuthenticationManager

    + (id) instance {
        static AuthenticationManager *singleInstance = nil;
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^ {
            singleInstance = [[self alloc] init];
        });

        return singleInstance;
    }

    - (id) init {
        if (self = [super init]) {
            // Get app ID and scopes from AuthSettings.plist
            NSString* authConfigPath =
            [NSBundle.mainBundle pathForResource:@"AuthSettings" ofType:@"plist"];
            NSString* bundleId = NSBundle.mainBundle.bundleIdentifier;
            NSDictionary* authConfig = [NSDictionary dictionaryWithContentsOfFile:authConfigPath];

            self.appId = authConfig[@"AppId"];
            self.graphScopes = authConfig[@"GraphScopes"];

            // Create the MSAL client
            self.publicClient = [[MSALPublicClientApplication alloc] initWithClientId:self.appId
                                                                        keychainGroup:bundleId
                                                                                error:nil];
        }

        return self;
    }

    - (void) getTokenInteractivelyWithCompletionBlock:(GetTokenCompletionBlock)completionBlock {
        // Call acquireToken to open a browser so the user can sign in
        [self.publicClient
         acquireTokenForScopes:self.graphScopes
         completionBlock:^(MSALResult * _Nullable result, NSError * _Nullable error) {

            // Check error
            if (error) {
                completionBlock(nil, error);
                return;
            }

            // Check result
            if (!result) {
                NSMutableDictionary* details = [NSMutableDictionary dictionary];
                [details setValue:@"No result was returned" forKey:NSDebugDescriptionErrorKey];
                completionBlock(nil, [NSError errorWithDomain:@"AuthenticationManager" code:0 userInfo:details]);
                return;
            }

            NSLog(@"Got token interactively: %@", result.accessToken);
            completionBlock(result.accessToken, nil);
        }];
    }

    - (void) getTokenSilentlyWithCompletionBlock:(GetTokenCompletionBlock)completionBlock {
        // Check if there is an account in the cache
        NSError* msalError;
        MSALAccount* account = [self.publicClient allAccounts:&msalError].firstObject;

        if (msalError || !account) {
            NSMutableDictionary* details = [NSMutableDictionary dictionary];
            [details setValue:@"Could not retrieve account from cache" forKey:NSDebugDescriptionErrorKey];
            completionBlock(nil, [NSError errorWithDomain:@"AuthenticationManager" code:0 userInfo:details]);
            return;
        }

        // Attempt to get token silently
        [self.publicClient
         acquireTokenSilentForScopes:self.graphScopes account:account
         completionBlock:^(MSALResult * _Nullable result, NSError * _Nullable error) {
             // Check error
             if (error) {
                 completionBlock(nil, error);
                 return;
             }

             // Check result
             if (!result) {
                 NSMutableDictionary* details = [NSMutableDictionary dictionary];
                 [details setValue:@"No result was returned" forKey:NSDebugDescriptionErrorKey];
                 completionBlock(nil, [NSError errorWithDomain:@"AuthenticationManager" code:0 userInfo:details]);
                 return;
             }

             NSLog(@"Got token silently: %@", result.accessToken);
             completionBlock(result.accessToken, nil);
         }];
    }

    - (void) signOut {
        NSError* msalError;
        NSArray* accounts = [self.publicClient allAccounts:&msalError];

        if (msalError) {
            NSLog(@"Error getting accounts from cache: %@", msalError.debugDescription);
            return;
        }

        for (id account in accounts) {
            [self.publicClient removeAccount:account error:nil];
        }
    }

    @end
    ```

### <a name="add-sign-in-and-sign-out"></a><span data-ttu-id="17ce0-127">Agregar Inicio y cierre de sesión</span><span class="sxs-lookup"><span data-stu-id="17ce0-127">Add sign-in and sign-out</span></span>

1. <span data-ttu-id="17ce0-128">Abra el archivo **SignInViewController. m** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="17ce0-128">Open the **SignInViewController.m** file and replace its contents with the following code.</span></span>

    ```objc
    #import "SignInViewController.h"
    #import "SpinnerViewController.h"
    #import "AuthenticationManager.h"

    @interface SignInViewController ()
    @property SpinnerViewController* spinner;
    @end

    @implementation SignInViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        self.spinner = [SpinnerViewController alloc];
        [self.spinner startWithContainer:self];

        [AuthenticationManager.instance
         getTokenSilentlyWithCompletionBlock:^(NSString * _Nullable accessToken, NSError * _Nullable error) {
             dispatch_async(dispatch_get_main_queue(), ^{
                 [self.spinner stop];

                 if (error || !accessToken) {
                     // If there is no token or if there's an error,
                     // no user is signed in, so stay here
                     return;
                 }

                 // Since we got a token, user is signed in
                 // Go to welcome page
                 [self performSegueWithIdentifier: @"userSignedIn" sender: nil];
             });
        }];
    }

    - (IBAction)signIn {
        self.spinner = [SpinnerViewController alloc];
        [self.spinner startWithContainer:self];

        [AuthenticationManager.instance
         getTokenInteractivelyWithCompletionBlock:^(NSString * _Nullable accessToken, NSError * _Nullable error) {
             dispatch_async(dispatch_get_main_queue(), ^{
                 [self.spinner stop];

                 if (error || !accessToken) {
                     // Show the error and stay on the sign-in page
                     UIAlertController* alert = [UIAlertController
                                                 alertControllerWithTitle:@"Error signing in"
                                                 message:error.debugDescription
                                                 preferredStyle:UIAlertControllerStyleAlert];

                     UIAlertAction* okButton = [UIAlertAction
                                                actionWithTitle:@"OK"
                                                style:UIAlertActionStyleDefault
                                                handler:nil];

                     [alert addAction:okButton];
                     [self presentViewController:alert animated:true completion:nil];
                     return;
                 }

                 // Since we got a token, user is signed in
                 // Go to welcome page
                 [self performSegueWithIdentifier: @"userSignedIn" sender: nil];
             });
         }];
    }
    @end
    ```

1. <span data-ttu-id="17ce0-129">Abra **WelcomeViewController. m** y reemplace la función `signOut` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="17ce0-129">Open **WelcomeViewController.m** and replace the existing `signOut` function with the following.</span></span>

    ```objc
    - (IBAction)signOut {
        [AuthenticationManager.instance signOut];
        [self performSegueWithIdentifier: @"userSignedOut" sender: nil];
    }
    ```

1. <span data-ttu-id="17ce0-130">Guarde los cambios y reinicie la aplicación en el simulador.</span><span class="sxs-lookup"><span data-stu-id="17ce0-130">Save your changes and restart the application in Simulator.</span></span>

<span data-ttu-id="17ce0-131">Si inicia sesión en la aplicación, debería ver un token de acceso que se muestra en la ventana de salida de Xcode.</span><span class="sxs-lookup"><span data-stu-id="17ce0-131">If you sign in to the app, you should see an access token displayed in the output window in Xcode.</span></span>

![Captura de pantalla de la ventana de salida de Xcode que muestra un token de acceso](./images/access-token-output.png)

## <a name="get-user-details"></a><span data-ttu-id="17ce0-133">Obtener detalles del usuario</span><span class="sxs-lookup"><span data-stu-id="17ce0-133">Get user details</span></span>

<span data-ttu-id="17ce0-134">En esta sección, creará una clase auxiliar que contendrá todas las llamadas a Microsoft Graph y actualizará el `WelcomeViewController` para que use esta nueva clase para obtener el usuario que ha iniciado sesión.</span><span class="sxs-lookup"><span data-stu-id="17ce0-134">In this section you will create a helper class to hold all of the calls to Microsoft Graph and update the `WelcomeViewController` to use this new class to get the logged-in user.</span></span>

1. <span data-ttu-id="17ce0-135">Abra **AuthenticationManager. h** y agregue la siguiente `#import` instrucción en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="17ce0-135">Open **AuthenticationManager.h** and add the following `#import` statement at the top of the file.</span></span>

    ```objc
    #import <MSGraphMSALAuthProvider/MSGraphMSALAuthProvider.h>
    ```

1. <span data-ttu-id="17ce0-136">Agregue la siguiente línea dentro de `@interface` la declaración.</span><span class="sxs-lookup"><span data-stu-id="17ce0-136">Add the following line inside the `@interface` declaration.</span></span>

    ```objc
    - (MSALAuthenticationProvider*) getGraphAuthProvider;
    ```

1. <span data-ttu-id="17ce0-137">Abra **AuthenticationManager. m** y agregue la siguiente función a la `AuthenticationManager` clase.</span><span class="sxs-lookup"><span data-stu-id="17ce0-137">Open **AuthenticationManager.m** and add the following function to the `AuthenticationManager` class.</span></span>

    ```objc
    - (MSALAuthenticationProvider*) getGraphAuthProvider {
        // Create an MSAL auth provider for use with the Graph client
        return [[MSALAuthenticationProvider alloc]
                initWithPublicClientApplication:self.publicClient
                andScopes:self.graphScopes];
    }
    ```

1. <span data-ttu-id="17ce0-138">Cree una nueva **clase táctil de cacao** en el proyecto **GraphTutorial** denominado **GraphManager**.</span><span class="sxs-lookup"><span data-stu-id="17ce0-138">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **GraphManager**.</span></span> <span data-ttu-id="17ce0-139">Elija **NSObject** en el campo **subclase de** .</span><span class="sxs-lookup"><span data-stu-id="17ce0-139">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="17ce0-140">Abra **GraphManager. h** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="17ce0-140">Open **GraphManager.h** and replace its contents with the following code.</span></span>

    ```objc
    #import <Foundation/Foundation.h>
    #import <MSGraphClientSDK/MSGraphClientSDK.h>
    #import <MSGraphClientModels/MSGraphClientModels.h>
    #import "AuthenticationManager.h"

    NS_ASSUME_NONNULL_BEGIN

    typedef void (^GetMeCompletionBlock)(MSGraphUser* _Nullable user, NSError* _Nullable error);

    @interface GraphManager : NSObject

    + (id) instance;
    - (void) getMeWithCompletionBlock: (GetMeCompletionBlock)completionBlock;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. <span data-ttu-id="17ce0-141">Abra **GraphManager. m** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="17ce0-141">Open **GraphManager.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "GraphManager.h"

    @interface GraphManager()

    @property MSHTTPClient* graphClient;

    @end

    @implementation GraphManager

    + (id) instance {
        static GraphManager *singleInstance = nil;
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^ {
            singleInstance = [[self alloc] init];
        });

        return singleInstance;
    }

    - (id) init {
        if (self = [super init]) {

            MSALAuthenticationProvider* authProvider =
            [AuthenticationManager.instance getGraphAuthProvider];

            // Create the Graph client
            self.graphClient = [MSClientFactory
                                createHTTPClientWithAuthenticationProvider:authProvider];
        }

        return self;
    }

    - (void) getMeWithCompletionBlock:(GetMeCompletionBlock)completionBlock {
        // GET /me
        NSString* meUrlString = [NSString stringWithFormat:@"%@/me", MSGraphBaseURL];
        NSURL* meUrl = [[NSURL alloc] initWithString:meUrlString];
        NSMutableURLRequest* meRequest = [[NSMutableURLRequest alloc] initWithURL:meUrl];

        MSURLSessionDataTask* meDataTask =
        [[MSURLSessionDataTask alloc]
            initWithRequest:meRequest
            client:self.graphClient
            completion:^(NSData *data, NSURLResponse *response, NSError *error) {
                if (error) {
                    completionBlock(nil, error);
                    return;
                }

                // Deserialize the response as a user
                NSError* graphError;
                MSGraphUser* user = [[MSGraphUser alloc] initWithData:data error:&graphError];

                if (graphError) {
                    completionBlock(nil, graphError);
                } else {
                    completionBlock(user, nil);
                }
            }];

        // Execute the request
        [meDataTask execute];
    }

    @end
    ```

1. <span data-ttu-id="17ce0-142">Abra **WelcomeViewController. m** y agregue las siguientes `#import` instrucciones en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="17ce0-142">Open **WelcomeViewController.m** and add the following `#import` statements at the top of the file.</span></span>

    ```objc
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>
    ```

1. <span data-ttu-id="17ce0-143">Agregue la siguiente propiedad a la `WelcomeViewController` declaración de interfaz.</span><span class="sxs-lookup"><span data-stu-id="17ce0-143">Add the following property to the `WelcomeViewController` interface declaration.</span></span>

    ```objc
    @property SpinnerViewController* spinner;
    ```

1. <span data-ttu-id="17ce0-144">Reemplace el existente `viewDidLoad` por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="17ce0-144">Replace the existing `viewDidLoad` with the following code.</span></span>

    ```objc
    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        self.spinner = [SpinnerViewController alloc];
        [self.spinner startWithContainer:self];

        self.userProfilePhoto.image = [UIImage imageNamed:@"DefaultUserPhoto"];

        [GraphManager.instance
         getMeWithCompletionBlock:^(MSGraphUser * _Nullable user, NSError * _Nullable error) {
            dispatch_async(dispatch_get_main_queue(), ^{
                [self.spinner stop];

                if (error) {
                    // Show the error
                    UIAlertController* alert = [UIAlertController
                                                alertControllerWithTitle:@"Error getting user profile"
                                                message:error.debugDescription
                                                preferredStyle:UIAlertControllerStyleAlert];

                    UIAlertAction* okButton = [UIAlertAction
                                               actionWithTitle:@"OK"
                                               style:UIAlertActionStyleDefault
                                               handler:nil];

                    [alert addAction:okButton];
                    [self presentViewController:alert animated:true completion:nil];
                    return;
                }

                // Set display name
                self.userDisplayName.text = user.displayName ? : @"Mysterious Stranger";
                [self.userDisplayName sizeToFit];

                // AAD users have email in the mail attribute
                // Personal accounts have email in the userPrincipalName attribute
                self.userEmail.text = user.mail ? : user.userPrincipalName;
                [self.userEmail sizeToFit];
            });
         }];
    }
    ```

<span data-ttu-id="17ce0-145">Si guarda los cambios y reinicia la aplicación ahora, después de iniciar sesión, la interfaz de usuario se actualizará con el nombre para mostrar y la dirección de correo electrónico del usuario.</span><span class="sxs-lookup"><span data-stu-id="17ce0-145">If you save your changes and restart the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>
