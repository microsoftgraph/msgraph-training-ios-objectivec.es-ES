# <a name="contributing-to-microsoft-graph-training-repositories"></a>Contribución a repositorios de aprendizaje de Microsoft Graph

Gracias por contribuir a este proyecto. Antes de enviar la solicitud de extracción, asegúrese de tener en cuenta lo siguiente.

## <a name="overview"></a>Información general

El código de este repositorio sirve para tres propósitos:

- Los archivos markdown de la [carpeta del tutorial](/tutorial) se publican como un tutorial en la página de [tutoriales de Microsoft Graph.](https://docs.microsoft.com/graph/tutorials)
- El proyecto de ejemplo de la [carpeta de demostración](/demo) es el origen de un inicio rápido de [Microsoft Graph](https://developer.microsoft.com/graph/quick-start).* *\** _
- El proyecto de ejemplo de la carpeta de demostración también se puede descargar directamente desde GitHub y debe ejecutarse tal como está después de una configuración sencilla.

> _*\**_ No todos los repositorios de aprendizaje están disponibles como inicios rápidos (todavía).

Esto es importante tener en cuenta, ya que los cambios en un lugar _may* requieren cambios en otro, para mantener las cosas sincronizadas. Siempre que sea posible, los archivos Markdown hacen referencia directamente a los archivos de código fuente (mediante una sintaxis personalizada), de modo que la actualización del código en el origen actualizará automáticamente el código `:::code` en Markdown.

## <a name="updating-code"></a>Actualizar código

La `:::code` sintaxis usada en Markdown depende de comentarios específicos en el archivo de código fuente. Estos comentarios tienen el siguiente aspecto:

```csharp
// <MySnippet>
Console.WriteLine("Hello World!");
// </MySnippet>
```

Si actualiza el código entre estos comentarios de "marcador", los archivos de Markdown recibirán automáticamente esos cambios cuando se publiquen en el sitio de documentación de Microsoft Graph. Si actualiza código fuera de esos comentarios, es muy probable que necesite actualizar el Markdown correspondiente.

## <a name="adding-features"></a>Agregar características

Aunque se agradece el entusiasmo, no envíes solicitudes de extracción para agregar nuevas características a la muestra. Dado que este repositorio es principalmente un tutorial de "compilación de la primera aplicación", el conjunto de características está limitado, por diseño.

## <a name="submitting-pull-requests"></a>Envío de solicitudes de extracción

Envíe todas las solicitudes de extracción a la `master` rama.

## <a name="when-do-changes-get-published"></a>¿Cuándo se publican los cambios?

La publicación de actualizaciones en el sitio [de tutoriales](https://docs.microsoft.com/graph/tutorials) de Microsoft Graph no es automática. Los cambios deben promoverse primero a la rama y, a continuación, los administradores del sitio deben desencadenar una `live` compilación. Esto suele hacerse "según sea necesario".

## <a name="code-of-conduct"></a>Código de conducta

Este proyecto ha adoptado el [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/) (Código de conducta de código abierto de Microsoft). Para obtener más información, consulte las [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) (Preguntas más frecuentes del código de conducta) o póngase en contacto con [opencode@microsoft.com](mailto:opencode@microsoft.com) con otras preguntas o comentarios.
