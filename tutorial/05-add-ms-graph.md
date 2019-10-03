<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="0b6a0-101">En este ejercicio, incorporará Microsoft Graph a la aplicación.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="0b6a0-102">Para esta aplicación, usará el [SDK de Microsoft Graph para Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) para realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-102">For this application, you will use the [Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="0b6a0-103">Obtener eventos de calendario de Outlook</span><span class="sxs-lookup"><span data-stu-id="0b6a0-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="0b6a0-104">En esta sección, ampliará la `GraphManager` clase para agregar una función para obtener los eventos del usuario y actualizar `CalendarViewController` para usar estas funciones nuevas.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-104">In this section you will extend the `GraphManager` class to add a function to get the user's events and update `CalendarViewController` to use these new functions.</span></span>

1. <span data-ttu-id="0b6a0-105">Abra **GraphManager. h** y agregue el siguiente código sobre la `@interface` declaración.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-105">Open **GraphManager.h** and add the following code above the `@interface` declaration.</span></span>

    ```objc
    typedef void (^GetEventsCompletionBlock)(NSData* _Nullable data, NSError* _Nullable error);
    ```

1. <span data-ttu-id="0b6a0-106">Agregue el siguiente código a la `@interface` declaración.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-106">Add the following code to the `@interface` declaration.</span></span>

    ```objc
    - (void) getEventsWithCompletionBlock: (GetEventsCompletionBlock)completionBlock;
    ```

1. <span data-ttu-id="0b6a0-107">Abra **GraphManager. m** y agregue la siguiente función a la `GraphManager` clase.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-107">Open **GraphManager.m** and add the following function to the `GraphManager` class.</span></span>

    ```objc
    - (void) getEventsWithCompletionBlock:(GetEventsCompletionBlock)completionBlock {
        // GET /me/events?$select='subject,organizer,start,end'$orderby=createdDateTime DESC
        NSString* eventsUrlString =
        [NSString stringWithFormat:@"%@/me/events?%@&%@",
         MSGraphBaseURL,
         // Only return these fields in results
         @"$select=subject,organizer,start,end",
         // Sort results by when they were created, newest first
         @"$orderby=createdDateTime+DESC"];

        NSURL* eventsUrl = [[NSURL alloc] initWithString:eventsUrlString];
        NSMutableURLRequest* eventsRequest = [[NSMutableURLRequest alloc] initWithURL:eventsUrl];

        MSURLSessionDataTask* eventsDataTask =
        [[MSURLSessionDataTask alloc]
         initWithRequest:eventsRequest
         client:self.graphClient
         completion:^(NSData *data, NSURLResponse *response, NSError *error) {
             if (error) {
                 completionBlock(nil, error);
                 return;
             }

             // TEMPORARY
             completionBlock(data, nil);
         }];

        // Execute the request
        [eventsDataTask execute];
    }
    ```

    > [!NOTE]
    > <span data-ttu-id="0b6a0-108">Tenga en cuenta lo que `getEventsWithCompletionBlock` hace el código.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-108">Consider what the code in `getEventsWithCompletionBlock` is doing.</span></span>
    >
    > - <span data-ttu-id="0b6a0-109">La dirección URL a la que se `/v1.0/me/events`llamará es.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-109">The URL that will be called is `/v1.0/me/events`.</span></span>
    > - <span data-ttu-id="0b6a0-110">El `select` parámetro de consulta limita los campos devueltos para cada evento a solo aquellos que la aplicación usará realmente.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-110">The `select` query parameter limits the fields returned for each events to just those the app will actually use.</span></span>
    > - <span data-ttu-id="0b6a0-111">El `orderby` parámetro de consulta ordena los resultados por la fecha y hora en que se crearon, con el elemento más reciente en primer lugar.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-111">The `orderby` query parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="0b6a0-112">Abra **CalendarViewController. m** y reemplace todo su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-112">Open **CalendarViewController.m** and replace its entire contents with the following code.</span></span>

    ```objc
    #import "CalendarViewController.h"
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>

    @interface CalendarViewController ()

    @property SpinnerViewController* spinner;

    @end

    @implementation CalendarViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        self.spinner = [SpinnerViewController alloc];
        [self.spinner startWithContainer:self];

        [GraphManager.instance
         getEventsWithCompletionBlock:^(NSData * _Nullable data, NSError * _Nullable error) {
             dispatch_async(dispatch_get_main_queue(), ^{
                 [self.spinner stop];

                 if (error) {
                     // Show the error
                     UIAlertController* alert = [UIAlertController
                                                 alertControllerWithTitle:@"Error getting events"
                                                 message:error.debugDescription
                                                 preferredStyle:UIAlertControllerStyleAlert];

                     UIAlertAction* okButton = [UIAlertAction
                                                actionWithTitle:@"OK"
                                                style:UIAlertActionStyleDefault
                                                handler:nil];

                     [alert addAction:okButton];
                     [self presentViewController:alert animated:true completion:nil];
                     return;
                 }

                 // TEMPORARY
                 self.calendarJSON.text = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
                 [self.calendarJSON sizeToFit];
             });
         }];
    }

    @end
    ```

