<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="2f0a7-101">Este tutorial le enseña a crear una aplicación nativa reAct que use la API de Microsoft Graph para recuperar la información de calendario de un usuario.</span><span class="sxs-lookup"><span data-stu-id="2f0a7-101">This tutorial teaches you how to build an React Native app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="2f0a7-102">Si prefiere descargar solo el tutorial completo, puede descargar o clonar el repositorio de [GitHub](https://github.com/microsoftgraph/msgraph-training-ios-objectivec).</span><span class="sxs-lookup"><span data-stu-id="2f0a7-102">If you prefer to just download the completed tutorial, you can download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-ios-objectivec).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="2f0a7-103">Requisitos previos</span><span class="sxs-lookup"><span data-stu-id="2f0a7-103">Prerequisites</span></span>

<span data-ttu-id="2f0a7-104">Antes de iniciar este tutorial, debe tener instalado lo siguiente en el equipo de desarrollo.</span><span class="sxs-lookup"><span data-stu-id="2f0a7-104">Before you start this tutorial, you should have the following installed on your development machine.</span></span>

- [<span data-ttu-id="2f0a7-105">Xcode</span><span class="sxs-lookup"><span data-stu-id="2f0a7-105">Xcode</span></span>](https://developer.apple.com/xcode/)
- [<span data-ttu-id="2f0a7-106">CocoaPods</span><span class="sxs-lookup"><span data-stu-id="2f0a7-106">CocoaPods</span></span>](https://cocoapods.org)

<span data-ttu-id="2f0a7-107">También debe tener una cuenta de Microsoft personal con un buzón de correo en Outlook.com o una cuenta profesional o educativa de Microsoft.</span><span class="sxs-lookup"><span data-stu-id="2f0a7-107">You should also have either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span> <span data-ttu-id="2f0a7-108">Si no tiene una cuenta de Microsoft, hay un par de opciones para obtener una cuenta gratuita:</span><span class="sxs-lookup"><span data-stu-id="2f0a7-108">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="2f0a7-109">Puede [registrarse para obtener una nueva cuenta Microsoft personal](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span><span class="sxs-lookup"><span data-stu-id="2f0a7-109">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="2f0a7-110">Puede [registrarse para el programa de desarrolladores de office 365](https://developer.microsoft.com/office/dev-program) para obtener una suscripción gratuita a Office 365.</span><span class="sxs-lookup"><span data-stu-id="2f0a7-110">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="2f0a7-111">Este tutorial se ha escrito con la versión 11,4 de Xcode y la versión 1.9.1 de CocoaPods.</span><span class="sxs-lookup"><span data-stu-id="2f0a7-111">This tutorial was written using Xcode version 11.4 and CocoaPods version 1.9.1.</span></span> <span data-ttu-id="2f0a7-112">Los pasos de esta guía pueden funcionar con otras versiones, pero no se han probado.</span><span class="sxs-lookup"><span data-stu-id="2f0a7-112">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="2f0a7-113">Comentarios</span><span class="sxs-lookup"><span data-stu-id="2f0a7-113">Feedback</span></span>

<span data-ttu-id="2f0a7-114">Envíe sus comentarios sobre este tutorial en el [repositorio de github](https://github.com/microsoftgraph/msgraph-training-ios-objectivec).</span><span class="sxs-lookup"><span data-stu-id="2f0a7-114">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-ios-objectivec).</span></span>
