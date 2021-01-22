<!-- markdownlint-disable MD002 MD041 -->

Comience por crear un nuevo proyecto objective-C.

1. Abra Xcode. En el **menú** Archivo, seleccione **Nuevo** y, a **continuación, Proyecto**.
1. Elija la **plantilla aplicación** y seleccione **Siguiente**.

    ![Captura de pantalla del cuadro de diálogo de selección de plantilla Xcode](./images/xcode-select-template.png)

1. Establezca el **nombre del producto** en y el `GraphTutorial` **idioma** en **Objective-C**.
1. Rellene los campos restantes y seleccione **Siguiente**.
1. Elija una ubicación para el proyecto y seleccione **Crear**.

## <a name="install-dependencies"></a>Instalar dependencias

Antes de pasar, instale algunas dependencias adicionales que usará más adelante.

- [Biblioteca de autenticación de Microsoft (MSAL) para iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) para autenticarse con Azure AD.
- [SDK de Microsoft Graph para objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) para realizar llamadas a Microsoft Graph.
- [SDK de modelos de Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-objc-models) para Objective C para objetos fuertemente tipos que representan recursos de Microsoft Graph como usuarios o eventos.

1. Salga de Xcode.
1. Abra Terminal y cambie el directorio a la ubicación del **proyecto graphTutorial.**
1. Ejecute el siguiente comando para crear un Podfile.

    ```Shell
    pod init
    ```

1. Abre el Podfile y agrega las siguientes líneas justo después de la `use_frameworks!` línea.

    ```Ruby
    pod 'MSAL', '~> 1.1.13'
    pod 'MSGraphClientSDK', ' ~> 1.0.0'
    pod 'MSGraphClientModels', '~> 1.3.0'
    ```

1. Guarde el podfile y, a continuación, ejecute el siguiente comando para instalar las dependencias.

    ```Shell
    pod install
    ```

1. Una vez completado el comando, abra el **graphTutorial.xcworkspace** recién creado en Xcode.

## <a name="design-the-app"></a>Diseñar la aplicación

En esta sección creará las vistas para la aplicación: una página de inicio de sesión, un navegador de barras de pestañas, una página de bienvenida y una página de calendario. También crearás una superposición de indicador de actividad.

### <a name="create-sign-in-page"></a>Crear página de inicio de sesión

1. Expanda la **carpeta GraphTutorial** en Xcode y, a continuación, seleccione el **archivo ViewController.m.**
1. En el **Inspector de** archivos, cambie el **nombre** del archivo a `SignInViewController.m` .

    ![Captura de pantalla del Inspector de archivos](./images/rename-file.png)

1. Abra **SignInViewController.m y** reemplace su contenido por el siguiente código.

    ```objc
    #import "SignInViewController.h"

    @interface SignInViewController ()

    @end

    @implementation SignInViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.
    }

    - (IBAction)signIn {
        [self performSegueWithIdentifier: @"userSignedIn" sender: nil];
    }
    @end
    ```

1. Seleccione el **archivo ViewController.h.**
1. En el **Inspector de** archivos, cambie el **nombre** del archivo a `SignInViewController.h` .
1. Abra **SignInViewController.h** y cambie todas las instancias de `ViewController` a `SignInViewController` .

1. Abre el **archivo Main.storyboard.**
1. Expanda **Ver escena del controlador** y, a continuación, seleccione Ver **controlador.**

    ![Captura de pantalla de Xcode con el controlador de vista seleccionado](./images/storyboard-select-view-controller.png)

1. Seleccione el **inspector de identidad** y, a continuación, cambie el desplegable **de** clases a **SignInViewController**.

    ![Captura de pantalla del inspector de identidad](./images/change-class.png)

1. Seleccione la **biblioteca y,** a continuación, arrastre un **botón** al controlador **de vista de inicio de sesión.**

    ![Captura de pantalla de la biblioteca en Xcode](./images/add-button-to-view.png)

1. Con el botón seleccionado, seleccione el **Inspector de atributos** y cambie el **título** del botón a `Sign In` .

    ![Captura de pantalla del campo Título en el Inspector de atributos en Xcode](./images/set-button-title.png)

1. Con el botón seleccionado, selecciona el botón **Alinear** en la parte inferior del guión gráfico. Seleccione horizontalmente en **contenedor y** **verticalmente** en restricciones de contenedor, deje sus valores como 0 y, a continuación, seleccione **Agregar 2 restricciones**.

    ![Captura de pantalla de la configuración de restricciones de alineación en Xcode](./images/add-alignment-constraints.png)

1. Seleccione el **controlador de vista de inicio de sesión** y, a continuación, seleccione el Inspector de **conexiones**.
1. En **Acciones recibidas,** arrastre el círculo sin rellenar junto a **signIn** en el botón. Selecciona **Tocar dentro en** el menú emergente.

    ![Captura de pantalla de arrastrar la acción signIn al botón en Xcode](./images/connect-sign-in-button.png)

