# <a name="how-to-run-the-completed-project"></a>Cómo ejecutar el proyecto completado

## <a name="prerequisites"></a>Requisitos previos

Para ejecutar el proyecto completado en esta carpeta, necesita lo siguiente:

- [Xcode](https://developer.apple.com/xcode/)
- [CocoaPods](https://cocoapods.org)
- Una cuenta de Microsoft personal con un buzón de correo en Outlook.com o una cuenta profesional o educativa de Microsoft.

> **Nota:** Este tutorial se ha escrito con la versión 10,3 de Xcode y la versión 1.7.5 de CocoaPods. Los pasos de esta guía pueden funcionar con otras versiones, pero no se han probado.

Si no tiene una cuenta de Microsoft, hay un par de opciones para obtener una cuenta gratuita:

- Puede [registrarse para obtener una nueva cuenta Microsoft personal](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).
- Puede [registrarse para el programa de desarrolladores de office 365](https://developer.microsoft.com/office/dev-program) para obtener una suscripción gratuita a Office 365.

## <a name="register-a-web-application-with-the-azure-active-directory-admin-center"></a>Registro de una aplicación web con el centro de administración de Azure Active Directory

1. Abra un explorador y vaya al [centro de administración de Azure Active Directory](https://aad.portal.azure.com). Inicie sesión con una **cuenta personal** (también conocida como: cuenta Microsoft) o una **cuenta profesional o educativa**.

1. Seleccione **Azure Active Directory** en el panel de navegación de la izquierda y, después, seleccione **registros de aplicaciones** en **administrar**.

    ![Una captura de pantalla de los registros de la aplicación ](/tutorial/images/aad-portal-app-registrations.png)

1. Seleccione **Nuevo registro**. En la página **Registrar una aplicación**, establezca los valores siguientes.

    - Establezca **Nombre** como `iOS Objective-C Graph Tutorial`.
    - Establezca **Tipos de cuenta admitidos** en **Cuentas en cualquier directorio de organización y cuentas personales de Microsoft**.
    - En **URI de redireccionamiento**, cambie la lista desplegable a **cliente público (móvil & escritorio)** y establezca `msauth.YOUR_BUNDLE_ID://auth`el valor `YOUR_BUNDLE_ID` en, reemplazando con el identificador de la agrupación de la aplicación.

    ![Captura de pantalla de la página registrar una aplicación](/tutorial/images/aad-register-an-app.png)

1. Elija **Registrar**. En la página Tutorial de los **gráficos de iOS Objective-C** , copie el valor del identificador de la **aplicación (cliente)** y guárdelo, lo necesitará en el paso siguiente.

    ![Captura de pantalla del identificador de la aplicación del nuevo registro de la aplicación](/tutorial/images/aad-application-id.png)

## <a name="configure-the-sample"></a>Configuración del ejemplo

1. Cambie el nombre `AuthSettings.plist.example` del archivo `AuthSettings.plist`a.
1. Edite `AuthSettings.plist` el archivo y realice los cambios siguientes.
    1. Reemplace `YOUR_APP_ID_HERE` por el **identificador de aplicación** que obtuvo desde el portal de registro de aplicaciones.
1. Abra terminal en el directorio **GraphTutorial** y ejecute el siguiente comando.

    ```Shell
    pod install
    ```

## <a name="run-the-sample"></a>Ejecutar el ejemplo

En Xcode, seleccione el menú **producto** y, después, haga clic en **Ejecutar**.
