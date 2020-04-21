# <a name="how-to-run-the-completed-project"></a><span data-ttu-id="6b1e6-101">Cómo ejecutar el proyecto completado</span><span class="sxs-lookup"><span data-stu-id="6b1e6-101">How to run the completed project</span></span>

## <a name="prerequisites"></a><span data-ttu-id="6b1e6-102">Requisitos previos</span><span class="sxs-lookup"><span data-stu-id="6b1e6-102">Prerequisites</span></span>

<span data-ttu-id="6b1e6-103">Para ejecutar el proyecto completado en esta carpeta, necesita lo siguiente:</span><span class="sxs-lookup"><span data-stu-id="6b1e6-103">To run the completed project in this folder, you need the following:</span></span>

- [<span data-ttu-id="6b1e6-104">Xcode</span><span class="sxs-lookup"><span data-stu-id="6b1e6-104">Xcode</span></span>](https://developer.apple.com/xcode/)
- [<span data-ttu-id="6b1e6-105">CocoaPods</span><span class="sxs-lookup"><span data-stu-id="6b1e6-105">CocoaPods</span></span>](https://cocoapods.org)
- <span data-ttu-id="6b1e6-106">Una cuenta de Microsoft personal con un buzón de correo en Outlook.com o una cuenta profesional o educativa de Microsoft.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-106">Either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span>

> <span data-ttu-id="6b1e6-107">**Nota:** Este tutorial se ha escrito con la versión 11,4 de Xcode y la versión 1.9.1 de CocoaPods.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-107">**Note:** This tutorial was written using Xcode version 11.4 and CocoaPods version 1.9.1.</span></span> <span data-ttu-id="6b1e6-108">Los pasos de esta guía pueden funcionar con otras versiones, pero no se han probado.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-108">The steps in this guide may work with other versions, but that has not been tested.</span></span>

<span data-ttu-id="6b1e6-109">Si no tiene una cuenta de Microsoft, hay un par de opciones para obtener una cuenta gratuita:</span><span class="sxs-lookup"><span data-stu-id="6b1e6-109">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="6b1e6-110">Puede [registrarse para obtener una nueva cuenta Microsoft personal](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span><span class="sxs-lookup"><span data-stu-id="6b1e6-110">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="6b1e6-111">Puede [registrarse para el programa de desarrolladores de office 365](https://developer.microsoft.com/office/dev-program) para obtener una suscripción gratuita a Office 365.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-111">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

## <a name="register-a-web-application-with-the-azure-active-directory-admin-center"></a><span data-ttu-id="6b1e6-112">Registro de una aplicación web con el centro de administración de Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="6b1e6-112">Register a web application with the Azure Active Directory admin center</span></span>

1. <span data-ttu-id="6b1e6-113">Abra un explorador y vaya al [centro de administración de Azure Active Directory](https://aad.portal.azure.com).</span><span class="sxs-lookup"><span data-stu-id="6b1e6-113">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com).</span></span> <span data-ttu-id="6b1e6-114">Inicie sesión con una **cuenta personal** (también conocida como: cuenta Microsoft) o una **cuenta profesional o educativa**.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-114">Login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="6b1e6-115">Seleccione **Azure Active Directory** en el panel de navegación de la izquierda y, después, seleccione **registros de aplicaciones** en **administrar**.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-115">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations** under **Manage**.</span></span>

    ![<span data-ttu-id="6b1e6-116">Una captura de pantalla de los registros de la aplicación</span><span class="sxs-lookup"><span data-stu-id="6b1e6-116">A screenshot of the App registrations</span></span> ](/tutorial/images/aad-portal-app-registrations.png)

1. <span data-ttu-id="6b1e6-117">Seleccione **Nuevo registro**.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-117">Select **New registration**.</span></span> <span data-ttu-id="6b1e6-118">En la página **Registrar una aplicación**, establezca los valores siguientes.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-118">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="6b1e6-119">Establezca **Nombre** como `iOS Objective-C Graph Tutorial`.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-119">Set **Name** to `iOS Objective-C Graph Tutorial`.</span></span>
    - <span data-ttu-id="6b1e6-120">Establezca **Tipos de cuenta admitidos** en **Cuentas en cualquier directorio de organización y cuentas personales de Microsoft**.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-120">Set **Supported account types** to **Accounts in any organizational directory and personal Microsoft accounts**.</span></span>
    - <span data-ttu-id="6b1e6-121">Deje **URI de redireccionamiento** vacía.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-121">Leave **Redirect URI** empty.</span></span>

    ![Captura de pantalla de la página registrar una aplicación](/tutorial/images/aad-register-an-app.png)

1. <span data-ttu-id="6b1e6-123">Elija **Registrar**.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-123">Choose **Register**.</span></span> <span data-ttu-id="6b1e6-124">En la página Tutorial de los **gráficos de iOS Objective-C** , copie el valor del identificador de la **aplicación (cliente)** y guárdelo, lo necesitará en el paso siguiente.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-124">On the **iOS Objective-C Graph Tutorial** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

    ![Captura de pantalla del identificador de la aplicación del nuevo registro de la aplicación](/tutorial/images/aad-application-id.png)

1. <span data-ttu-id="6b1e6-126">Seleccione **Autenticación** en **Administrar**.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-126">Select **Authentication** under **Manage**.</span></span> <span data-ttu-id="6b1e6-127">Seleccione **Agregar una plataforma**y, a continuación, **iOS/MacOS**.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-127">Select **Add a platform**, then **iOS / macOS**.</span></span>

1. <span data-ttu-id="6b1e6-128">Escriba el identificador de paquete de la aplicación y seleccione **configurar**y, después, haga clic en **listo**.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-128">Enter your app's Bundle ID and select **Configure**, then select **Done**.</span></span>

## <a name="configure-the-sample"></a><span data-ttu-id="6b1e6-129">Configuración del ejemplo</span><span class="sxs-lookup"><span data-stu-id="6b1e6-129">Configure the sample</span></span>

1. <span data-ttu-id="6b1e6-130">Cambie el nombre `AuthSettings.plist.example` del archivo `AuthSettings.plist`a.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-130">Rename the `AuthSettings.plist.example` file to `AuthSettings.plist`.</span></span>
1. <span data-ttu-id="6b1e6-131">Edite `AuthSettings.plist` el archivo y realice los cambios siguientes.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-131">Edit the `AuthSettings.plist` file and make the following changes.</span></span>
    1. <span data-ttu-id="6b1e6-132">Reemplace `YOUR_APP_ID_HERE` por el **identificador de aplicación** que obtuvo desde el portal de registro de aplicaciones.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-132">Replace `YOUR_APP_ID_HERE` with the **Application Id** you got from the App Registration Portal.</span></span>
1. <span data-ttu-id="6b1e6-133">Abra terminal en el directorio **GraphTutorial** y ejecute el siguiente comando.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-133">Open Terminal in the **GraphTutorial** directory and run the following command.</span></span>

    ```Shell
    pod install
    ```

## <a name="run-the-sample"></a><span data-ttu-id="6b1e6-134">Ejecutar el ejemplo</span><span class="sxs-lookup"><span data-stu-id="6b1e6-134">Run the sample</span></span>

<span data-ttu-id="6b1e6-135">En Xcode, seleccione el menú **producto** y, después, haga clic en **Ejecutar**.</span><span class="sxs-lookup"><span data-stu-id="6b1e6-135">In Xcode, select the **Product** menu, then select **Run**.</span></span>
