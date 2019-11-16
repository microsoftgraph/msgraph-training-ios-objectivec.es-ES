<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="2dad7-101">Para empezar, cree un nuevo proyecto SWIFT.</span><span class="sxs-lookup"><span data-stu-id="2dad7-101">Begin by creating a new Swift project.</span></span>

1. <span data-ttu-id="2dad7-102">Abra Xcode.</span><span class="sxs-lookup"><span data-stu-id="2dad7-102">Open Xcode.</span></span> <span data-ttu-id="2dad7-103">En el menú **archivo** , seleccione **nuevo**y, a continuación, **proyecto**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-103">On the **File** menu, select **New**, then **Project**.</span></span>
1. <span data-ttu-id="2dad7-104">Elija la plantilla **aplicación de vista única** y seleccione **siguiente**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-104">Choose the **Single View App** template and select **Next**.</span></span>

    ![Captura de pantalla del cuadro de diálogo de selección de plantilla de Xcode](./images/xcode-select-template.png)

1. <span data-ttu-id="2dad7-106">Establezca el **nombre** del producto `GraphTutorial` en y el **idioma** en **Objective-C**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-106">Set the **Product Name** to `GraphTutorial` and the **Language** to **Objective-C**.</span></span>
1. <span data-ttu-id="2dad7-107">Rellene los campos restantes y seleccione **siguiente**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-107">Fill in the remaining fields and select **Next**.</span></span>
1. <span data-ttu-id="2dad7-108">Elija una ubicación para el proyecto y seleccione **crear**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-108">Choose a location for the project and select **Create**.</span></span>

## <a name="install-dependencies"></a><span data-ttu-id="2dad7-109">Instalar dependencias</span><span class="sxs-lookup"><span data-stu-id="2dad7-109">Install dependencies</span></span>

<span data-ttu-id="2dad7-110">Antes de continuar, instale algunas dependencias adicionales que usará más adelante.</span><span class="sxs-lookup"><span data-stu-id="2dad7-110">Before moving on, install some additional dependencies that you will use later.</span></span>

