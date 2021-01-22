<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="bdee3-101">En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="bdee3-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="bdee3-102">Esto es necesario para obtener el token de acceso OAuth necesario para llamar a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="bdee3-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="bdee3-103">Para ello, integrará la Biblioteca de autenticación de [Microsoft (MSAL) para iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="bdee3-103">To do this, you will integrate the [Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) into the application.</span></span>

1. <span data-ttu-id="bdee3-104">Cree un nuevo archivo **de lista de** propiedades en el proyecto **GraphTutorial** denominado **AuthSettings.plist**.</span><span class="sxs-lookup"><span data-stu-id="bdee3-104">Create a new **Property List** file in the **GraphTutorial** project named **AuthSettings.plist**.</span></span>
1. <span data-ttu-id="bdee3-105">Agregue los siguientes elementos al archivo en el **diccionario** raíz.</span><span class="sxs-lookup"><span data-stu-id="bdee3-105">Add the following items to the file in the **Root** dictionary.</span></span>

    | <span data-ttu-id="bdee3-106">Key </span><span class="sxs-lookup"><span data-stu-id="bdee3-106">Key</span></span> | <span data-ttu-id="bdee3-107">Tipo</span><span class="sxs-lookup"><span data-stu-id="bdee3-107">Type</span></span> | <span data-ttu-id="bdee3-108">Valor</span><span class="sxs-lookup"><span data-stu-id="bdee3-108">Value</span></span> |
    |-----|------|-------|
    | `AppId` | <span data-ttu-id="bdee3-109">Cadena</span><span class="sxs-lookup"><span data-stu-id="bdee3-109">String</span></span> | <span data-ttu-id="bdee3-110">El id. de aplicación de Azure Portal</span><span class="sxs-lookup"><span data-stu-id="bdee3-110">The application ID from the Azure portal</span></span> |
    | `GraphScopes` | <span data-ttu-id="bdee3-111">Matriz</span><span class="sxs-lookup"><span data-stu-id="bdee3-111">Array</span></span> | <span data-ttu-id="bdee3-112">Tres valores de cadena: `User.Read` `MailboxSettings.Read` , y `Calendars.ReadWrite`</span><span class="sxs-lookup"><span data-stu-id="bdee3-112">Three String values: `User.Read`, `MailboxSettings.Read`, and `Calendars.ReadWrite`</span></span> |

    ![Captura de pantalla del archivo AuthSettings.plist en Xcode](./images/auth-settings.png)

> [!IMPORTANT]
> <span data-ttu-id="bdee3-114">Si usas el control de código fuente como Git, ahora sería un buen momento para excluir el archivo **AuthSettings.plist** del control de código fuente para evitar la pérdida involuntaria del identificador de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="bdee3-114">If you're using source control such as git, now would be a good time to exclude the **AuthSettings.plist** file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="bdee3-115">Implementar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="bdee3-115">Implement sign-in</span></span>

<span data-ttu-id="bdee3-116">En esta sección configurará el proyecto para MSAL, creará una clase de administrador de autenticación y actualizará la aplicación para iniciar y cerrar sesión.</span><span class="sxs-lookup"><span data-stu-id="bdee3-116">In this section you will configure the project for MSAL, create an authentication manager class, and update the app to sign in and sign out.</span></span>

### <a name="configure-project-for-msal"></a><span data-ttu-id="bdee3-117">Configurar el proyecto para MSAL</span><span class="sxs-lookup"><span data-stu-id="bdee3-117">Configure project for MSAL</span></span>

1. <span data-ttu-id="bdee3-118">Agregue un nuevo grupo de llaves a las capacidades del proyecto.</span><span class="sxs-lookup"><span data-stu-id="bdee3-118">Add a new keychain group to your project's capabilities.</span></span>
    1. <span data-ttu-id="bdee3-119">Seleccione el **proyecto GraphTutorial** y, a **continuación, firme & capacidades.**</span><span class="sxs-lookup"><span data-stu-id="bdee3-119">Select the **GraphTutorial** project, then **Signing & Capabilities**.</span></span>
    1. <span data-ttu-id="bdee3-120">Seleccione **+ Capacidad y, a** continuación, haga doble clic en Compartir **cadenas de teclas.**</span><span class="sxs-lookup"><span data-stu-id="bdee3-120">Select **+ Capability**, then double-click **Keychain Sharing**.</span></span>
    1. <span data-ttu-id="bdee3-121">Agregue un grupo de llaves con el valor `com.microsoft.adalcache` .</span><span class="sxs-lookup"><span data-stu-id="bdee3-121">Add a keychain group with the value `com.microsoft.adalcache`.</span></span>

1. <span data-ttu-id="bdee3-122">Control click **Info.plist** and select **Open As**, then **Source Code**.</span><span class="sxs-lookup"><span data-stu-id="bdee3-122">Control click **Info.plist** and select **Open As**, then **Source Code**.</span></span>
1. <span data-ttu-id="bdee3-123">Agregue lo siguiente dentro del `<dict>` elemento.</span><span class="sxs-lookup"><span data-stu-id="bdee3-123">Add the following inside the `<dict>` element.</span></span>

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

1. <span data-ttu-id="bdee3-124">Abra **AppDelegate.m y** agregue la siguiente instrucción import en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="bdee3-124">Open **AppDelegate.m** and add the following import statement at the top of the file.</span></span>

    ```objc
    #import <MSAL/MSAL.h>
    ```

1. <span data-ttu-id="bdee3-125">Agregue la siguiente función a la clase `AppDelegate`.</span><span class="sxs-lookup"><span data-stu-id="bdee3-125">Add the following function to the `AppDelegate` class.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.m" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a><span data-ttu-id="bdee3-126">Crear administrador de autenticación</span><span class="sxs-lookup"><span data-stu-id="bdee3-126">Create authentication manager</span></span>

1. <span data-ttu-id="bdee3-127">Cree una nueva **clase Cocoa Touch en** el proyecto **GraphTutorial** denominado **AuthenticationManager**.</span><span class="sxs-lookup"><span data-stu-id="bdee3-127">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **AuthenticationManager**.</span></span> <span data-ttu-id="bdee3-128">Elija **NSObject** en la **subclase del** campo.</span><span class="sxs-lookup"><span data-stu-id="bdee3-128">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="bdee3-129">Abra **AuthenticationManager.h** y reemplace su contenido por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="bdee3-129">Open **AuthenticationManager.h** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.h" id="AuthManagerSnippet":::

1. <span data-ttu-id="bdee3-130">Abra **AuthenticationManager.m** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="bdee3-130">Open **AuthenticationManager.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.m" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a><span data-ttu-id="bdee3-131">Agregar inicio y cerrar sesión</span><span class="sxs-lookup"><span data-stu-id="bdee3-131">Add sign-in and sign-out</span></span>

1. <span data-ttu-id="bdee3-132">Abra el **archivo SignInViewController.m** y reemplace su contenido por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="bdee3-132">Open the **SignInViewController.m** file and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.m" id="SignInViewSnippet":::

1. <span data-ttu-id="bdee3-133">Abra **WelcomeViewController.m** y agregue la siguiente `import` instrucción en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="bdee3-133">Open **WelcomeViewController.m** and add the following `import` statement to the top of the file.</span></span>

    ```objc
    #import "AuthenticationManager.h"
    ```

1. <span data-ttu-id="bdee3-134">Reemplace la función `signOut` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="bdee3-134">Replace the existing `signOut` function with the following.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="SignOutSnippet":::

1. <span data-ttu-id="bdee3-135">Guarda los cambios y reinicia la aplicación en simulador.</span><span class="sxs-lookup"><span data-stu-id="bdee3-135">Save your changes and restart the application in Simulator.</span></span>

<span data-ttu-id="bdee3-136">Si inicias sesión en la aplicación, deberías ver un token de acceso en la ventana de salida en Xcode.</span><span class="sxs-lookup"><span data-stu-id="bdee3-136">If you sign in to the app, you should see an access token displayed in the output window in Xcode.</span></span>

![Captura de pantalla de la ventana de salida en Xcode que muestra un token de acceso](./images/access-token-output.png)

## <a name="get-user-details"></a><span data-ttu-id="bdee3-138">Obtener detalles del usuario</span><span class="sxs-lookup"><span data-stu-id="bdee3-138">Get user details</span></span>

<span data-ttu-id="bdee3-139">En esta sección creará una clase auxiliar para contener todas las llamadas a Microsoft Graph y actualizará el uso de esta nueva clase para obtener el usuario que ha `WelcomeViewController` iniciado sesión.</span><span class="sxs-lookup"><span data-stu-id="bdee3-139">In this section you will create a helper class to hold all of the calls to Microsoft Graph and update the `WelcomeViewController` to use this new class to get the logged-in user.</span></span>

1. <span data-ttu-id="bdee3-140">Cree una nueva **clase Cocoa Touch** en el proyecto **GraphTutorial** denominado **GraphManager**.</span><span class="sxs-lookup"><span data-stu-id="bdee3-140">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **GraphManager**.</span></span> <span data-ttu-id="bdee3-141">Elija **NSObject** en la **subclase del** campo.</span><span class="sxs-lookup"><span data-stu-id="bdee3-141">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="bdee3-142">Abra **GraphManager.h** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="bdee3-142">Open **GraphManager.h** and replace its contents with the following code.</span></span>

    ```objc
    #import <Foundation/Foundation.h>
    #import <MSGraphClientSDK/MSGraphClientSDK.h>
    #import <MSGraphClientModels/MSGraphClientModels.h>
    #import <MSGraphClientModels/MSCollection.h>
    #import "AuthenticationManager.h"

    NS_ASSUME_NONNULL_BEGIN

    typedef void (^GetMeCompletionBlock)(MSGraphUser* _Nullable user,
                                         NSError* _Nullable error);

    @interface GraphManager : NSObject

    + (id) instance;
    - (void) getMeWithCompletionBlock: (GetMeCompletionBlock) completion;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. <span data-ttu-id="bdee3-143">Abra **GraphManager.m** y reemplace su contenido por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="bdee3-143">Open **GraphManager.m** and replace its contents with the following code.</span></span>

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
            // Create the Graph client
            self.graphClient = [MSClientFactory
                                createHTTPClientWithAuthenticationProvider:AuthenticationManager.instance];
        }

        return self;
    }

    - (void) getMeWithCompletionBlock: (GetMeCompletionBlock) completion {
        // GET /me
        NSString* meUrlString = [NSString stringWithFormat:@"%@/me?%@",
                                 MSGraphBaseURL,
                                 @"$select=displayName,mail,mailboxSettings,userPrincipalName"];
        NSURL* meUrl = [[NSURL alloc] initWithString:meUrlString];
        NSMutableURLRequest* meRequest = [[NSMutableURLRequest alloc] initWithURL:meUrl];

        MSURLSessionDataTask* meDataTask =
        [[MSURLSessionDataTask alloc]
            initWithRequest:meRequest
            client:self.graphClient
            completion:^(NSData *data, NSURLResponse *response, NSError *error) {
                if (error) {
                    completion(nil, error);
                    return;
                }

                // Deserialize the response as a user
                NSError* graphError;
                MSGraphUser* user = [[MSGraphUser alloc] initWithData:data error:&graphError];

                if (graphError) {
                    completion(nil, graphError);
                } else {
                    completion(user, nil);
                }
            }];

        // Execute the request
        [meDataTask execute];
    }

    @end
    ```

1. <span data-ttu-id="bdee3-144">Abra **WelcomeViewController.m** y agregue las siguientes `#import` instrucciones en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="bdee3-144">Open **WelcomeViewController.m** and add the following `#import` statements at the top of the file.</span></span>

    ```objc
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>
    ```

1. <span data-ttu-id="bdee3-145">Agregue la siguiente propiedad a la declaración `WelcomeViewController` de interfaz.</span><span class="sxs-lookup"><span data-stu-id="bdee3-145">Add the following property to the `WelcomeViewController` interface declaration.</span></span>

    ```objc
    @property SpinnerViewController* spinner;
    ```

1. <span data-ttu-id="bdee3-146">Reemplace el existente `viewDidLoad` por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="bdee3-146">Replace the existing `viewDidLoad` with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="ViewDidLoadSnippet":::

<span data-ttu-id="bdee3-147">Si guarda los cambios y reinicia la aplicación ahora, después de iniciar sesión, la interfaz de usuario se actualiza con el nombre para mostrar y la dirección de correo electrónico del usuario.</span><span class="sxs-lookup"><span data-stu-id="bdee3-147">If you save your changes and restart the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>
