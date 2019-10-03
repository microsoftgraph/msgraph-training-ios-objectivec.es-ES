<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, incorporará Microsoft Graph a la aplicación. Para esta aplicación, usará el [SDK de Microsoft Graph para Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) para realizar llamadas a Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Obtener eventos de calendario de Outlook

En esta sección, ampliará la `GraphManager` clase para agregar una función para obtener los eventos del usuario y actualizar `CalendarViewController` para usar estas funciones nuevas.

1. Abra **GraphManager. h** y agregue el siguiente código sobre la `@interface` declaración.

    ```objc
    typedef void (^GetEventsCompletionBlock)(NSData* _Nullable data, NSError* _Nullable error);
    ```

1. Agregue el siguiente código a la `@interface` declaración.

    ```objc
    - (void) getEventsWithCompletionBlock: (GetEventsCompletionBlock)completionBlock;
    ```

1. Abra **GraphManager. m** y agregue la siguiente función a la `GraphManager` clase.

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
    > Tenga en cuenta lo que `getEventsWithCompletionBlock` hace el código.
    >
    > - La dirección URL a la que se `/v1.0/me/events`llamará es.
    > - El `select` parámetro de consulta limita los campos devueltos para cada evento a solo aquellos que la aplicación usará realmente.
    > - El `orderby` parámetro de consulta ordena los resultados por la fecha y hora en que se crearon, con el elemento más reciente en primer lugar.

1. Abra **CalendarViewController. m** y reemplace todo su contenido por el código siguiente.

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

Ahora puede ejecutar la aplicación, iniciar sesión y puntear en el elemento de navegación **calendario** del menú. Debería ver un volcado JSON de los eventos en la aplicación.

## <a name="display-the-results"></a>Mostrar los resultados

Ahora puede reemplazar el volcado de JSON con algo para mostrar los resultados de forma fácil de uso. En esta sección, modificará la `getEventsWithCompletionBlock` función para que devuelva objetos con establecimiento inflexible de tipos `CalendarViewController` y lo modifique para usar una vista de tabla para representar los eventos.

### <a name="update-getevents"></a>Actualizar getEvents

1. Abra **GraphManager. h**. Cambie la `GetEventsCompletionBlock` definición de tipo por la siguiente.

    ```objc
    typedef void (^GetEventsCompletionBlock)(NSArray<MSGraphEvent*>* _Nullable events, NSError* _Nullable error);
    ```

1. Abra **GraphManager. m**. Reemplace la `completionBlock(data, nil);` línea en la `getEventsWithCompletionBlock` función por el siguiente código.

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

### <a name="update-calendarviewcontroller"></a>Actualizar CalendarViewController

1. Cree un nuevo archivo de **clase táctil de cacao** en el proyecto `CalendarTableViewCell` **GraphTutorial** denominado. Elija **UITableViewCell** en el campo **subclase de** .
1. Abra **CalendarTableViewCell. h** y reemplace su contenido por el código siguiente.

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

1. Abra **CalendarTableViewCell. m** y reemplace su contenido por el código siguiente.

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

1. Abra **Main. Storyboard** y busque la **escena del calendario**. Seleccione la **vista** de la **escena del calendario** y elimínela.

    ![Captura de pantalla de la vista en la escena del calendario](./images/view-in-calendar-scene.png)

1. Agregue una **vista de tabla** de la **biblioteca** a la **escena del calendario**.
1. Seleccione la vista de tabla y, a continuación, seleccione el **Inspector de atributos**. Establezca **las celdas del prototipo** en **1**.
1. Use la **biblioteca** para agregar tres **etiquetas** a la celda del prototipo.
1. Seleccione la celda prototype y, a continuación, seleccione el **Inspector de identidad**. Cambie la **clase** a **CalendarTableViewCell**.
1. Seleccione el **Inspector de atributos** y establezca **identificador** en `EventCell`.
1. En el menú **Editor** , seleccione **resolver problemas de diseño automático**y, a continuación, seleccione **Agregar restricciones que faltan** debajo **de todas las vistas en el controlador de vista de bienvenida**.
1. Con la **EventCell** seleccionada, seleccione el **Inspector de conexiones** y `durationLabel`Conéctese `organizerLabel`, `subjectLabel` y a las etiquetas que haya agregado a la celda en el guión gráfico.

    ![Captura de pantalla del diseño de celda de prototipo](./images/prototype-cell-layout.png)

1. Abra **CalendarViewController. h** y quite la `calendarJSON` propiedad.
1. Cambie la `@interface` declaración a la siguiente.

    ```objc
    @interface CalendarViewController : UITableViewController
    ```

1. Abra **CalendarViewController. m** y reemplace su contenido por el código siguiente.

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

1. Ejecute la aplicación, inicie sesión y pulse la pestaña **calendario** . Debe ver la lista de eventos.

    ![Captura de pantalla de la tabla de eventos](./images/calendar-list.png)
