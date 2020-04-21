<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="61acc-101">En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="61acc-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="61acc-102">Esto es necesario para obtener el token de acceso de OAuth necesario para llamar a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="61acc-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="61acc-103">Para ello, integrará la [biblioteca de autenticación de Microsoft (MSAL) para iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="61acc-103">To do this, you will integrate the [Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) into the application.</span></span>

1. <span data-ttu-id="61acc-104">Cree un nuevo archivo de **lista de propiedades** en el proyecto **GraphTutorial** denominado **AuthSettings. plist**.</span><span class="sxs-lookup"><span data-stu-id="61acc-104">Create a new **Property List** file in the **GraphTutorial** project named **AuthSettings.plist**.</span></span>
1. <span data-ttu-id="61acc-105">Agregue los siguientes elementos al archivo en el diccionario **raíz** .</span><span class="sxs-lookup"><span data-stu-id="61acc-105">Add the following items to the file in the **Root** dictionary.</span></span>

    | <span data-ttu-id="61acc-106">Key </span><span class="sxs-lookup"><span data-stu-id="61acc-106">Key</span></span> | <span data-ttu-id="61acc-107">Tipo</span><span class="sxs-lookup"><span data-stu-id="61acc-107">Type</span></span> | <span data-ttu-id="61acc-108">Valor</span><span class="sxs-lookup"><span data-stu-id="61acc-108">Value</span></span> |
    |-----|------|-------|
    | `AppId` | <span data-ttu-id="61acc-109">Cadena</span><span class="sxs-lookup"><span data-stu-id="61acc-109">String</span></span> | <span data-ttu-id="61acc-110">El identificador de la aplicación del portal de Azure</span><span class="sxs-lookup"><span data-stu-id="61acc-110">The application ID from the Azure portal</span></span> |
    | `GraphScopes` | <span data-ttu-id="61acc-111">Matriz</span><span class="sxs-lookup"><span data-stu-id="61acc-111">Array</span></span> | <span data-ttu-id="61acc-112">Dos valores de cadena `User.Read` : y`Calendars.Read`</span><span class="sxs-lookup"><span data-stu-id="61acc-112">Two String values: `User.Read` and `Calendars.Read`</span></span> |

    ![Captura de pantalla del archivo AuthSettings. plist en Xcode](./images/auth-settings.png)

> [!IMPORTANT]
> <span data-ttu-id="61acc-114">Si usa un control de código fuente como GIT, ahora sería un buen momento para excluir el archivo **AuthSettings. plist** del control de código fuente para evitar la pérdida inadvertida del identificador de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="61acc-114">If you're using source control such as git, now would be a good time to exclude the **AuthSettings.plist** file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="61acc-115">Implementar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="61acc-115">Implement sign-in</span></span>

<span data-ttu-id="61acc-116">En esta sección, configurará el proyecto para MSAL, creará una clase de administrador de autenticación y actualizará la aplicación para iniciar y cerrar sesión.</span><span class="sxs-lookup"><span data-stu-id="61acc-116">In this section you will configure the project for MSAL, create an authentication manager class, and update the app to sign in and sign out.</span></span>

### <a name="configure-project-for-msal"></a><span data-ttu-id="61acc-117">Configurar Project para MSAL</span><span class="sxs-lookup"><span data-stu-id="61acc-117">Configure project for MSAL</span></span>

1. <span data-ttu-id="61acc-118">Agregue un nuevo grupo de llaves a las capacidades del proyecto.</span><span class="sxs-lookup"><span data-stu-id="61acc-118">Add a new keychain group to your project's capabilities.</span></span>
    1. <span data-ttu-id="61acc-119">Seleccione el proyecto **GraphTutorial** y, a continuación, **firme funciones de &**.</span><span class="sxs-lookup"><span data-stu-id="61acc-119">Select the **GraphTutorial** project, then **Signing & Capabilities**.</span></span>
    1. <span data-ttu-id="61acc-120">Seleccione **+ capacidad**y, a continuación, haga doble clic en **uso compartido de llaves**.</span><span class="sxs-lookup"><span data-stu-id="61acc-120">Select **+ Capability**, then double-click **Keychain Sharing**.</span></span>
    1. <span data-ttu-id="61acc-121">Agregue un grupo de llaves con el `com.microsoft.adalcache`valor.</span><span class="sxs-lookup"><span data-stu-id="61acc-121">Add a keychain group with the value `com.microsoft.adalcache`.</span></span>

1. <span data-ttu-id="61acc-122">Control haga clic en **info. plist** y seleccione **Abrir como**y, a continuación, **código fuente**.</span><span class="sxs-lookup"><span data-stu-id="61acc-122">Control click **Info.plist** and select **Open As**, then **Source Code**.</span></span>
1. <span data-ttu-id="61acc-123">Agregue lo siguiente dentro del `<dict>` elemento.</span><span class="sxs-lookup"><span data-stu-id="61acc-123">Add the following inside the `<dict>` element.</span></span>

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

1. <span data-ttu-id="61acc-124">Abra **AppDelegate. m** y agregue la siguiente instrucción Import en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="61acc-124">Open **AppDelegate.m** and add the following import statement at the top of the file.</span></span>

    ```objc
    #import <MSAL/MSAL.h>
    ```

1. <span data-ttu-id="61acc-125">Agregue la siguiente función a la clase `AppDelegate`.</span><span class="sxs-lookup"><span data-stu-id="61acc-125">Add the following function to the `AppDelegate` class.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.m" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a><span data-ttu-id="61acc-126">Crear el administrador de autenticación</span><span class="sxs-lookup"><span data-stu-id="61acc-126">Create authentication manager</span></span>

1. <span data-ttu-id="61acc-127">Cree una nueva **clase táctil de cacao** en el proyecto **GraphTutorial** denominado **AuthenticationManager**.</span><span class="sxs-lookup"><span data-stu-id="61acc-127">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **AuthenticationManager**.</span></span> <span data-ttu-id="61acc-128">Elija **NSObject** en el campo **subclase de** .</span><span class="sxs-lookup"><span data-stu-id="61acc-128">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="61acc-129">Abra **AuthenticationManager. h** y reemplace el contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="61acc-129">Open **AuthenticationManager.h** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.h" id="AuthManagerSnippet":::

1. <span data-ttu-id="61acc-130">Abra **AuthenticationManager. m** y reemplace el contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="61acc-130">Open **AuthenticationManager.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.m" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a><span data-ttu-id="61acc-131">Agregar Inicio y cierre de sesión</span><span class="sxs-lookup"><span data-stu-id="61acc-131">Add sign-in and sign-out</span></span>

1. <span data-ttu-id="61acc-132">Abra el archivo **SignInViewController. m** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="61acc-132">Open the **SignInViewController.m** file and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.m" id="SignInViewSnippet":::

1. <span data-ttu-id="61acc-133">Abra **WelcomeViewController. m** y agregue la siguiente `import` instrucción a la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="61acc-133">Open **WelcomeViewController.m** and add the following `import` statement to the top of the file.</span></span>

    ```objc
    #import "AuthenticationManager.h"
    ```

1. <span data-ttu-id="61acc-134">Reemplace la función `signOut` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="61acc-134">Replace the existing `signOut` function with the following.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="SignOutSnippet":::

1. <span data-ttu-id="61acc-135">Guarde los cambios y reinicie la aplicación en el simulador.</span><span class="sxs-lookup"><span data-stu-id="61acc-135">Save your changes and restart the application in Simulator.</span></span>

<span data-ttu-id="61acc-136">Si inicia sesión en la aplicación, debería ver un token de acceso que se muestra en la ventana de salida de Xcode.</span><span class="sxs-lookup"><span data-stu-id="61acc-136">If you sign in to the app, you should see an access token displayed in the output window in Xcode.</span></span>

![Captura de pantalla de la ventana de salida de Xcode que muestra un token de acceso](./images/access-token-output.png)

## <a name="get-user-details"></a><span data-ttu-id="61acc-138">Obtener detalles del usuario</span><span class="sxs-lookup"><span data-stu-id="61acc-138">Get user details</span></span>

<span data-ttu-id="61acc-139">En esta sección, creará una clase auxiliar que contendrá todas las llamadas a Microsoft Graph y actualizará el `WelcomeViewController` para que use esta nueva clase para obtener el usuario que ha iniciado sesión.</span><span class="sxs-lookup"><span data-stu-id="61acc-139">In this section you will create a helper class to hold all of the calls to Microsoft Graph and update the `WelcomeViewController` to use this new class to get the logged-in user.</span></span>

1. <span data-ttu-id="61acc-140">Cree una nueva **clase táctil de cacao** en el proyecto **GraphTutorial** denominado **GraphManager**.</span><span class="sxs-lookup"><span data-stu-id="61acc-140">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **GraphManager**.</span></span> <span data-ttu-id="61acc-141">Elija **NSObject** en el campo **subclase de** .</span><span class="sxs-lookup"><span data-stu-id="61acc-141">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="61acc-142">Abra **GraphManager. h** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="61acc-142">Open **GraphManager.h** and replace its contents with the following code.</span></span>

    ```objc
    #import <Foundation/Foundation.h>
    #import <MSGraphClientSDK/MSGraphClientSDK.h>
    #import <MSGraphClientModels/MSGraphClientModels.h>
    #import <MSGraphClientModels/MSCollection.h>
    #import "AuthenticationManager.h"

    NS_ASSUME_NONNULL_BEGIN

    typedef void (^GetMeCompletionBlock)(MSGraphUser* _Nullable user, NSError* _Nullable error);

    @interface GraphManager : NSObject

    + (id) instance;
    - (void) getMeWithCompletionBlock: (GetMeCompletionBlock)completionBlock;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. <span data-ttu-id="61acc-143">Abra **GraphManager. m** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="61acc-143">Open **GraphManager.m** and replace its contents with the following code.</span></span>

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

1. <span data-ttu-id="61acc-144">Abra **WelcomeViewController. m** y agregue las siguientes `#import` instrucciones en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="61acc-144">Open **WelcomeViewController.m** and add the following `#import` statements at the top of the file.</span></span>

    ```objc
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>
    ```

1. <span data-ttu-id="61acc-145">Agregue la siguiente propiedad a la `WelcomeViewController` declaración de interfaz.</span><span class="sxs-lookup"><span data-stu-id="61acc-145">Add the following property to the `WelcomeViewController` interface declaration.</span></span>

    ```objc
    @property SpinnerViewController* spinner;
    ```

1. <span data-ttu-id="61acc-146">Reemplace el existente `viewDidLoad` por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="61acc-146">Replace the existing `viewDidLoad` with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="ViewDidLoadSnippet":::

<span data-ttu-id="61acc-147">Si guarda los cambios y reinicia la aplicación ahora, después de iniciar sesión, la interfaz de usuario se actualizará con el nombre para mostrar y la dirección de correo electrónico del usuario.</span><span class="sxs-lookup"><span data-stu-id="61acc-147">If you save your changes and restart the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>
