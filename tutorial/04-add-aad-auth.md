<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD. Esto es necesario para obtener el token de acceso OAuth necesario para llamar a Microsoft Graph. Para ello, integrará la Biblioteca de autenticación de [Microsoft (MSAL) para iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) en la aplicación.

1. Cree un nuevo archivo **de lista de** propiedades en el proyecto **GraphTutorial** denominado **AuthSettings.plist**.
1. Agregue los siguientes elementos al archivo en el **diccionario** raíz.

    | Key  | Tipo | Valor |
    |-----|------|-------|
    | `AppId` | Cadena | El id. de aplicación de Azure Portal |
    | `GraphScopes` | Matriz | Tres valores de cadena: `User.Read` `MailboxSettings.Read` , y `Calendars.ReadWrite` |

    ![Captura de pantalla del archivo AuthSettings.plist en Xcode](./images/auth-settings.png)

> [!IMPORTANT]
> Si usas el control de código fuente como Git, ahora sería un buen momento para excluir el archivo **AuthSettings.plist** del control de código fuente para evitar la pérdida involuntaria del identificador de la aplicación.

## <a name="implement-sign-in"></a>Implementar el inicio de sesión

En esta sección configurará el proyecto para MSAL, creará una clase de administrador de autenticación y actualizará la aplicación para iniciar y cerrar sesión.

### <a name="configure-project-for-msal"></a>Configurar el proyecto para MSAL

1. Agregue un nuevo grupo de llaves a las capacidades del proyecto.
    1. Seleccione el **proyecto GraphTutorial** y, a **continuación, firme & capacidades.**
    1. Seleccione **+ Capacidad y, a** continuación, haga doble clic en Compartir **cadenas de teclas.**
    1. Agregue un grupo de llaves con el valor `com.microsoft.adalcache` .

1. Control click **Info.plist** and select **Open As**, then **Source Code**.
1. Agregue lo siguiente dentro del `<dict>` elemento.

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

1. Abra **AppDelegate.m y** agregue la siguiente instrucción import en la parte superior del archivo.

    ```objc
    #import <MSAL/MSAL.h>
    ```

1. Agregue la siguiente función a la clase `AppDelegate`.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.m" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a>Crear administrador de autenticación

1. Cree una nueva **clase Cocoa Touch en** el proyecto **GraphTutorial** denominado **AuthenticationManager**. Elija **NSObject** en la **subclase del** campo.
1. Abra **AuthenticationManager.h** y reemplace su contenido por el siguiente código.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.h" id="AuthManagerSnippet":::

1. Abra **AuthenticationManager.m** y reemplace su contenido por el código siguiente.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.m" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a>Agregar inicio y cerrar sesión

1. Abra el **archivo SignInViewController.m** y reemplace su contenido por el siguiente código.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.m" id="SignInViewSnippet":::

1. Abra **WelcomeViewController.m** y agregue la siguiente `import` instrucción en la parte superior del archivo.

    ```objc
    #import "AuthenticationManager.h"
    ```

1. Reemplace la función `signOut` existente por lo siguiente.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="SignOutSnippet":::

1. Guarda los cambios y reinicia la aplicación en simulador.

Si inicias sesión en la aplicación, deberías ver un token de acceso en la ventana de salida en Xcode.

![Captura de pantalla de la ventana de salida en Xcode que muestra un token de acceso](./images/access-token-output.png)

## <a name="get-user-details"></a>Obtener detalles del usuario

En esta sección creará una clase auxiliar para contener todas las llamadas a Microsoft Graph y actualizará el uso de esta nueva clase para obtener el usuario que ha `WelcomeViewController` iniciado sesión.

1. Cree una nueva **clase Cocoa Touch** en el proyecto **GraphTutorial** denominado **GraphManager**. Elija **NSObject** en la **subclase del** campo.
1. Abra **GraphManager.h** y reemplace su contenido por el código siguiente.

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

1. Abra **GraphManager.m** y reemplace su contenido por el siguiente código.

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

1. Abra **WelcomeViewController.m** y agregue las siguientes `#import` instrucciones en la parte superior del archivo.

    ```objc
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>
    ```

1. Agregue la siguiente propiedad a la declaración `WelcomeViewController` de interfaz.

    ```objc
    @property SpinnerViewController* spinner;
    ```

1. Reemplace el existente `viewDidLoad` por el código siguiente.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="ViewDidLoadSnippet":::

Si guarda los cambios y reinicia la aplicación ahora, después de iniciar sesión, la interfaz de usuario se actualiza con el nombre para mostrar y la dirección de correo electrónico del usuario.