### <a name="create-tab-bar"></a>Crear barra de pestañas

1. Selecciona la **biblioteca y,** a continuación, arrastra un controlador **de barra de pestañas** al guión gráfico.
1. Seleccione el **controlador de vista de inicio de sesión** y, a continuación, seleccione el Inspector de **conexiones**.
1. En **Segues desencadenado,** arrastra el círculo sin rellenar junto a **manual** en el controlador de la barra **de pestañas** en el guión gráfico. Seleccione **Presentar modalmente** en el menú emergente.

    ![Captura de pantalla de arrastrar un segue manual al nuevo controlador de barra de pestañas en Xcode](./images/add-segue.png)

1. Seleccione el segue que acaba de agregar y, a continuación, seleccione el **Inspector de atributos.** Establezca el **campo Identificador** en y `userSignedIn` establezca **Presentación** en **Pantalla completa.**

    ![Captura de pantalla del campo Identificador en el Inspector de atributos en Xcode](./images/set-segue-identifier.png)

1. Seleccione la **escena elemento 1** y, a continuación, seleccione el **Inspector de conexiones**.
1. En **Segues desencadenado,** arrastra el círculo sin rellenar junto a **manual** en el controlador de **vista de** inicio de sesión en el guión gráfico. Seleccione **Presentar modalmente** en el menú emergente.
1. Seleccione el segue que acaba de agregar y, a continuación, seleccione el **Inspector de atributos.** Establezca el **campo Identificador** en y `userSignedOut` establezca **Presentación** en **Pantalla completa.**

### <a name="create-welcome-page"></a>Crear página principal

1. Selecciona el **archivo Assets.xcassets.**
1. En el **menú Editor,** seleccione **Agregar nuevo** activo y, a continuación, Conjunto **de imágenes.**
1. Seleccione el nuevo **activo de** imagen y use el **Inspector de** atributos para establecer su **nombre** en `DefaultUserPhoto` .
1. Agrega cualquier imagen que quieras que sirva como foto de perfil de usuario predeterminada.

    ![Captura de pantalla de la vista de activos del conjunto de imágenes en Xcode](./images/add-default-user-photo.png)

1. Cree un nuevo **archivo de clase Cocoa Touch** en la carpeta **GraphTutorial** denominada `WelcomeViewController` . Elija **UIViewController** en la **subclase del** campo.
1. Abre **WelcomeViewController.h** y agrega el siguiente código dentro de la `@interface` declaración.

    ```objc
    @property (nonatomic) IBOutlet UIImageView *userProfilePhoto;
    @property (nonatomic) IBOutlet UILabel *userDisplayName;
    @property (nonatomic) IBOutlet UILabel *userEmail;
    ```

1. Abra **WelcomeViewController.m y** reemplace su contenido por el código siguiente.

    ```objc
    #import "WelcomeViewController.h"

    @interface WelcomeViewController ()

    @end

    @implementation WelcomeViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        // TEMPORARY
        self.userProfilePhoto.image = [UIImage imageNamed:@"DefaultUserPhoto"];
        self.userDisplayName.text = @"Default User";
        [self.userDisplayName sizeToFit];
        self.userEmail.text = @"default@contoso.com";
        [self.userEmail sizeToFit];
    }

    - (IBAction)signOut {
        [self performSegueWithIdentifier: @"userSignedOut" sender: nil];
    }

    @end
    ```

1. Abra **Main.storyboard**. Seleccione la **escena elemento 1** y, a continuación, seleccione el **inspector de identidad**. Cambie el **valor de** clase a **WelcomeViewController**.
1. Mediante la **biblioteca**, agregue los siguientes elementos a la **escena de elemento 1**.

    - Una **vista de imagen**
    - Dos **etiquetas**
    - Un **botón**

1. Con el **Inspector de** conexiones, realice las siguientes conexiones.

    - Vincula la **salida userDisplayName** a la primera etiqueta.
    - Vincular la **salida userEmail** a la segunda etiqueta.
    - Vincular la **salida userProfilePhoto** a la vista de imagen.
    - Vincular la **acción de inicio de** sesión recibido a la pantalla Táctil dentro del **botón.**

1. Seleccione la vista de imagen y, a continuación, seleccione **el Inspector de tamaño**.
1. Establezca el **ancho** y **el alto** en 196.
1. Usa el **botón** Alinear para agregar **horizontalmente la restricción de** contenedor con un valor de 0.
1. Usa el **botón Agregar nuevas restricciones** (junto **al** botón Alinear) para agregar las siguientes restricciones:

    - Alinear arriba a: área segura, valor: 0
    - Espacio inferior para: Nombre para mostrar del usuario, valor: Estándar
    - Alto, valor: 196
    - Ancho, valor: 196

    ![Captura de pantalla de la nueva configuración de restricciones en Xcode](./images/add-new-constraints.png)

