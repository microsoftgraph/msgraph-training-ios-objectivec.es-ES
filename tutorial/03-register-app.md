<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="52b36-101">En este ejercicio, creará una nueva aplicación nativa de Azure AD con el centro de administración de Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="52b36-101">In this exercise you will create a new Azure AD native application using the Azure Active Directory admin center.</span></span>

1. <span data-ttu-id="52b36-102">Abra un explorador y vaya al [centro de administración de Azure Active Directory](https://aad.portal.azure.com) e inicie sesión con una **cuenta personal** (también conocida como: cuenta Microsoft) o una **cuenta profesional o educativa**.</span><span class="sxs-lookup"><span data-stu-id="52b36-102">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com) and login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="52b36-103">Seleccione **Azure Active Directory** en el panel de navegación de la izquierda y, después, seleccione **registros de aplicaciones** en **administrar**.</span><span class="sxs-lookup"><span data-stu-id="52b36-103">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations** under **Manage**.</span></span>

    ![<span data-ttu-id="52b36-104">Una captura de pantalla de los registros de la aplicación</span><span class="sxs-lookup"><span data-stu-id="52b36-104">A screenshot of the App registrations</span></span> ](./images/aad-portal-app-registrations.png)

1. <span data-ttu-id="52b36-105">Seleccione **Nuevo registro**.</span><span class="sxs-lookup"><span data-stu-id="52b36-105">Select **New registration**.</span></span> <span data-ttu-id="52b36-106">En la página **Registrar una aplicación**, establezca los valores siguientes.</span><span class="sxs-lookup"><span data-stu-id="52b36-106">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="52b36-107">Establezca **Nombre** como `iOS Objective-C Graph Tutorial`.</span><span class="sxs-lookup"><span data-stu-id="52b36-107">Set **Name** to `iOS Objective-C Graph Tutorial`.</span></span>
    - <span data-ttu-id="52b36-108">Establezca **Tipos de cuenta admitidos** en **Cuentas en cualquier directorio de organización y cuentas personales de Microsoft**.</span><span class="sxs-lookup"><span data-stu-id="52b36-108">Set **Supported account types** to **Accounts in any organizational directory and personal Microsoft accounts**.</span></span>
    - <span data-ttu-id="52b36-109">Deje **URI de redireccionamiento** vacía.</span><span class="sxs-lookup"><span data-stu-id="52b36-109">Leave **Redirect URI** empty.</span></span>

    ![Captura de pantalla de la página registrar una aplicación](./images/aad-register-an-app.png)

1. <span data-ttu-id="52b36-111">Seleccione **registrar**.</span><span class="sxs-lookup"><span data-stu-id="52b36-111">Select **Register**.</span></span> <span data-ttu-id="52b36-112">En la página Tutorial de los **gráficos de iOS Objective-C** , copie el valor del identificador de la **aplicación (cliente)** y guárdelo, lo necesitará en el paso siguiente.</span><span class="sxs-lookup"><span data-stu-id="52b36-112">On the **iOS Objective-C Graph Tutorial** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

    ![Captura de pantalla del identificador de la aplicación del nuevo registro de la aplicación](./images/aad-application-id.png)

1. <span data-ttu-id="52b36-114">Seleccione **Autenticación** en **Administrar**.</span><span class="sxs-lookup"><span data-stu-id="52b36-114">Select **Authentication** under **Manage**.</span></span> <span data-ttu-id="52b36-115">Seleccione **Agregar una plataforma**y, a continuación, **iOS/MacOS**.</span><span class="sxs-lookup"><span data-stu-id="52b36-115">Select **Add a platform**, then **iOS / macOS**.</span></span>

1. <span data-ttu-id="52b36-116">Escriba el identificador de paquete de la aplicación y seleccione **configurar**y, después, haga clic en **listo**.</span><span class="sxs-lookup"><span data-stu-id="52b36-116">Enter your app's Bundle ID and select **Configure**, then select **Done**.</span></span>