- <span data-ttu-id="2dad7-111">[Biblioteca de autenticación de Microsoft (MSAL) para iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) para la autenticación con Azure ad.</span><span class="sxs-lookup"><span data-stu-id="2dad7-111">[Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) for authenticating to with Azure AD.</span></span>
- <span data-ttu-id="2dad7-112">[SDK de Microsoft Graph para Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) para realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="2dad7-112">[Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="2dad7-113">El [SDK de modelos de Microsoft Graph para Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc-models) para objetos con establecimiento inflexible de tipos que representan recursos de Microsoft Graph, como usuarios o eventos.</span><span class="sxs-lookup"><span data-stu-id="2dad7-113">[Microsoft Graph Models SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc-models) for strongly-typed objects representing Microsoft Graph resources like users or events.</span></span>

1. <span data-ttu-id="2dad7-114">Salga de Xcode.</span><span class="sxs-lookup"><span data-stu-id="2dad7-114">Quit Xcode.</span></span>
1. <span data-ttu-id="2dad7-115">Abra terminal y cambie el directorio a la ubicación del proyecto de **GraphTutorial** .</span><span class="sxs-lookup"><span data-stu-id="2dad7-115">Open Terminal and change the directory to the location of your **GraphTutorial** project.</span></span>
1. <span data-ttu-id="2dad7-116">Ejecute el siguiente comando para crear un Podfile.</span><span class="sxs-lookup"><span data-stu-id="2dad7-116">Run the following command to create a Podfile.</span></span>

    ```Shell
    pod init
    ```

1. <span data-ttu-id="2dad7-117">Abra Podfile y agregue las siguientes líneas justo después de la `use_frameworks!` línea.</span><span class="sxs-lookup"><span data-stu-id="2dad7-117">Open the Podfile and add the following lines just after the `use_frameworks!` line.</span></span>

    ```Ruby
    pod 'MSAL', '~> 1.0.2'
    pod 'MSGraphClientSDK', ' ~> 1.0.0'
    pod 'MSGraphClientModels', '~> 1.3.0'
    ```

1. <span data-ttu-id="2dad7-118">Guarde la Podfile y, a continuación, ejecute el siguiente comando para instalar las dependencias.</span><span class="sxs-lookup"><span data-stu-id="2dad7-118">Save the Podfile, then run the following command to install the dependencies.</span></span>

    ```Shell
    pod install
    ```

1. <span data-ttu-id="2dad7-119">Una vez que se complete el comando, abra el **GraphTutorial. xcworkspace** recién creado en Xcode.</span><span class="sxs-lookup"><span data-stu-id="2dad7-119">Once the command completes, open the newly created **GraphTutorial.xcworkspace** in Xcode.</span></span>

## <a name="design-the-app"></a><span data-ttu-id="2dad7-120">Diseñar la aplicación</span><span class="sxs-lookup"><span data-stu-id="2dad7-120">Design the app</span></span>

<span data-ttu-id="2dad7-121">En esta sección, creará las vistas de la aplicación: una página de inicio de sesión, un explorador de barras de pestañas, una página de bienvenida y una página de calendario.</span><span class="sxs-lookup"><span data-stu-id="2dad7-121">In this section you will create the views for the app: a sign in page, a tab bar navigator, a welcome page, and a calendar page.</span></span> <span data-ttu-id="2dad7-122">También creará una superposición del indicador de actividad.</span><span class="sxs-lookup"><span data-stu-id="2dad7-122">You'll also create an activity indicator overlay.</span></span>

### <a name="create-sign-in-page"></a><span data-ttu-id="2dad7-123">Página de creación de inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="2dad7-123">Create sign in page</span></span>

1. <span data-ttu-id="2dad7-124">Expanda la carpeta **GraphTutorial** en Xcode y, a continuación, seleccione el archivo **ViewController. m** .</span><span class="sxs-lookup"><span data-stu-id="2dad7-124">Expand the **GraphTutorial** folder in Xcode, then select the **ViewController.m** file.</span></span>
1. <span data-ttu-id="2dad7-125">En el **Inspector de archivos**, cambie el **nombre** del archivo a `SignInViewController.m`.</span><span class="sxs-lookup"><span data-stu-id="2dad7-125">In the **File Inspector**, change the **Name** of the file to `SignInViewController.m`.</span></span>

    ![Captura de pantalla del inspector de archivos](./images/rename-file.png)

1. <span data-ttu-id="2dad7-127">Abra **SignInViewController. m** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="2dad7-127">Open **SignInViewController.m** and replace its contents with the following code.</span></span>

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

1. <span data-ttu-id="2dad7-128">Seleccione el archivo **ViewController. h** .</span><span class="sxs-lookup"><span data-stu-id="2dad7-128">Select the **ViewController.h** file.</span></span>
1. <span data-ttu-id="2dad7-129">En el **Inspector de archivos**, cambie el **nombre** del archivo a `SignInViewController.h`.</span><span class="sxs-lookup"><span data-stu-id="2dad7-129">In the **File Inspector**, change the **Name** of the file to `SignInViewController.h`.</span></span>
1. <span data-ttu-id="2dad7-130">Abra **SignInViewController. h** y cambie todas las instancias `ViewController` de `SignInViewController`por.</span><span class="sxs-lookup"><span data-stu-id="2dad7-130">Open **SignInViewController.h** and change all instances of `ViewController` to `SignInViewController`.</span></span>

1. <span data-ttu-id="2dad7-131">Abra el archivo **Main. Storyboard** .</span><span class="sxs-lookup"><span data-stu-id="2dad7-131">Open the **Main.storyboard** file.</span></span>
1. <span data-ttu-id="2dad7-132">Expanda **View Controller Scene**y, a continuación, seleccione **View Controller**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-132">Expand **View Controller Scene**, then select **View Controller**.</span></span>

    ![Una captura de pantalla de Xcode con el controlador de vista seleccionado](./images/storyboard-select-view-controller.png)

1. <span data-ttu-id="2dad7-134">Seleccione el **Inspector de identidad**y, a continuación, cambie el desplegable de **clase** a **SignInViewController**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-134">Select the **Identity Inspector**, then change the **Class** dropdown to **SignInViewController**.</span></span>

    ![Captura de pantalla del inspector de identidades](./images/change-class.png)

1. <span data-ttu-id="2dad7-136">Seleccione la **biblioteca**y, a continuación, arrastre un **botón** hasta el **controlador de vista de inicio de sesión**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-136">Select the **Library**, then drag a **Button** onto the **Sign In View Controller**.</span></span>

    ![Captura de pantalla de la biblioteca en Xcode](./images/add-button-to-view.png)

1. <span data-ttu-id="2dad7-138">Con el botón seleccionado, seleccione el **Inspector de atributos** y cambie el **título** del botón a `Sign In`.</span><span class="sxs-lookup"><span data-stu-id="2dad7-138">With the button selected, select the **Attributes Inspector** and change the **Title** of the button to `Sign In`.</span></span>

    ![Captura de pantalla del campo título en el inspector de atributos de Xcode](./images/set-button-title.png)

1. <span data-ttu-id="2dad7-140">Seleccione el **controlador de vista de inicio de sesión**y, a continuación, seleccione el **Inspector de conexiones**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-140">Select the **Sign In View Controller**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="2dad7-141">En **acciones recibidas**, arrastre el círculo no rellenado junto a **iniciar sesión** en el botón.</span><span class="sxs-lookup"><span data-stu-id="2dad7-141">Under **Received Actions**, drag the unfilled circle next to **signIn** onto the button.</span></span> <span data-ttu-id="2dad7-142">Seleccione **retocar dentro** del menú emergente.</span><span class="sxs-lookup"><span data-stu-id="2dad7-142">Select **Touch Up Inside** on the pop-up menu.</span></span>

    ![Una captura de pantalla de cómo arrastrar la acción de inicio de sesión al botón en Xcode](./images/connect-sign-in-button.png)

1. <span data-ttu-id="2dad7-144">En el menú **Editor** , seleccione **resolver problemas de diseño automático**y, a continuación, seleccione **Agregar restricciones que faltan** debajo **de todas las vistas en el controlador de vista de inicio de sesión**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-144">On the **Editor** menu, select **Resolve Auto Layout Issues**, then select **Add Missing Constraints** underneath **All Views in Sign In View Controller**.</span></span>

### <a name="create-tab-bar"></a><span data-ttu-id="2dad7-145">Crear barra de pestañas</span><span class="sxs-lookup"><span data-stu-id="2dad7-145">Create tab bar</span></span>

1. <span data-ttu-id="2dad7-146">Seleccione la **biblioteca**y, a continuación, arrastre un **controlador de barra de pestañas** hasta el guión gráfico.</span><span class="sxs-lookup"><span data-stu-id="2dad7-146">Select the **Library**, then drag a **Tab Bar Controller** onto the storyboard.</span></span>
1. <span data-ttu-id="2dad7-147">Seleccione el **controlador de vista de inicio de sesión**y, a continuación, seleccione el **Inspector de conexiones**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-147">Select the **Sign In View Controller**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="2dad7-148">En **segues desencadenados**, arrastre el círculo no rellenado junto a **manual** en el controlador de la **barra de pestañas** del guión gráfico.</span><span class="sxs-lookup"><span data-stu-id="2dad7-148">Under **Triggered Segues**, drag the unfilled circle next to **manual** onto the **Tab Bar Controller** on the storyboard.</span></span> <span data-ttu-id="2dad7-149">Seleccione **presente** de forma modal en el menú emergente.</span><span class="sxs-lookup"><span data-stu-id="2dad7-149">Select **Present Modally** in the pop-up menu.</span></span>

    ![Captura de pantalla de cómo arrastrar un segue manual al nuevo controlador de barra de pestañas de Xcode](./images/add-segue.png)

1. <span data-ttu-id="2dad7-151">Seleccione el segue que acaba de agregar y, a continuación, seleccione el **Inspector de atributos**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-151">Select the segue you just added, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="2dad7-152">Establezca el campo **identificador** en `userSignedIn`y establezca **presentación** en **pantalla completa**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-152">Set the **Identifier** field to `userSignedIn`, and set **Presentation** to **Full Screen**.</span></span>

    ![Captura de pantalla del campo identificador en el inspector de atributos de Xcode](./images/set-segue-identifier.png)

1. <span data-ttu-id="2dad7-154">Seleccione la **escena elemento 1**y, a continuación, seleccione el **Inspector de conexiones**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-154">Select the **Item 1 Scene**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="2dad7-155">En **segues desencadenados**, arrastre el círculo no rellenado junto a **manual** en el **controlador de vista de inicio de sesión** del guión gráfico.</span><span class="sxs-lookup"><span data-stu-id="2dad7-155">Under **Triggered Segues**, drag the unfilled circle next to **manual** onto the **Sign In View Controller** on the storyboard.</span></span> <span data-ttu-id="2dad7-156">Seleccione **presente** de forma modal en el menú emergente.</span><span class="sxs-lookup"><span data-stu-id="2dad7-156">Select **Present Modally** in the pop-up menu.</span></span>
1. <span data-ttu-id="2dad7-157">Seleccione el segue que acaba de agregar y, a continuación, seleccione el **Inspector de atributos**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-157">Select the segue you just added, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="2dad7-158">Establezca el campo **identificador** en `userSignedOut`y establezca **presentación** en **pantalla completa**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-158">Set the **Identifier** field to `userSignedOut`, and set **Presentation** to **Full Screen**.</span></span>

### <a name="create-welcome-page"></a><span data-ttu-id="2dad7-159">Crear Página principal</span><span class="sxs-lookup"><span data-stu-id="2dad7-159">Create welcome page</span></span>

1. <span data-ttu-id="2dad7-160">Seleccione el archivo **assets. xcassets** .</span><span class="sxs-lookup"><span data-stu-id="2dad7-160">Select the **Assets.xcassets** file.</span></span>
1. <span data-ttu-id="2dad7-161">En el menú **Editor** , seleccione **Agregar activos**y, a continuación, **nuevo conjunto de imágenes**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-161">On the **Editor** menu, select **Add Assets**, then **New Image Set**.</span></span>
1. <span data-ttu-id="2dad7-162">Seleccione el nuevo recurso de **imagen** y use el **Inspector de atributos** para establecer su `DefaultUserPhoto` **nombre** en.</span><span class="sxs-lookup"><span data-stu-id="2dad7-162">Select the new **Image** asset and use the **Attribute Inspector** to set its **Name** to `DefaultUserPhoto`.</span></span>
1. <span data-ttu-id="2dad7-163">Agregue cualquier imagen que quiera que sirva como una foto de Perfil de usuario predeterminada.</span><span class="sxs-lookup"><span data-stu-id="2dad7-163">Add any image you like to serve as a default user profile photo.</span></span>

    ![Captura de pantalla de la vista de activos del conjunto de imágenes en Xcode](./images/add-default-user-photo.png)

1. <span data-ttu-id="2dad7-165">Cree un nuevo archivo de **clase táctil de cacao** en la carpeta `WelcomeViewController` **GraphTutorial** denominada.</span><span class="sxs-lookup"><span data-stu-id="2dad7-165">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `WelcomeViewController`.</span></span> <span data-ttu-id="2dad7-166">Elija **UIViewController** en el campo **subclase de** .</span><span class="sxs-lookup"><span data-stu-id="2dad7-166">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="2dad7-167">Abra **WelcomeViewController. h** y agregue el siguiente código dentro de `@interface` la declaración.</span><span class="sxs-lookup"><span data-stu-id="2dad7-167">Open **WelcomeViewController.h** and add the following code inside the `@interface` declaration.</span></span>

    ```objc
    @property (nonatomic) IBOutlet UIImageView *userProfilePhoto;
    @property (nonatomic) IBOutlet UILabel *userDisplayName;
    @property (nonatomic) IBOutlet UILabel *userEmail;
    ```

1. <span data-ttu-id="2dad7-168">Abra **WelcomeViewController. m** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="2dad7-168">Open **WelcomeViewController.m** and replace its contents with the following code.</span></span>

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

1. <span data-ttu-id="2dad7-169">Abra **Main. Storyboard**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-169">Open **Main.storyboard**.</span></span> <span data-ttu-id="2dad7-170">Seleccione la **escena elemento 1**y, a continuación, seleccione el **Inspector de identidad**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-170">Select the **Item 1 Scene**, then select the **Identity Inspector**.</span></span> <span data-ttu-id="2dad7-171">Cambie el valor de la **clase** a **WelcomeViewController**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-171">Change the **Class** value to **WelcomeViewController**.</span></span>
1. <span data-ttu-id="2dad7-172">Mediante el uso de la **biblioteca**, agregue los siguientes elementos a la **escena elemento 1**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-172">Using the **Library**, add the following items to the **Item 1 Scene**.</span></span>

    - <span data-ttu-id="2dad7-173">Una **vista de imagen**</span><span class="sxs-lookup"><span data-stu-id="2dad7-173">One **Image View**</span></span>
    - <span data-ttu-id="2dad7-174">Dos **etiquetas**</span><span class="sxs-lookup"><span data-stu-id="2dad7-174">Two **Labels**</span></span>
    - <span data-ttu-id="2dad7-175">Un **botón**</span><span class="sxs-lookup"><span data-stu-id="2dad7-175">One **Button**</span></span>

1. <span data-ttu-id="2dad7-176">Seleccione la vista de imagen y, a continuación, seleccione el **Inspector de tamaño**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-176">Select the image view, then select the **Size Inspector**.</span></span>
1. <span data-ttu-id="2dad7-177">Establezca el **ancho** y el **alto** en 196.</span><span class="sxs-lookup"><span data-stu-id="2dad7-177">Set the **Width** and **Height** to 196.</span></span>
1. <span data-ttu-id="2dad7-178">Seleccione la segunda etiqueta y, a continuación, seleccione el **Inspector de atributos**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-178">Select the second label, then select the **Attributes Inspector**.</span></span>
1. <span data-ttu-id="2dad7-179">Cambie el **color** a **color gris oscuro**y cambie la **fuente** a **sistema 12,0**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-179">Change the **Color** to **Dark Gray Color**, and change the **Font** to **System 12.0**.</span></span>
1. <span data-ttu-id="2dad7-180">Seleccione el botón y, a continuación, seleccione el **Inspector de atributos**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-180">Select the button, then select the **Attributes Inspector**.</span></span>
1. <span data-ttu-id="2dad7-181">Cambie el **título** a `Sign Out`.</span><span class="sxs-lookup"><span data-stu-id="2dad7-181">Change the **Title** to `Sign Out`.</span></span>
1. <span data-ttu-id="2dad7-182">Mediante el **Inspector de conexiones**, realice las siguientes conexiones.</span><span class="sxs-lookup"><span data-stu-id="2dad7-182">Using the **Connections Inspector**, make the following connections.</span></span>

    - <span data-ttu-id="2dad7-183">Vincule el tomacorriente **userDisplayName** a la primera etiqueta.</span><span class="sxs-lookup"><span data-stu-id="2dad7-183">Link the **userDisplayName** outlet to the first label.</span></span>
    - <span data-ttu-id="2dad7-184">Vincule la toma de **userEmail** a la segunda etiqueta.</span><span class="sxs-lookup"><span data-stu-id="2dad7-184">Link the **userEmail** outlet to the second label.</span></span>
    - <span data-ttu-id="2dad7-185">Vincule el tomacorriente **userProfilePhoto** a la vista de imagen.</span><span class="sxs-lookup"><span data-stu-id="2dad7-185">Link the **userProfilePhoto** outlet to the image view.</span></span>
    - <span data-ttu-id="2dad7-186">Vincular la acción de **signOut** recibido a la **pantalla táctil**del botón en.</span><span class="sxs-lookup"><span data-stu-id="2dad7-186">Link the **signOut** received action to the button's **Touch Up Inside**.</span></span>

1. <span data-ttu-id="2dad7-187">Seleccione el elemento de la barra de pestañas en la parte inferior de la escena y, a continuación, seleccione el **Inspector de atributos**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-187">Select the tab bar item at the bottom of the scene, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="2dad7-188">Cambie el **título** a `Me`.</span><span class="sxs-lookup"><span data-stu-id="2dad7-188">Change the **Title** to `Me`.</span></span>
1. <span data-ttu-id="2dad7-189">En el menú **Editor** , seleccione **resolver problemas de diseño automático**y, a continuación, seleccione **Agregar restricciones que faltan** debajo **de todas las vistas en el controlador de vista de bienvenida**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-189">On the **Editor** menu, select **Resolve Auto Layout Issues**, then select **Add Missing Constraints** underneath **All Views in Welcome View Controller**.</span></span>

<span data-ttu-id="2dad7-190">La escena de bienvenida debe tener una apariencia similar a la siguiente una vez que haya terminado.</span><span class="sxs-lookup"><span data-stu-id="2dad7-190">The welcome scene should look similar to this once you're done.</span></span>

![Captura de pantalla del diseño de la escena de bienvenida](./images/welcome-scene-layout.png)

### <a name="create-calendar-page"></a><span data-ttu-id="2dad7-192">Crear página de calendario</span><span class="sxs-lookup"><span data-stu-id="2dad7-192">Create calendar page</span></span>

1. <span data-ttu-id="2dad7-193">Cree un nuevo archivo de **clase táctil de cacao** en la carpeta `CalendarViewController` **GraphTutorial** denominada.</span><span class="sxs-lookup"><span data-stu-id="2dad7-193">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `CalendarViewController`.</span></span> <span data-ttu-id="2dad7-194">Elija **UIViewController** en el campo **subclase de** .</span><span class="sxs-lookup"><span data-stu-id="2dad7-194">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="2dad7-195">Abra **CalendarViewController. h** y agregue el siguiente código dentro de `@interface` la declaración.</span><span class="sxs-lookup"><span data-stu-id="2dad7-195">Open **CalendarViewController.h** and add the following code inside the `@interface` declaration.</span></span>

    ```objc
    @property (nonatomic) IBOutlet UITextView *calendarJSON;
    ```

1. <span data-ttu-id="2dad7-196">Abra **CalendarViewController. m** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="2dad7-196">Open **CalendarViewController.m** and replace its contents with the following code.</span></span>

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

1. <span data-ttu-id="2dad7-197">Abra **Main. Storyboard**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-197">Open **Main.storyboard**.</span></span> <span data-ttu-id="2dad7-198">Seleccione la **escena elemento 2**y, a continuación, seleccione el **Inspector de identidad**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-198">Select the **Item 2 Scene**, then select the **Identity Inspector**.</span></span> <span data-ttu-id="2dad7-199">Cambie el valor de la **clase** a **CalendarViewController**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-199">Change the **Class** value to **CalendarViewController**.</span></span>
1. <span data-ttu-id="2dad7-200">Con la **biblioteca**, agregue una **vista de texto** a la **escena del elemento 2**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-200">Using the **Library**, add a **Text View** to the **Item 2 Scene**.</span></span>
1. <span data-ttu-id="2dad7-201">Seleccione la vista de texto que acaba de agregar.</span><span class="sxs-lookup"><span data-stu-id="2dad7-201">Select the text view you just added.</span></span> <span data-ttu-id="2dad7-202">En el **Editor**, elija **incrustar en**y, a continuación, **vista de desplazamiento**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-202">On the **Editor**, choose **Embed In**, then **Scroll View**.</span></span>
1. <span data-ttu-id="2dad7-203">Mediante el **Inspector de conexiones**, conecte el tomacorriente **calendarJSON** a la vista de texto.</span><span class="sxs-lookup"><span data-stu-id="2dad7-203">Using the **Connections Inspector**, connect the **calendarJSON** outlet to the text view.</span></span>
1. 1. <span data-ttu-id="2dad7-204">Seleccione el elemento de la barra de pestañas en la parte inferior de la escena y, a continuación, seleccione el **Inspector de atributos**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-204">Select the tab bar item at the bottom of the scene, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="2dad7-205">Cambie el **título** a `Calendar`.</span><span class="sxs-lookup"><span data-stu-id="2dad7-205">Change the **Title** to `Calendar`.</span></span>
1. <span data-ttu-id="2dad7-206">En el menú **Editor** , seleccione **resolver problemas de diseño automático**y, a continuación, seleccione **Agregar restricciones que faltan** debajo **de todas las vistas en el controlador de vista de bienvenida**.</span><span class="sxs-lookup"><span data-stu-id="2dad7-206">On the **Editor** menu, select **Resolve Auto Layout Issues**, then select **Add Missing Constraints** underneath **All Views in Welcome View Controller**.</span></span>

<span data-ttu-id="2dad7-207">La escena del calendario debe tener un aspecto similar a este, una vez que haya terminado.</span><span class="sxs-lookup"><span data-stu-id="2dad7-207">The calendar scene should look similar to this once you're done.</span></span>

![Captura de pantalla del diseño de la escena de calendario](./images/calendar-scene-layout.png)

### <a name="create-activity-indicator"></a><span data-ttu-id="2dad7-209">Crear indicador de actividad</span><span class="sxs-lookup"><span data-stu-id="2dad7-209">Create activity indicator</span></span>

1. <span data-ttu-id="2dad7-210">Cree un nuevo archivo de **clase táctil de cacao** en la carpeta `SpinnerViewController` **GraphTutorial** denominada.</span><span class="sxs-lookup"><span data-stu-id="2dad7-210">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `SpinnerViewController`.</span></span> <span data-ttu-id="2dad7-211">Elija **UIViewController** en el campo **subclase de** .</span><span class="sxs-lookup"><span data-stu-id="2dad7-211">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="2dad7-212">Abra **SpinnerViewController. h** y agregue el siguiente código dentro de `@interface` la declaración.</span><span class="sxs-lookup"><span data-stu-id="2dad7-212">Open **SpinnerViewController.h** and add the following code inside the `@interface` declaration.</span></span>

    ```objc
    - (void) startWithContainer:(UIViewController*) container;
    - (void) stop;
    ```

1. <span data-ttu-id="2dad7-213">Abra **SpinnerViewController. m** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="2dad7-213">Open **SpinnerViewController.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "SpinnerViewController.h"

    @interface SpinnerViewController ()
    @property (nonatomic) UIActivityIndicatorView* spinner;
    @end

    @implementation SpinnerViewController

    - (void)viewDidLoad {
        [super viewDidLoad];

        _spinner = [[UIActivityIndicatorView alloc] initWithActivityIndicatorStyle:
                    UIActivityIndicatorViewStyleWhiteLarge];

        self.view.backgroundColor = [UIColor colorWithWhite:0 alpha:0.7];
        [self.view addSubview:_spinner];

        _spinner.translatesAutoresizingMaskIntoConstraints = false;
        [_spinner startAnimating];

        [_spinner.centerXAnchor constraintEqualToAnchor:self.view.centerXAnchor].active = true;
        [_spinner.centerYAnchor constraintEqualToAnchor:self.view.centerYAnchor].active = true;
    }

    - (void) startWithContainer:(UIViewController *)container {
        [container addChildViewController:self];
        self.view.frame = container.view.frame;
        [container.view addSubview:self.view];
        [self didMoveToParentViewController:container];
    }

    - (void) stop {
        [self willMoveToParentViewController:nil];
        [self.view removeFromSuperview];
        [self removeFromParentViewController];
    }

    @end
    ```

## <a name="test-the-app"></a><span data-ttu-id="2dad7-214">Probar la aplicación</span><span class="sxs-lookup"><span data-stu-id="2dad7-214">Test the app</span></span>

<span data-ttu-id="2dad7-215">Guarde los cambios e inicie la aplicación.</span><span class="sxs-lookup"><span data-stu-id="2dad7-215">Save your changes and launch the app.</span></span> <span data-ttu-id="2dad7-216">Debe poder moverse entre las pantallas con los botones **iniciar sesión** y **Cerrar sesión** y la barra de pestañas.</span><span class="sxs-lookup"><span data-stu-id="2dad7-216">You should be able to move between the screens using the **Sign In** and **Sign Out** buttons and the tab bar.</span></span>

![Capturas de pantallas de la aplicación](./images/app-screens.png)
