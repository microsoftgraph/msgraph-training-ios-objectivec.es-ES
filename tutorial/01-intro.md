<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="ac595-101">En este tutorial se explica cómo crear una aplicación nativa de React que use la API de Microsoft Graph para recuperar información de calendario de un usuario.</span><span class="sxs-lookup"><span data-stu-id="ac595-101">This tutorial teaches you how to build an React Native app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="ac595-102">Si prefiere descargar el tutorial completado, puede descargar o clonar el [repositorio de GitHub.](https://github.com/microsoftgraph/msgraph-training-ios-objectivec)</span><span class="sxs-lookup"><span data-stu-id="ac595-102">If you prefer to just download the completed tutorial, you can download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-ios-objectivec).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="ac595-103">Requisitos previos</span><span class="sxs-lookup"><span data-stu-id="ac595-103">Prerequisites</span></span>

<span data-ttu-id="ac595-104">Antes de iniciar este tutorial, debes tener lo siguiente instalado en el equipo de desarrollo.</span><span class="sxs-lookup"><span data-stu-id="ac595-104">Before you start this tutorial, you should have the following installed on your development machine.</span></span>

- [<span data-ttu-id="ac595-105">Xcode</span><span class="sxs-lookup"><span data-stu-id="ac595-105">Xcode</span></span>](https://developer.apple.com/xcode/)
- [<span data-ttu-id="ac595-106">CocoaPods</span><span class="sxs-lookup"><span data-stu-id="ac595-106">CocoaPods</span></span>](https://cocoapods.org)

<span data-ttu-id="ac595-107">También debe tener una cuenta personal de Microsoft con un buzón en Outlook.com o una cuenta de Microsoft para el trabajo o la escuela.</span><span class="sxs-lookup"><span data-stu-id="ac595-107">You should also have either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span> <span data-ttu-id="ac595-108">Si no tienes una cuenta de Microsoft, hay un par de opciones para obtener una cuenta gratuita:</span><span class="sxs-lookup"><span data-stu-id="ac595-108">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="ac595-109">Puede registrarse [para obtener una nueva cuenta personal de Microsoft.](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)</span><span class="sxs-lookup"><span data-stu-id="ac595-109">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="ac595-110">Puede registrarse en el Programa de desarrolladores de [Office 365](https://developer.microsoft.com/office/dev-program) para obtener una suscripción gratuita a Office 365.</span><span class="sxs-lookup"><span data-stu-id="ac595-110">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="ac595-111">Este tutorial se escribió con Xcode versión 12.3 y CocoaPods versión 1.10.1.</span><span class="sxs-lookup"><span data-stu-id="ac595-111">This tutorial was written using Xcode version 12.3 and CocoaPods version 1.10.1.</span></span> <span data-ttu-id="ac595-112">Los pasos de esta guía pueden funcionar con otras versiones, pero no se han probado.</span><span class="sxs-lookup"><span data-stu-id="ac595-112">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="ac595-113">Comentarios</span><span class="sxs-lookup"><span data-stu-id="ac595-113">Feedback</span></span>

<span data-ttu-id="ac595-114">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-ios-objectivec).</span><span class="sxs-lookup"><span data-stu-id="ac595-114">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-ios-objectivec).</span></span>
