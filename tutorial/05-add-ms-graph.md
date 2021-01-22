<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio incorporará Microsoft Graph en la aplicación. Para esta aplicación, usará el SDK de Microsoft Graph para [el Objetivo C](https://github.com/microsoftgraph/msgraph-sdk-objc) para realizar llamadas a Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Obtener eventos de calendario de Outlook

En esta sección, ampliará la clase para agregar una función para obtener los eventos del usuario de la semana actual y actualizar para usar `GraphManager` `CalendarViewController` esta nueva función.

1. Abra **GraphManager.h y** agregue el siguiente código encima de la `@interface` declaración.

    ```objc
    typedef void (^GetCalendarViewCompletionBlock)(NSData* _Nullable data,
                                                   NSError* _Nullable error);
    ```

1. Agregue el siguiente código a la `@interface` declaración.

    ```objc
    - (void) getCalendarViewStartingAt: (NSString*) viewStart
                              endingAt: (NSString*) viewEnd
                   withCompletionBlock: (GetCalendarViewCompletionBlock) completion;
    ```

1. Abra **GraphManager.m** y agregue la siguiente función a la `GraphManager` clase.

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
    > Ten en cuenta lo que hace `getCalendarViewStartingAt` el código.
    >
    > - La dirección URL a la que se llamará es `/v1.0/me/calendarview` .
    >   - Los `startDateTime` parámetros de consulta y de consulta definen el inicio y el final de la vista de `endDateTime` calendario.
    >   - El `select` parámetro de consulta limita los campos devueltos para cada evento a solo aquellos que la vista usará realmente.
    >   - El `orderby` parámetro de consulta ordena los resultados por hora de inicio.
    >   - El `top` parámetro de consulta solicita 25 resultados por página.
    >   - el encabezado hace que Microsoft Graph devuelva las horas de inicio y finalización de cada evento en la `Prefer: outlook.timezone` zona horaria del usuario.

1. Cree una nueva **clase Cocoa Touch** en el proyecto **GraphTutorial** denominado **GraphToIana**. Elija **NSObject** en la **subclase del** campo.
1. Abra **GraphToIana.h** y reemplace su contenido por el siguiente código.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.h" id="GraphToIanaSnippet":::

1. Abra **GraphToIana.m y** reemplace su contenido por el código siguiente.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.m" id="GraphToIanaSnippet":::

    Esto hace una búsqueda simple para buscar un identificador de zona horaria IANA basado en el nombre de zona horaria devuelto por Microsoft Graph.

1. Abra **CalendarViewController.m y** reemplace todo su contenido por el siguiente código.

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

1. Ejecuta la aplicación, inicia sesión y pulsa el **elemento de** navegación Calendario en el menú. Debería ver un volcado JSON de los eventos en la aplicación.

## <a name="display-the-results"></a>Mostrar los resultados

Ahora puede reemplazar el volcado json por algo para mostrar los resultados de forma fácil de usar. En esta sección, modificará la función para devolver objetos fuertemente especificados y modificará para usar una vista de tabla `getCalendarViewStartingAt` `CalendarViewController` para representar los eventos.

### <a name="update-getcalendarviewstartingat"></a>Actualizar getCalendarViewStartingAt

1. Abra **GraphManager.h**. Cambie la `GetCalendarViewCompletionBlock` definición de tipo a lo siguiente.

    ```objc
    typedef void (^GetCalendarViewCompletionBlock)(NSArray<MSGraphEvent*>* _Nullable events, NSError* _Nullable error);
    ```

1. Abra **GraphManager.m**. Reemplace la función `getCalendarViewStartingAt` existente por el siguiente código.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphManager.m" id="GetCalendarViewSnippet" highlight="42-61":::

### <a name="update-calendarviewcontroller"></a>Actualizar CalendarViewController

1. Cree un nuevo **archivo de clase Cocoa Touch** en el proyecto **GraphTutorial** denominado `CalendarTableViewCell` . Elija **UITableViewCell** en la **subclase del** campo.
1. Abre **CalendarTableViewCell.h** y reemplaza su contenido por el siguiente código.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.h" id="CalendarTableCellSnippet":::

1. Abre **CalendarTableViewCell.m y** reemplaza su contenido por el siguiente código.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.m" id="CalendarTableCellSnippet":::

1. Cree un nuevo **archivo de clase Cocoa Touch** en el proyecto **GraphTutorial** denominado `CalendarTableViewController` . Elija **UITableViewController** en la **subclase del** campo.
1. Abra **CalendarTableViewController.h** y reemplace su contenido por el siguiente código.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.h" id="CalendarTableViewControllerSnippet":::

1. Abra **CalendarTableViewController.m y** reemplace su contenido por el siguiente código.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.m" id="CalendarTableViewControllerSnippet":::

1. Abra **Main.storyboard** y busque la **escena del calendario.** Elimine la vista de desplazamiento de la vista raíz.
1. Mediante la **biblioteca,** agregue una **barra de navegación** en la parte superior de la vista.
1. Haga doble clic en **el título** de la barra de navegación y actualíquelo a `Calendar` .
1. Mediante la **biblioteca,** agregue un elemento **de botón** de barra a la derecha de la barra de navegación.
1. Seleccione el botón nueva barra y, a continuación, seleccione el **Inspector de atributos.** Cambiar **imagen** a **más**.
1. Agregue una **vista de contenedor** de la biblioteca **a** la vista debajo de la barra de navegación. Cambie el tamaño de la vista de contenedor para que tome todo el espacio restante en la vista.
1. Establezca las restricciones en la barra de navegación y la vista de contenedor de la siguiente manera.

    - **Barra de navegación**
        - Agregar restricción: Alto, valor: 44
        - Agregar restricción: Espacio inicial a Área segura, valor: 0
        - Agregar restricción: Espacio final a Área segura, valor: 0
        - Agregar restricción: Espacio superior a Área segura, valor: 0
    - **Vista de contenedor**
        - Agregar restricción: Espacio inicial a Área segura, valor: 0
        - Agregar restricción: Espacio final a Área segura, valor: 0
        - Agregar restricción: Espacio superior en la parte inferior de la barra de navegación, valor: 0
        - Agregar restricción: espacio inferior a Área segura, valor: 0

1. Busca el segundo controlador de vista agregado al guión gráfico cuando agregaste la vista de contenedor. Se conecta a la escena **de calendario** mediante un segue de inserción. Seleccione este controlador y use el **Inspector de identidad** para cambiar la **clase** **a CalendarTableViewController**.
1. Elimine la **vista del** controlador de vista de tabla **de calendario.**
1. Agregue una **vista de tabla** de la biblioteca **al** controlador de vista de tabla **de calendario.**
1. Seleccione la vista de tabla y, a continuación, seleccione el **Inspector de atributos.** Establecer **celdas prototipo** en **1**.
1. Arrastre el borde inferior de la celda prototipo para darle un área más grande con la que trabajar.
1. Use la **biblioteca para** agregar tres **etiquetas a** la celda prototipo.
1. Seleccione la celda prototipo y, a continuación, seleccione **el Inspector de identidad.** Cambiar **clase** a **CalendarTableViewCell**.
1. Seleccione el **Inspector de atributos** y establezca el **identificador** en `EventCell` .
1. Con **eventCell** seleccionado, seleccione el **Inspector de** conexiones y conéctese, y a las etiquetas que agregó a la celda en `durationLabel` el `organizerLabel` `subjectLabel` guión gráfico.
1. Establezca las propiedades y restricciones de las tres etiquetas de la siguiente manera.

    - **Etiqueta de asunto**
        - Agregar restricción: espacio inicial al margen inicial de la vista de contenido, valor: 0
        - Agregar restricción: espacio final al margen final de la vista de contenido, valor: 0
        - Agregar restricción: Espacio superior al margen superior de la vista de contenido, valor: 0
    - **Etiqueta del organizador**
        - Fuente: System 12.0
        - Agregar restricción: Alto, valor: 15
        - Agregar restricción: espacio inicial al margen inicial de la vista de contenido, valor: 0
        - Agregar restricción: espacio final al margen final de la vista de contenido, valor: 0
        - Agregar restricción: Espacio superior a La parte inferior de la etiqueta de asunto, valor: Estándar
    - **Etiqueta de duración**
        - Fuente: System 12.0
        - Color: Color gris oscuro
        - Agregar restricción: Alto, valor: 15
        - Agregar restricción: espacio inicial al margen inicial de la vista de contenido, valor: 0
        - Agregar restricción: espacio final al margen final de la vista de contenido, valor: 0
        - Agregar restricción: espacio superior en la parte inferior de la etiqueta del organizador, valor: Estándar
        - Agregar restricción: espacio inferior al margen inferior de la vista de contenido, valor: 0

1. Seleccione **eventCell** y, a continuación, seleccione **el Inspector de tamaño**. Habilitar **Automático para** alto de **fila.**

    ![Captura de pantalla de los controladores de vista de tabla de calendario y calendario](images/calendar-table-storyboard.png)

1. Abra **CalendarViewController.h** y quite la `calendarJSON` propiedad.

1. Abra **CalendarViewController.m y** reemplace su contenido por el siguiente código.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarViewController.m" id="CalendarViewControllerSnippet":::

1. Ejecuta la aplicación, inicia sesión y pulsa en la **pestaña** Calendario. Debería ver la lista de eventos.

    ![Captura de pantalla de la tabla de eventos](./images/calendar-list.png)