<span data-ttu-id="0b6a0-113">Ahora puede ejecutar la aplicación, iniciar sesión y puntear en el elemento de navegación **calendario** del menú.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-113">You can now run the app, sign in, and tap the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="0b6a0-114">Debería ver un volcado JSON de los eventos en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-114">You should see a JSON dump of the events in the app.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="0b6a0-115">Mostrar los resultados</span><span class="sxs-lookup"><span data-stu-id="0b6a0-115">Display the results</span></span>

<span data-ttu-id="0b6a0-116">Ahora puede reemplazar el volcado de JSON con algo para mostrar los resultados de forma fácil de uso.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-116">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="0b6a0-117">En esta sección, modificará la `getEventsWithCompletionBlock` función para que devuelva objetos con establecimiento inflexible de tipos `CalendarViewController` y lo modifique para usar una vista de tabla para representar los eventos.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-117">In this section, you will modify the `getEventsWithCompletionBlock` function to return strongly-typed objects, and modify `CalendarViewController` to use a table view to render the events.</span></span>

### <a name="update-getevents"></a><span data-ttu-id="0b6a0-118">Actualizar getEvents</span><span class="sxs-lookup"><span data-stu-id="0b6a0-118">Update getEvents</span></span>

1. <span data-ttu-id="0b6a0-119">Abra **GraphManager. h**.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-119">Open **GraphManager.h**.</span></span> <span data-ttu-id="0b6a0-120">Cambie la `GetEventsCompletionBlock` definición de tipo por la siguiente.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-120">Change the `GetEventsCompletionBlock` type definition to the following.</span></span>

    ```objc
    typedef void (^GetEventsCompletionBlock)(NSArray<MSGraphEvent*>* _Nullable events, NSError* _Nullable error);
    ```

1. <span data-ttu-id="0b6a0-121">Abra **GraphManager. m**.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-121">Open **GraphManager.m**.</span></span> <span data-ttu-id="0b6a0-122">Reemplace la `completionBlock(data, nil);` línea en la `getEventsWithCompletionBlock` función por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-122">Replace the `completionBlock(data, nil);` line in the `getEventsWithCompletionBlock` function with the following code.</span></span>

    ```objc
    NSError* graphError;

    // Deserialize to an events collection
    MSCollection* eventsCollection = [[MSCollection alloc] initWithData:data error:&graphError];
    if (graphError) {
        completionBlock(nil, graphError);
        return;
    }

    // Create an array to return
    NSMutableArray* eventsArray = [[NSMutableArray alloc]
                                initWithCapacity:eventsCollection.value.count];

    for (id event in eventsCollection.value) {
        // Deserialize the event and add to the array
        MSGraphEvent* graphEvent = [[MSGraphEvent alloc] initWithDictionary:event];
        [eventsArray addObject:graphEvent];
    }

    completionBlock(eventsArray, nil);
    ```

