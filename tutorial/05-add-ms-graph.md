<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="ef0c8-101">En este ejercicio incorporará Microsoft Graph en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="ef0c8-102">Para esta aplicación, usará el SDK de Microsoft Graph para [el Objetivo C](https://github.com/microsoftgraph/msgraph-sdk-objc) para realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-102">For this application, you will use the [Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="ef0c8-103">Obtener eventos de calendario de Outlook</span><span class="sxs-lookup"><span data-stu-id="ef0c8-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="ef0c8-104">En esta sección, ampliará la clase para agregar una función para obtener los eventos del usuario de la semana actual y actualizar para usar `GraphManager` `CalendarViewController` esta nueva función.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-104">In this section you will extend the `GraphManager` class to add a function to get the user's events for the current week and update `CalendarViewController` to use this new function.</span></span>

1. <span data-ttu-id="ef0c8-105">Abra **GraphManager.h y** agregue el siguiente código encima de la `@interface` declaración.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-105">Open **GraphManager.h** and add the following code above the `@interface` declaration.</span></span>

    ```objc
    typedef void (^GetCalendarViewCompletionBlock)(NSData* _Nullable data,
                                                   NSError* _Nullable error);
    ```

1. <span data-ttu-id="ef0c8-106">Agregue el siguiente código a la `@interface` declaración.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-106">Add the following code to the `@interface` declaration.</span></span>

    ```objc
    - (void) getCalendarViewStartingAt: (NSString*) viewStart
                              endingAt: (NSString*) viewEnd
                   withCompletionBlock: (GetCalendarViewCompletionBlock) completion;
    ```

1. <span data-ttu-id="ef0c8-107">Abra **GraphManager.m** y agregue la siguiente función a la `GraphManager` clase.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-107">Open **GraphManager.m** and add the following function to the `GraphManager` class.</span></span>

    ```objc
    - (void) getCalendarViewStartingAt: (NSString *) viewStart
                              endingAt: (NSString *) viewEnd
                   withCompletionBlock: (GetCalendarViewCompletionBlock) completion {
        // Set calendar view start and end parameters
        NSString* viewStartEndString =
        [NSString stringWithFormat:@"startDateTime=%@&endDateTime=%@",
         viewStart,
         viewEnd];

        // GET /me/calendarview
        NSString* eventsUrlString =
        [NSString stringWithFormat:@"%@/me/calendarview?%@&%@&%@&%@",
         MSGraphBaseURL,
         viewStartEndString,
         // Only return these fields in results
         @"$select=subject,organizer,start,end",
         // Sort results by start time
         @"$orderby=start/dateTime",
         // Request at most 25 results
         @"$top=25"];

        NSURL* eventsUrl = [[NSURL alloc] initWithString:eventsUrlString];
        NSMutableURLRequest* eventsRequest = [[NSMutableURLRequest alloc] initWithURL:eventsUrl];

        // Add the Prefer: outlook.timezone header to get start and end times
        // in user's time zone
        NSString* preferHeader =
        [NSString stringWithFormat:@"outlook.timezone=\"%@\"",
         self.graphTimeZone];
        [eventsRequest addValue:preferHeader forHTTPHeaderField:@"Prefer"];

        MSURLSessionDataTask* eventsDataTask =
        [[MSURLSessionDataTask alloc]
         initWithRequest:eventsRequest
         client:self.graphClient
         completion:^(NSData *data, NSURLResponse *response, NSError *error) {
             if (error) {
                 completion(nil, error);
                 return;
             }

             // TEMPORARY
             completion(data, nil);
         }];

        // Execute the request
        [eventsDataTask execute];
    }
    ```

    > [!NOTE]
    > <span data-ttu-id="ef0c8-108">Ten en cuenta lo que hace `getCalendarViewStartingAt` el código.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-108">Consider what the code in `getCalendarViewStartingAt` is doing.</span></span>
    >
    > - <span data-ttu-id="ef0c8-109">La dirección URL a la que se llamará es `/v1.0/me/calendarview` .</span><span class="sxs-lookup"><span data-stu-id="ef0c8-109">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
    >   - <span data-ttu-id="ef0c8-110">Los `startDateTime` parámetros de consulta y de consulta definen el inicio y el final de la vista de `endDateTime` calendario.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-110">The `startDateTime` and `endDateTime` query parameters define the start and end of the calendar view.</span></span>
    >   - <span data-ttu-id="ef0c8-111">El `select` parámetro de consulta limita los campos devueltos para cada evento a solo aquellos que la vista usará realmente.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-111">The `select` query parameter limits the fields returned for each events to just those the view will actually use.</span></span>
    >   - <span data-ttu-id="ef0c8-112">El `orderby` parámetro de consulta ordena los resultados por hora de inicio.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-112">The `orderby` query parameter sorts the results by start time.</span></span>
    >   - <span data-ttu-id="ef0c8-113">El `top` parámetro de consulta solicita 25 resultados por página.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-113">The `top` query parameter requests 25 results per page.</span></span>
    >   - <span data-ttu-id="ef0c8-114">el encabezado hace que Microsoft Graph devuelva las horas de inicio y finalización de cada evento en la `Prefer: outlook.timezone` zona horaria del usuario.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-114">the `Prefer: outlook.timezone` header causes the Microsoft Graph to return the start and end times of each event in the user's time zone.</span></span>

1. <span data-ttu-id="ef0c8-115">Cree una nueva **clase Cocoa Touch** en el proyecto **GraphTutorial** denominado **GraphToIana**.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-115">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **GraphToIana**.</span></span> <span data-ttu-id="ef0c8-116">Elija **NSObject** en la **subclase del** campo.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-116">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="ef0c8-117">Abra **GraphToIana.h** y reemplace su contenido por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-117">Open **GraphToIana.h** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.h" id="GraphToIanaSnippet":::

1. <span data-ttu-id="ef0c8-118">Abra **GraphToIana.m y** reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-118">Open **GraphToIana.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.m" id="GraphToIanaSnippet":::

    <span data-ttu-id="ef0c8-119">Esto hace una búsqueda simple para buscar un identificador de zona horaria IANA basado en el nombre de zona horaria devuelto por Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-119">This does a simple lookup to find an IANA time zone identifier based on the time zone name returned by Microsoft Graph.</span></span>

1. <span data-ttu-id="ef0c8-120">Abra **CalendarViewController.m y** reemplace todo su contenido por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-120">Open **CalendarViewController.m** and replace its entire contents with the following code.</span></span>

    ```objc
    #import "CalendarViewController.h"
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import "GraphToIana.h"
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

        // Calculate the start and end of the current week
        NSString* timeZoneId = [GraphToIana
                                getIanaIdentifierFromGraphIdentifier:
                                [GraphManager.instance graphTimeZone]];

        NSDate* now = [NSDate date];
        NSCalendar* calendar = [NSCalendar calendarWithIdentifier:NSCalendarIdentifierGregorian];
        NSTimeZone* timeZone = [NSTimeZone timeZoneWithName:timeZoneId];
        [calendar setTimeZone:timeZone];

        NSDateComponents* startOfWeekComponents = [calendar
                                                   components:NSCalendarUnitCalendar |
                                                   NSCalendarUnitYearForWeekOfYear |
                                                   NSCalendarUnitWeekOfYear
                                                   fromDate:now];
        NSDate* startOfWeek = [startOfWeekComponents date];
        NSDate* endOfWeek = [calendar dateByAddingUnit:NSCalendarUnitDay
                                                 value:7
                                                toDate:startOfWeek
                                               options:0];

        // Convert start and end to ISO 8601 strings
        NSISO8601DateFormatter* isoFormatter = [[NSISO8601DateFormatter alloc] init];
        NSString* viewStart = [isoFormatter stringFromDate:startOfWeek];
        NSString* viewEnd = [isoFormatter stringFromDate:endOfWeek];

        [GraphManager.instance
         getCalendarViewStartingAt:viewStart
         endingAt:viewEnd
         withCompletionBlock:^(NSData * _Nullable data, NSError * _Nullable error) {
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

1. <span data-ttu-id="ef0c8-121">Ejecuta la aplicación, inicia sesión y pulsa el **elemento de** navegación Calendario en el menú.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-121">Run the app, sign in, and tap the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="ef0c8-122">Debería ver un volcado JSON de los eventos en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-122">You should see a JSON dump of the events in the app.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="ef0c8-123">Mostrar los resultados</span><span class="sxs-lookup"><span data-stu-id="ef0c8-123">Display the results</span></span>

<span data-ttu-id="ef0c8-124">Ahora puede reemplazar el volcado json por algo para mostrar los resultados de forma fácil de usar.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-124">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="ef0c8-125">En esta sección, modificará la función para devolver objetos fuertemente especificados y modificará para usar una vista de tabla `getCalendarViewStartingAt` `CalendarViewController` para representar los eventos.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-125">In this section, you will modify the `getCalendarViewStartingAt` function to return strongly-typed objects, and modify `CalendarViewController` to use a table view to render the events.</span></span>

### <a name="update-getcalendarviewstartingat"></a><span data-ttu-id="ef0c8-126">Actualizar getCalendarViewStartingAt</span><span class="sxs-lookup"><span data-stu-id="ef0c8-126">Update getCalendarViewStartingAt</span></span>

1. <span data-ttu-id="ef0c8-127">Abra **GraphManager.h**.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-127">Open **GraphManager.h**.</span></span> <span data-ttu-id="ef0c8-128">Cambie la `GetCalendarViewCompletionBlock` definición de tipo a lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-128">Change the `GetCalendarViewCompletionBlock` type definition to the following.</span></span>

    ```objc
    typedef void (^GetCalendarViewCompletionBlock)(NSArray<MSGraphEvent*>* _Nullable events, NSError* _Nullable error);
    ```

1. <span data-ttu-id="ef0c8-129">Abra **GraphManager.m**.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-129">Open **GraphManager.m**.</span></span> <span data-ttu-id="ef0c8-130">Reemplace la función `getCalendarViewStartingAt` existente por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-130">Replace the existing `getCalendarViewStartingAt` function with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphManager.m" id="GetCalendarViewSnippet" highlight="42-61":::

### <a name="update-calendarviewcontroller"></a><span data-ttu-id="ef0c8-131">Actualizar CalendarViewController</span><span class="sxs-lookup"><span data-stu-id="ef0c8-131">Update CalendarViewController</span></span>

1. <span data-ttu-id="ef0c8-132">Cree un nuevo **archivo de clase Cocoa Touch** en el proyecto **GraphTutorial** denominado `CalendarTableViewCell` .</span><span class="sxs-lookup"><span data-stu-id="ef0c8-132">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewCell`.</span></span> <span data-ttu-id="ef0c8-133">Elija **UITableViewCell** en la **subclase del** campo.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-133">Choose **UITableViewCell** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="ef0c8-134">Abre **CalendarTableViewCell.h** y reemplaza su contenido por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-134">Open **CalendarTableViewCell.h** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.h" id="CalendarTableCellSnippet":::

1. <span data-ttu-id="ef0c8-135">Abre **CalendarTableViewCell.m y** reemplaza su contenido por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-135">Open **CalendarTableViewCell.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.m" id="CalendarTableCellSnippet":::

1. <span data-ttu-id="ef0c8-136">Cree un nuevo **archivo de clase Cocoa Touch** en el proyecto **GraphTutorial** denominado `CalendarTableViewController` .</span><span class="sxs-lookup"><span data-stu-id="ef0c8-136">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewController`.</span></span> <span data-ttu-id="ef0c8-137">Elija **UITableViewController** en la **subclase del** campo.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-137">Choose **UITableViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="ef0c8-138">Abra **CalendarTableViewController.h** y reemplace su contenido por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-138">Open **CalendarTableViewController.h** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.h" id="CalendarTableViewControllerSnippet":::

1. <span data-ttu-id="ef0c8-139">Abra **CalendarTableViewController.m y** reemplace su contenido por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-139">Open **CalendarTableViewController.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.m" id="CalendarTableViewControllerSnippet":::

1. <span data-ttu-id="ef0c8-140">Abra **Main.storyboard** y busque la **escena del calendario.**</span><span class="sxs-lookup"><span data-stu-id="ef0c8-140">Open **Main.storyboard** and locate the **Calendar Scene**.</span></span> <span data-ttu-id="ef0c8-141">Elimine la vista de desplazamiento de la vista raíz.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-141">Delete the scroll view from the root view.</span></span>
1. <span data-ttu-id="ef0c8-142">Mediante la **biblioteca,** agregue una **barra de navegación** en la parte superior de la vista.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-142">Using the **Library**, add a **Navigation Bar** to the top of the view.</span></span>
1. <span data-ttu-id="ef0c8-143">Haga doble clic en **el título** de la barra de navegación y actualíquelo a `Calendar` .</span><span class="sxs-lookup"><span data-stu-id="ef0c8-143">Double-click the **Title** in the navigation bar and update it to `Calendar`.</span></span>
1. <span data-ttu-id="ef0c8-144">Mediante la **biblioteca,** agregue un elemento **de botón** de barra a la derecha de la barra de navegación.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-144">Using the **Library**, add a **Bar Button Item** to the right-hand side of the navigation bar.</span></span>
1. <span data-ttu-id="ef0c8-145">Seleccione el botón nueva barra y, a continuación, seleccione el **Inspector de atributos.**</span><span class="sxs-lookup"><span data-stu-id="ef0c8-145">Select the new bar button, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="ef0c8-146">Cambiar **imagen** a **más**.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-146">Change **Image** to **plus**.</span></span>
1. <span data-ttu-id="ef0c8-147">Agregue una **vista de contenedor** de la biblioteca **a** la vista debajo de la barra de navegación.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-147">Add a **Container View** from the **Library** to the view under the navigation bar.</span></span> <span data-ttu-id="ef0c8-148">Cambie el tamaño de la vista de contenedor para que tome todo el espacio restante en la vista.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-148">Resize the container view to take all of the remaining space in the view.</span></span>
1. <span data-ttu-id="ef0c8-149">Establezca las restricciones en la barra de navegación y la vista de contenedor de la siguiente manera.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-149">Set constraints on the navigation bar and container view as follows.</span></span>

    - <span data-ttu-id="ef0c8-150">**Barra de navegación**</span><span class="sxs-lookup"><span data-stu-id="ef0c8-150">**Navigation Bar**</span></span>
        - <span data-ttu-id="ef0c8-151">Agregar restricción: Alto, valor: 44</span><span class="sxs-lookup"><span data-stu-id="ef0c8-151">Add constraint: Height, value: 44</span></span>
        - <span data-ttu-id="ef0c8-152">Agregar restricción: Espacio inicial a Área segura, valor: 0</span><span class="sxs-lookup"><span data-stu-id="ef0c8-152">Add constraint: Leading space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="ef0c8-153">Agregar restricción: Espacio final a Área segura, valor: 0</span><span class="sxs-lookup"><span data-stu-id="ef0c8-153">Add constraint: Trailing space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="ef0c8-154">Agregar restricción: Espacio superior a Área segura, valor: 0</span><span class="sxs-lookup"><span data-stu-id="ef0c8-154">Add constraint: Top space to Safe Area, value: 0</span></span>
    - <span data-ttu-id="ef0c8-155">**Vista de contenedor**</span><span class="sxs-lookup"><span data-stu-id="ef0c8-155">**Container View**</span></span>
        - <span data-ttu-id="ef0c8-156">Agregar restricción: Espacio inicial a Área segura, valor: 0</span><span class="sxs-lookup"><span data-stu-id="ef0c8-156">Add constraint: Leading space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="ef0c8-157">Agregar restricción: Espacio final a Área segura, valor: 0</span><span class="sxs-lookup"><span data-stu-id="ef0c8-157">Add constraint: Trailing space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="ef0c8-158">Agregar restricción: Espacio superior en la parte inferior de la barra de navegación, valor: 0</span><span class="sxs-lookup"><span data-stu-id="ef0c8-158">Add constraint: Top space to Navigation Bar Bottom, value: 0</span></span>
        - <span data-ttu-id="ef0c8-159">Agregar restricción: espacio inferior a Área segura, valor: 0</span><span class="sxs-lookup"><span data-stu-id="ef0c8-159">Add constraint: Bottom space to Safe Area, value: 0</span></span>

1. <span data-ttu-id="ef0c8-160">Busca el segundo controlador de vista agregado al guión gráfico cuando agregaste la vista de contenedor.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-160">Locate the second view controller added to the storyboard when you added the container view.</span></span> <span data-ttu-id="ef0c8-161">Se conecta a la escena **de calendario** mediante un segue de inserción.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-161">It is connected to the **Calendar Scene** by an embed segue.</span></span> <span data-ttu-id="ef0c8-162">Seleccione este controlador y use el **Inspector de identidad** para cambiar la **clase** **a CalendarTableViewController**.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-162">Select this controller and use the **Identity Inspector** to change **Class** to **CalendarTableViewController**.</span></span>
1. <span data-ttu-id="ef0c8-163">Elimine la **vista del** controlador de vista de tabla **de calendario.**</span><span class="sxs-lookup"><span data-stu-id="ef0c8-163">Delete the **View** from the **Calendar Table View Controller**.</span></span>
1. <span data-ttu-id="ef0c8-164">Agregue una **vista de tabla** de la biblioteca **al** controlador de vista de tabla **de calendario.**</span><span class="sxs-lookup"><span data-stu-id="ef0c8-164">Add a **Table View** from the **Library** to the **Calendar Table View Controller**.</span></span>
1. <span data-ttu-id="ef0c8-165">Seleccione la vista de tabla y, a continuación, seleccione el **Inspector de atributos.**</span><span class="sxs-lookup"><span data-stu-id="ef0c8-165">Select the table view, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="ef0c8-166">Establecer **celdas prototipo** en **1**.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-166">Set **Prototype Cells** to **1**.</span></span>
1. <span data-ttu-id="ef0c8-167">Arrastre el borde inferior de la celda prototipo para darle un área más grande con la que trabajar.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-167">Drag the bottom edge of the prototype cell to give you a larger area to work with.</span></span>
1. <span data-ttu-id="ef0c8-168">Use la **biblioteca para** agregar tres **etiquetas a** la celda prototipo.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-168">Use the **Library** to add three **Labels** to the prototype cell.</span></span>
1. <span data-ttu-id="ef0c8-169">Seleccione la celda prototipo y, a continuación, seleccione **el Inspector de identidad.**</span><span class="sxs-lookup"><span data-stu-id="ef0c8-169">Select the prototype cell, then select the **Identity Inspector**.</span></span> <span data-ttu-id="ef0c8-170">Cambiar **clase** a **CalendarTableViewCell**.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-170">Change **Class** to **CalendarTableViewCell**.</span></span>
1. <span data-ttu-id="ef0c8-171">Seleccione el **Inspector de atributos** y establezca el **identificador** en `EventCell` .</span><span class="sxs-lookup"><span data-stu-id="ef0c8-171">Select the **Attributes Inspector** and set **Identifier** to `EventCell`.</span></span>
1. <span data-ttu-id="ef0c8-172">Con **eventCell** seleccionado, seleccione el **Inspector de** conexiones y conéctese, y a las etiquetas que agregó a la celda en `durationLabel` el `organizerLabel` `subjectLabel` guión gráfico.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-172">With the **EventCell** selected, select the **Connections Inspector** and connect `durationLabel`, `organizerLabel`, and `subjectLabel` to the labels you added to the cell on the storyboard.</span></span>
1. <span data-ttu-id="ef0c8-173">Establezca las propiedades y restricciones de las tres etiquetas de la siguiente manera.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-173">Set the properties and constraints on the three labels as follows.</span></span>

    - <span data-ttu-id="ef0c8-174">**Etiqueta de asunto**</span><span class="sxs-lookup"><span data-stu-id="ef0c8-174">**Subject Label**</span></span>
        - <span data-ttu-id="ef0c8-175">Agregar restricción: espacio inicial al margen inicial de la vista de contenido, valor: 0</span><span class="sxs-lookup"><span data-stu-id="ef0c8-175">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="ef0c8-176">Agregar restricción: espacio final al margen final de la vista de contenido, valor: 0</span><span class="sxs-lookup"><span data-stu-id="ef0c8-176">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="ef0c8-177">Agregar restricción: Espacio superior al margen superior de la vista de contenido, valor: 0</span><span class="sxs-lookup"><span data-stu-id="ef0c8-177">Add constraint: Top space to Content View Top Margin, value: 0</span></span>
    - <span data-ttu-id="ef0c8-178">**Etiqueta del organizador**</span><span class="sxs-lookup"><span data-stu-id="ef0c8-178">**Organizer Label**</span></span>
        - <span data-ttu-id="ef0c8-179">Fuente: System 12.0</span><span class="sxs-lookup"><span data-stu-id="ef0c8-179">Font: System 12.0</span></span>
        - <span data-ttu-id="ef0c8-180">Agregar restricción: Alto, valor: 15</span><span class="sxs-lookup"><span data-stu-id="ef0c8-180">Add constraint: Height, value: 15</span></span>
        - <span data-ttu-id="ef0c8-181">Agregar restricción: espacio inicial al margen inicial de la vista de contenido, valor: 0</span><span class="sxs-lookup"><span data-stu-id="ef0c8-181">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="ef0c8-182">Agregar restricción: espacio final al margen final de la vista de contenido, valor: 0</span><span class="sxs-lookup"><span data-stu-id="ef0c8-182">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="ef0c8-183">Agregar restricción: Espacio superior a La parte inferior de la etiqueta de asunto, valor: Estándar</span><span class="sxs-lookup"><span data-stu-id="ef0c8-183">Add constraint: Top space to Subject Label Bottom, value: Standard</span></span>
    - <span data-ttu-id="ef0c8-184">**Etiqueta de duración**</span><span class="sxs-lookup"><span data-stu-id="ef0c8-184">**Duration Label**</span></span>
        - <span data-ttu-id="ef0c8-185">Fuente: System 12.0</span><span class="sxs-lookup"><span data-stu-id="ef0c8-185">Font: System 12.0</span></span>
        - <span data-ttu-id="ef0c8-186">Color: Color gris oscuro</span><span class="sxs-lookup"><span data-stu-id="ef0c8-186">Color: Dark Gray Color</span></span>
        - <span data-ttu-id="ef0c8-187">Agregar restricción: Alto, valor: 15</span><span class="sxs-lookup"><span data-stu-id="ef0c8-187">Add constraint: Height, value: 15</span></span>
        - <span data-ttu-id="ef0c8-188">Agregar restricción: espacio inicial al margen inicial de la vista de contenido, valor: 0</span><span class="sxs-lookup"><span data-stu-id="ef0c8-188">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="ef0c8-189">Agregar restricción: espacio final al margen final de la vista de contenido, valor: 0</span><span class="sxs-lookup"><span data-stu-id="ef0c8-189">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="ef0c8-190">Agregar restricción: espacio superior en la parte inferior de la etiqueta del organizador, valor: Estándar</span><span class="sxs-lookup"><span data-stu-id="ef0c8-190">Add constraint: Top space to Organizer Label Bottom, value: Standard</span></span>
        - <span data-ttu-id="ef0c8-191">Agregar restricción: espacio inferior al margen inferior de la vista de contenido, valor: 0</span><span class="sxs-lookup"><span data-stu-id="ef0c8-191">Add constraint: Bottom space to Content View Bottom Margin, value: 0</span></span>

1. <span data-ttu-id="ef0c8-192">Seleccione **eventCell** y, a continuación, seleccione **el Inspector de tamaño**.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-192">Select the **EventCell**, then select the **Size Inspector**.</span></span> <span data-ttu-id="ef0c8-193">Habilitar **Automático para** alto de **fila.**</span><span class="sxs-lookup"><span data-stu-id="ef0c8-193">Enable **Automatic** for **Row Height**.</span></span>

    ![Captura de pantalla de los controladores de vista de tabla de calendario y calendario](images/calendar-table-storyboard.png)

1. <span data-ttu-id="ef0c8-195">Abra **CalendarViewController.h** y quite la `calendarJSON` propiedad.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-195">Open **CalendarViewController.h** and remove the `calendarJSON` property.</span></span>

1. <span data-ttu-id="ef0c8-196">Abra **CalendarViewController.m y** reemplace su contenido por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-196">Open **CalendarViewController.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarViewController.m" id="CalendarViewControllerSnippet":::

1. <span data-ttu-id="ef0c8-197">Ejecuta la aplicación, inicia sesión y pulsa en la **pestaña** Calendario. Debería ver la lista de eventos.</span><span class="sxs-lookup"><span data-stu-id="ef0c8-197">Run the app, sign in, and tap the **Calendar** tab. You should see the list of events.</span></span>

    ![Captura de pantalla de la tabla de eventos](./images/calendar-list.png)