1. Selecciona la primera etiqueta y, a continuación, usa el botón **Alinear** para agregar horizontalmente la restricción **de** contenedor con un valor de 0.
1. Use el **botón Agregar nuevas restricciones** para agregar las siguientes restricciones:

    - Espacio superior para: Foto de perfil de usuario, valor: Estándar
    - Espacio inferior a: Correo electrónico del usuario, valor: Estándar

1. Seleccione la segunda etiqueta y, a continuación, seleccione el **Inspector de atributos.**
1. Cambie el **color a** gris **oscuro y** cambie **la** fuente al **sistema 12.0**.
1. Usa el **botón** Alinear para agregar **horizontalmente la restricción de** contenedor con un valor de 0.
1. Use el **botón Agregar nuevas restricciones** para agregar las siguientes restricciones:

    - Espacio superior para: Nombre para mostrar del usuario, valor: Estándar
    - Espacio inferior para: Cerrar sesión, valor: 14

1. Seleccione el botón y, a continuación, seleccione el **Inspector de atributos.**
1. Cambie el **título** a `Sign Out` .
1. Usa el **botón** Alinear para agregar **horizontalmente la restricción de** contenedor con un valor de 0.
1. Use el **botón Agregar nuevas restricciones** para agregar las siguientes restricciones:

    - Espacio superior para: Correo electrónico del usuario, valor: 14

1. Seleccione el elemento de la barra de pestañas en la parte inferior de la escena y, a continuación, seleccione el **Inspector de atributos.** Cambie el **título** a `Me` .
1. En el **menú Editor,** seleccione Resolver problemas **de** diseño automático y, a continuación, seleccione Agregar **restricciones** que faltan debajo de Todas las **vistas en el controlador de vista de bienvenida.**

La escena de bienvenida debería ser similar a esta una vez que termines.

![Captura de pantalla del diseño de la escena de bienvenida](./images/welcome-scene-layout.png)

### <a name="create-calendar-page"></a>Crear página de calendario

1. Cree un nuevo **archivo de clase Cocoa Touch** en la carpeta **GraphTutorial** denominada `CalendarViewController` . Elija **UIViewController** en la **subclase del** campo.
1. Abre **CalendarViewController.h** y agrega el siguiente código dentro de la `@interface` declaración.

    ```objc
    @property (nonatomic) IBOutlet UITextView *calendarJSON;
    ```

1. Abra **CalendarViewController.m y** reemplace su contenido por el siguiente código.

    ```objc
    #import "CalendarViewController.h"

    @interface CalendarViewController ()

    @end

    @implementation CalendarViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        // TEMPORARY
        self.calendarJSON.text = @"Calendar";
        [self.calendarJSON sizeToFit];
    }

    @end
    ```

1. Abra **Main.storyboard**. Seleccione la **escena del elemento 2** y, a continuación, seleccione el **inspector de identidad**. Cambie el **valor de** clase a **CalendarViewController**.
1. Mediante la **biblioteca**, agregue una **vista de texto** a la escena de elemento **2**.
1. Seleccione la vista de texto que acaba de agregar. En el **Editor,** elija **Insertar en** y, a continuación, **Vista de desplazamiento**.
1. Mediante el **Inspector de conexiones,** conecte **la salida calendarJSON** a la vista de texto.
1. Seleccione el elemento de la barra de pestañas en la parte inferior de la escena y, a continuación, seleccione el **Inspector de atributos.** Cambie el **título** a `Calendar` .
1. En el **menú Editor,** seleccione Resolver problemas **de** diseño automático y, a continuación, seleccione Agregar **restricciones** que faltan debajo de Todas las **vistas en el controlador de vista de bienvenida.**

La escena del calendario debería ser similar a esta una vez que termines.

![Captura de pantalla del diseño de la escena calendario](./images/calendar-scene-layout.png)

### <a name="create-activity-indicator"></a>Crear indicador de actividad

1. Cree un nuevo **archivo de clase Cocoa Touch** en la carpeta **GraphTutorial** denominada `SpinnerViewController` . Elija **UIViewController** en la **subclase del** campo.
1. Abre **SpinnerViewController.h** y agrega el siguiente código dentro de la `@interface` declaración.

    ```objc
    - (void) startWithContainer:(UIViewController*) container;
    - (void) stop;
    ```

1. Abre **SpinnerViewController.m y** reemplaza su contenido por el siguiente código.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/SpinnerViewController.m" id="SpinnerViewSnippet":::

## <a name="test-the-app"></a>Probar la aplicación

Guarda los cambios e inicia la aplicación. Debería poder moverse entre las pantallas con los botones Iniciar y cerrar **sesión** y la barra de pestañas. 

![Capturas de pantalla de la aplicación](./images/app-screens.png)