### <a name="update-calendarviewcontroller"></a><span data-ttu-id="0b6a0-123">Actualizar CalendarViewController</span><span class="sxs-lookup"><span data-stu-id="0b6a0-123">Update CalendarViewController</span></span>

1. <span data-ttu-id="0b6a0-124">Cree un nuevo archivo de **clase táctil de cacao** en el proyecto `CalendarTableViewCell` **GraphTutorial** denominado.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-124">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewCell`.</span></span> <span data-ttu-id="0b6a0-125">Elija **UITableViewCell** en el campo **subclase de** .</span><span class="sxs-lookup"><span data-stu-id="0b6a0-125">Choose **UITableViewCell** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="0b6a0-126">Abra **CalendarTableViewCell. h** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-126">Open **CalendarTableViewCell.h** and replace its contents with the following code.</span></span>

    ```objc
    #import <UIKit/UIKit.h>

    NS_ASSUME_NONNULL_BEGIN

    @interface CalendarTableViewCell : UITableViewCell

    @property (nonatomic) NSString* subject;
    @property (nonatomic) NSString* organizer;
    @property (nonatomic) NSString* duration;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. <span data-ttu-id="0b6a0-127">Abra **CalendarTableViewCell. m** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-127">Open **CalendarTableViewCell.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "CalendarTableViewCell.h"

    @interface CalendarTableViewCell()

    @property (nonatomic) IBOutlet UILabel *subjectLabel;
    @property (nonatomic) IBOutlet UILabel *organizerLabel;
    @property (nonatomic) IBOutlet UILabel *durationLabel;

    @end

    @implementation CalendarTableViewCell

    - (void)awakeFromNib {
        [super awakeFromNib];
        // Initialization code
    }

    - (void)setSelected:(BOOL)selected animated:(BOOL)animated {
        [super setSelected:selected animated:animated];

        // Configure the view for the selected state
    }

    - (void) setSubject:(NSString *)subject {
        _subject = subject;
        self.subjectLabel.text = subject;
        [self.subjectLabel sizeToFit];
    }

    - (void) setOrganizer:(NSString *)organizer {
        _organizer = organizer;
        self.organizerLabel.text = organizer;
        [self.organizerLabel sizeToFit];
    }

    - (void) setDuration:(NSString *)duration {
        _duration = duration;
        self.durationLabel.text = duration;
        [self.durationLabel sizeToFit];
    }

    @end
    ```

1. <span data-ttu-id="0b6a0-128">Abra **Main. Storyboard** y busque la **escena del calendario**.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-128">Open **Main.storyboard** and locate the **Calendar Scene**.</span></span> <span data-ttu-id="0b6a0-129">Seleccione la **vista** de la **escena del calendario** y elimínela.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-129">Select the **View** in the **Calendar Scene** and delete it.</span></span>

    ![Captura de pantalla de la vista en la escena del calendario](./images/view-in-calendar-scene.png)

1. <span data-ttu-id="0b6a0-131">Agregue una **vista de tabla** de la **biblioteca** a la **escena del calendario**.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-131">Add a **Table View** from the **Library** to the **Calendar Scene**.</span></span>
1. <span data-ttu-id="0b6a0-132">Seleccione la vista de tabla y, a continuación, seleccione el **Inspector de atributos**.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-132">Select the table view, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="0b6a0-133">Establezca **las celdas del prototipo** en **1**.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-133">Set **Prototype Cells** to **1**.</span></span>
1. <span data-ttu-id="0b6a0-134">Use la **biblioteca** para agregar tres **etiquetas** a la celda del prototipo.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-134">Use the **Library** to add three **Labels** to the prototype cell.</span></span>
1. <span data-ttu-id="0b6a0-135">Seleccione la celda prototype y, a continuación, seleccione el **Inspector de identidad**.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-135">Select the prototype cell, then select the **Identity Inspector**.</span></span> <span data-ttu-id="0b6a0-136">Cambie la **clase** a **CalendarTableViewCell**.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-136">Change **Class** to **CalendarTableViewCell**.</span></span>
1. <span data-ttu-id="0b6a0-137">Seleccione el **Inspector de atributos** y establezca **identificador** en `EventCell`.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-137">Select the **Attributes Inspector** and set **Identifier** to `EventCell`.</span></span>
1. <span data-ttu-id="0b6a0-138">En el menú **Editor** , seleccione **resolver problemas de diseño automático**y, a continuación, seleccione **Agregar restricciones que faltan** debajo **de todas las vistas en el controlador de vista de bienvenida**.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-138">On the **Editor** menu, select **Resolve Auto Layout Issues**, then select **Add Missing Constraints** underneath **All Views in Welcome View Controller**.</span></span>
1. <span data-ttu-id="0b6a0-139">Con la **EventCell** seleccionada, seleccione el **Inspector de conexiones** y `durationLabel`Conéctese `organizerLabel`, `subjectLabel` y a las etiquetas que haya agregado a la celda en el guión gráfico.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-139">With the **EventCell** selected, select the **Connections Inspector** and connect `durationLabel`, `organizerLabel`, and `subjectLabel` to the labels you added to the cell on the storyboard.</span></span>

    ![Captura de pantalla del diseño de celda de prototipo](./images/prototype-cell-layout.png)

1. <span data-ttu-id="0b6a0-141">Abra **CalendarViewController. h** y quite la `calendarJSON` propiedad.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-141">Open **CalendarViewController.h** and remove the `calendarJSON` property.</span></span>
1. <span data-ttu-id="0b6a0-142">Cambie la `@interface` declaración a la siguiente.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-142">Change the `@interface` declaration to the following.</span></span>

    ```objc
    @interface CalendarViewController : UITableViewController
    ```

1. <span data-ttu-id="0b6a0-143">Abra **CalendarViewController. m** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-143">Open **CalendarViewController.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "WelcomeViewController.h"
    #import "SpinnerViewController.h"
    #import "AuthenticationManager.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>

    @interface WelcomeViewController ()

    @property SpinnerViewController* spinner;

    @end

    @implementation WelcomeViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        self.spinner = [SpinnerViewController alloc];
        [self.spinner startWithContainer:self];

        self.userProfilePhoto.image = [UIImage imageNamed:@"DefaultUserPhoto"];

        [GraphManager.instance
         getMeWithCompletionBlock:^(MSGraphUser * _Nullable user, NSError * _Nullable error) {
            dispatch_async(dispatch_get_main_queue(), ^{
                [self.spinner stop];

                if (error) {
                    // Show the error
                    UIAlertController* alert = [UIAlertController
                                                alertControllerWithTitle:@"Error getting user profile"
                                                message:error.debugDescription
                                                preferredStyle:UIAlertControllerStyleAlert];

                    UIAlertAction* okButton = [UIAlertAction
                                               actionWithTitle:@"OK"
                                               style:UIAlertActionStyleDefault
                                               handler:nil];

                    [alert addAction:okButton];
                    [self presentViewController:alert animated:true completion:nil];
                    return;
                }

                // Set display name
                self.userDisplayName.text = user.displayName ? : @"Mysterious Stranger";
                [self.userDisplayName sizeToFit];

                // AAD users have email in the mail attribute
                // Personal accounts have email in the userPrincipalName attribute
                self.userEmail.text = user.mail ? : user.userPrincipalName;
                [self.userEmail sizeToFit];
            });
         }];
    }

    - (IBAction)signOut {
        [AuthenticationManager.instance signOut];
        [self performSegueWithIdentifier: @"userSignedOut" sender: nil];
    }

    @end
    ```

1. <span data-ttu-id="0b6a0-144">Ejecute la aplicación, inicie sesión y pulse la pestaña **calendario** . Debe ver la lista de eventos.</span><span class="sxs-lookup"><span data-stu-id="0b6a0-144">Run the app, sign in, and tap the **Calendar** tab. You should see the list of events.</span></span>

    ![Captura de pantalla de la tabla de eventos](./images/calendar-list.png)
