
# Actividad: API ToDo Items

El objetivo de esta actividad es desarrollar una API con las siguientes características:

**![](https://lh3.googleusercontent.com/Vb7t_RUHJYjTDR4ROUSr6SQfCbFDunn4Q0mjovWCzdBOcAi3NzRLjEYoSY_qqXTDT4IBfaJhMuigPIe0hQNHcR9FQrrpqa_vrqhswu_mxYvGo4bz9TK4PjaoZdr60vzkAFhkRbaD=s0)**


## Clonar Repositorio (Clone/Checkout)
**1.Ejecutar comando clone para descargar repositorio:**

 ```bash
$ git clone https://github.com/utn-frc-iaew-2021/actividad_api_todo_items
 ```

**2. Ubicarse en la carpeta generada con el nombre del repositorio:**

 ```bash
$ cd actividad_login_acceso_basedatos
 ```
 
**3. Crear un nuevo branch (rama)**

Para crear una nueva rama (branch) y saltar a ella, en un solo paso, puedes utilizar el comando git checkout con la opción -b, indicando el nombre del nuevo branch (reemplazando el nro de legajo) de la siguiente forma branch_{legajo}, para el legajo 12345:

 ```bash
$ git checkout -b branch_12345 
Switched to a new branch "12345"
 ```

Y para que se vea reflejada en GitHub:
 ```bash
$ git push --set-upstream origin branch_12345
 ```

## 1. Crear proyecto

-   Abra el  [terminal integrado](https://code.visualstudio.com/docs/editor/integrated-terminal).
    
-   Cambie los directorios (`cd`) a la carpeta que va a contener la carpeta del proyecto.
    
-   Ejecute los comandos siguientes:
       
    ```bash
    dotnet new webapi -o TodoApi
    cd TodoApi
    dotnet add package Microsoft.EntityFrameworkCore.InMemory
    code -r ../TodoApi
    
    ```
    
-   Cuando en un cuadro de diálogo se le pregunte si quiere agregar al proyecto los recursos necesarios, seleccione  **Sí**.
    
  >  Los comandos anteriores:    
 >    -   Crean un nuevo proyecto de API web y lo abren en Visual Studio Code.
 >    -   Agregan los paquetes de NuGet que se requieren en la sección siguiente.

A continuación se puede ver la estructura del proyecto .net:

![El cliente está representado por un cuadro a la izquierda. Envía una solicitud y recibe una respuesta de la aplicación, representada por un cuadro en la parte derecha. En el cuadro de la aplicación, hay tres cuadros que representan el controlador, el modelo y la capa de acceso a datos. La solicitud entra en el controlador de la aplicación y se producen operaciones de lectura/escritura entre el controlador y la capa de acceso a datos. El modelo se serializa y se devuelve al cliente en la respuesta.](https://docs.microsoft.com/es-es/aspnet/core/tutorials/first-web-api/_static/architecture.png?view=aspnetcore-5.0)

## 2. Probar el proyecto

Copiar y pegar la  **URL de solicitud**  en el explorador:  `https://localhost:<port>/WeatherForecast`.

Se devuelve un JSON similar al siguiente:

 ``` json
[
    {
        "date": "2019-07-16T19:04:05.7257911-06:00",
        "temperatureC": 52,
        "temperatureF": 125,
        "summary": "Mild"
    },
    {
        "date": "2019-07-17T19:04:05.7258461-06:00",
        "temperatureC": 36,
        "temperatureF": 96,
        "summary": "Warm"
    },
    {
        "date": "2019-07-18T19:04:05.7258467-06:00",
        "temperatureC": 39,
        "temperatureF": 102,
        "summary": "Cool"
    },
    {
        "date": "2019-07-19T19:04:05.7258471-06:00",
        "temperatureC": 10,
        "temperatureF": 49,
        "summary": "Bracing"
    },
    {
        "date": "2019-07-20T19:04:05.7258474-06:00",
        "temperatureC": -1,
        "temperatureF": 31,
        "summary": "Chilly"
    }
]
 ```

## 3. Actualización de launchUrl

En _Properties\launchSettings.json_, actualice `launchUrl` de `"swagger"` a `"api/todoitems"`:


 ``` json
"launchUrl": "api/todoitems",
 ```
## 4. Incorporación de una clase de modelo

Un _modelo_ es un conjunto de clases que representan los datos que la aplicación administra. El modelo para esta aplicación es una clase `TodoItem` única.

-   Agregue una carpeta denominada  _Models_  .    
-   Agregue una clase  `TodoItem`  a la carpeta  _Models_  con el código siguiente:

 ``` csharp
namespace TodoApi.Models
{
    public class TodoItem
    {
        public long Id { get; set; }
        public string Name { get; set; }
        public bool IsComplete { get; set; }
    }
}
 ```
La propiedad  `Id`  funciona como clave única en una base de datos relacional.

Las clases de modelo pueden ir en cualquier lugar del proyecto, pero, por convención, se usa la carpeta  _Models_  .

## 5. Incorporación de un contexto de base de datos

El  _contexto de base de datos_  es la clase principal que coordina la funcionalidad de Entity Framework para un modelo de datos. Esta clase se crea derivándola de la clase  [Microsoft.EntityFrameworkCore.DbContext](https://docs.microsoft.com/es-es/dotnet/api/microsoft.entityframeworkcore.dbcontext).

-   Agregue una clase  `TodoContext`  a la carpeta  _Models_  .
 ``` csharp
using Microsoft.EntityFrameworkCore;

namespace TodoApi.Models
{
    public class TodoContext : DbContext
    {
        public TodoContext(DbContextOptions<TodoContext> options)
            : base(options)
        {
        }

        public DbSet<TodoItem> TodoItems { get; set; }
    }
}
 ```

## 6. Registro del contexto de base de datos

En ASP.NET Core, los servicios (como el contexto de la base de datos) deben registrarse con el contenedor de  [inserción de dependencias (DI)](https://docs.microsoft.com/es-es/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-5.0). El contenedor proporciona el servicio a los controladores.

Actualice  _Startup.cs_  con el siguiente código:
 ``` csharp
// Unused usings removed
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;


using Microsoft.EntityFrameworkCore; // Agregar
using TodoApi.Models; // Agregar

namespace TodoApi
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers();

            services.AddDbContext<TodoContext>(opt => // Agregar
                                               opt.UseInMemoryDatabase("TodoList"));// Agregar
            //services.AddSwaggerGen(c => 
            //{
            //    c.SwaggerDoc("v1", new OpenApiInfo { Title = "TodoApi", Version = "v1" });
            //});
        }

        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
                //app.UseSwagger();
                //app.UseSwaggerUI(c => c.SwaggerEndpoint("/swagger/v1/swagger.json", "TodoApi v1"));
            }

            app.UseHttpsRedirection();
            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
        }
    }
}
 ```

>El código anterior:
>
>-   Quita las llamadas de Swagger.
>-   Elimina las declaraciones  `using`  no utilizadas.
>-   Agrega el contexto de base de datos para el contenedor de DI.
>-   Especifica que el contexto de base de datos usará una base de datos en memoria.


## 7. Scaffolding de un controlador

Ejecute el comando siguiente desde la raíz del proyecto, `TodoApi/TodoApi`:

 ``` bash
 dotnet  add package Microsoft.VisualStudio.Web.CodeGeneration.Design 
 dotnet  add package Microsoft.EntityFrameworkCore.Design 
 dotnet  add package Microsoft.EntityFrameworkCore.SqlServer 
 dotnet  tool install -g dotnet-aspnet-codegenerator 
 dotnet aspnet-codegenerator controller -name TodoItemsController -async -api -m TodoItem -dc TodoContext -outDir Controllers
  ```
> Los comandos anteriores:
> 
> -   Agregan los paquetes NuGet necesarios para realizar scaffolding.
> -   Instalan el motor de scaffolding (`dotnet-aspnet-codegenerator`).
> -   Aplican scaffolding a  `TodoItemsController`.

El código generado:

-   Marca la clase con el atributo  [`[ApiController]`](https://docs.microsoft.com/es-es/dotnet/api/microsoft.aspnetcore.mvc.apicontrollerattribute). Este atributo indica que el controlador responde a las solicitudes de la API web. Para información sobre comportamientos específicos que permite el atributo, consulte  [Creación de API web con ASP.NET Core](https://docs.microsoft.com/es-es/aspnet/core/web-api/?view=aspnetcore-5.0).
-   Utiliza la inserción de dependencias para insertar el contexto de base de datos (`TodoContext`) en el controlador. El contexto de base de datos se usa en cada uno de los métodos  [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete)  del controlador.

Las plantillas de ASP.NET Core para:

-   Controladores con vistas incluyen  `[action]`  en la plantilla de ruta.
-   Controladores de API no incluyen  `[action]`  en la plantilla de ruta.

Si el token  `[action]`  no está en la plantilla de ruta, el nombre de la  [acción](https://docs.microsoft.com/es-es/aspnet/core/mvc/controllers/routing?view=aspnetcore-5.0#action)  se excluye de la ruta. Es decir, el nombre del método asociado a la acción no se usa en la ruta coincidente.

## 8. Actualización del método create de PostTodoItem

Actualice la instrucción "return" en  `PostTodoItem`  para usar el operador  [nameof](https://docs.microsoft.com/es-es/dotnet/csharp/language-reference/operators/nameof):

 ``` csharp
 // POST: api/TodoItems
[HttpPost]
public async Task<ActionResult<TodoItem>> PostTodoItem(TodoItem todoItem)
{
    _context.TodoItems.Add(todoItem);
    await _context.SaveChangesAsync();

    //return CreatedAtAction("GetTodoItem", new { id = todoItem.Id }, todoItem);
    return CreatedAtAction(nameof(GetTodoItem), new { id = todoItem.Id }, todoItem);
}
 ```

El código anterior es un método HTTP POST, indicado por el atributo  [`[HttpPost]`](https://docs.microsoft.com/es-es/dotnet/api/microsoft.aspnetcore.mvc.httppostattribute). El método obtiene el valor de tareas pendientes del cuerpo de la solicitud HTTP.

Para más información, vea  [Enrutamiento mediante atributos con atributos Http[Verb]](https://docs.microsoft.com/es-es/aspnet/core/mvc/controllers/routing?view=aspnetcore-5.0#attribute-routing-with-httpverb-attributes).

El método  [CreatedAtAction](https://docs.microsoft.com/es-es/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.createdataction)  realiza las acciones siguientes:

-   Devuelve un  [código de estado HTTP 201](https://developer.mozilla.org/docs/Web/HTTP/Status/201)  cuando se ha ejecutado correctamente. HTTP 201 es la respuesta estándar para un método HTTP POST que crea un recurso en el servidor.
-   Agrega un encabezado  [Location](https://developer.mozilla.org/docs/Web/HTTP/Headers/Location)  a la respuesta. El encabezado  `Location`  especifica el  [URI](https://developer.mozilla.org/docs/Glossary/URI)  de la tarea pendiente recién creada. Para obtener más información, consulte  [10.2.2 201 creado](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).
-   Hace referencia a la acción  `GetTodoItem`  para crear el identificador URI del encabezado  `Location`. La palabra clave  `nameof`  de C# se usa para evitar que se codifique de forma rígida el nombre de acción en la llamada a  `CreatedAtAction`.

### Prueba de PostTodoItem con Postman

-   Cree una nueva solicitud.
    
-   Establezca el método HTTP en  `POST`.
    
-   Establezca el URI en  `https://localhost:<port>/api/todoitems`. Por ejemplo:  `https://localhost:5001/api/todoitems`.
    
-   Seleccione la pestaña  **Cuerpo**.
    
-   Seleccione el botón de radio  **Raw**  (Sin formato).
    
-   Establezca el tipo en  **JSON (application/json)**  .
    
-   En el cuerpo de la solicitud, introduzca JSON para una tarea pendiente:
 ``` json
{
  "name":"walk dog",
  "isComplete":true
}
 ``` 
* Seleccione **Enviar**.
![Postman con solicitud de creación](https://docs.microsoft.com/es-es/aspnet/core/tutorials/first-web-api/_static/3/create.png?view=aspnetcore-5.0)

### Prueba del URI del encabezado de ubicación

El URI del encabezado de ubicación se puede probar en el explorador. Copie y pegue el URI del encabezado de ubicación en el explorador.

Para probarlo en Postman:

-   Seleccione la pestaña  **Encabezados**  en el panel  **Respuesta**.
    
-   Copie el valor de encabezado  **Ubicación**:
    
    ![Pestaña Encabezados de la consola de Postman](https://docs.microsoft.com/es-es/aspnet/core/tutorials/first-web-api/_static/3/create.png?view=aspnetcore-5.0)
    
-   Establezca el método HTTP en  `GET`.
    
-   Establezca el URI en  `https://localhost:<port>/api/todoitems/1`. Por ejemplo:  `https://localhost:5001/api/todoitems/1`.
    
-   Seleccione  **Enviar**.

## 9. Prueba de los métodos GET

Se implementan dos puntos de conexión GET:

-   `GET /api/todoitems`
-   `GET /api/todoitems/{id}`

Llame a los dos puntos de conexión desde un explorador o Postman para probar la aplicación. Por ejemplo:

-   `https://localhost:5001/api/todoitems`
-   `https://localhost:5001/api/todoitems/1`

La llamada a  `GetTodoItems`  genera una respuesta similar a la siguiente:
``` json
[
  {
    "id": 1,
    "name": "Item1",
    "isComplete": false
  }
]

```

### [](https://docs.microsoft.com/es-es/aspnet/core/tutorials/first-web-api?view=aspnetcore-5.0&tabs=visual-studio-code#test-get-with-postman)

### Prueba de Get con Postman

-   Cree una nueva solicitud.
-   Establezca el método HTTP en  **GET**.
-   Establezca el URI de solicitud en  `https://localhost:<port>/api/todoitems`. Por ejemplo:  `https://localhost:5001/api/todoitems`.
-   Establezca  **Vista de dos paneles**  en Postman.
-   Seleccione  **Enviar**.

Esta aplicación utiliza una base de datos en memoria. Si la aplicación se detiene y se inicia, la solicitud GET precedente no devolverá ningún dato. Si no se devuelve ningún dato, publique los datos en la aplicación con  [POST](https://docs.microsoft.com/es-es/aspnet/core/tutorials/first-web-api?view=aspnetcore-5.0&tabs=visual-studio-code#post).


## 10. Enrutamiento y rutas URL

El atributo  [`[HttpGet]`](https://docs.microsoft.com/es-es/dotnet/api/microsoft.aspnetcore.mvc.httpgetattribute)  indica un método que responde a una solicitud HTTP GET. La ruta de dirección URL para cada método se construye como sigue:

-   Comience por la cadena de plantilla en el atributo  `Route`  del controlador:
    
    
    ``` csharp	
    [Route("api/[controller]")]
    [ApiController]
    public class TodoItemsController : ControllerBase
    {
        private readonly TodoContext _context;
    
        public TodoItemsController(TodoContext context)
        {
            _context = context;
        }
    
    ```
    
-   Reemplace  `[controller]`  por el nombre del controlador, que convencionalmente es el nombre de clase de controlador sin el sufijo "Controller". En este ejemplo, el nombre de clase de controlador es  **TodoItems**  Controller; por tanto, el nombre del controlador es "TodoItems". El  [enrutamiento](https://docs.microsoft.com/es-es/aspnet/core/mvc/controllers/routing?view=aspnetcore-5.0)  en ASP.NET Core no distingue entre mayúsculas y minúsculas.
    
-   Si el atributo  `[HttpGet]`  tiene una plantilla de ruta (por ejemplo,  `[HttpGet("products")]`), anéxela a la ruta de acceso. En este ejemplo no se usa una plantilla. Para más información, vea  [Enrutamiento mediante atributos con atributos Http[Verb]](https://docs.microsoft.com/es-es/aspnet/core/mvc/controllers/routing?view=aspnetcore-5.0#attribute-routing-with-httpverb-attributes).
    

En el siguiente método  `GetTodoItem`,  `"{id}"`  es una variable de marcador de posición correspondiente al identificador único de la tarea pendiente. Al invocar a  `GetTodoItem`, el valor  `"{id}"`  de la URL se proporciona al método en su parámetro  `id`.


``` csharp
// GET: api/TodoItems/5
[HttpGet("{id}")]
public async Task<ActionResult<TodoItem>> GetTodoItem(long id)
{
    var todoItem = await _context.TodoItems.FindAsync(id);

    if (todoItem == null)
    {
        return NotFound();
    }

    return todoItem;
}

```

## 11. Valores devueltos

El tipo de valor devuelto de los métodos  `GetTodoItems`  y  `GetTodoItem`  es  [ActionResult<T> type](https://docs.microsoft.com/es-es/aspnet/core/web-api/action-return-types?view=aspnetcore-5.0#actionresultt-type). ASP.NET Core serializa automáticamente el objeto a  [JSON](https://www.json.org/)  y escribe el JSON en el cuerpo del mensaje de respuesta. El código de respuesta de este tipo de valor devuelto es  [200 OK](https://developer.mozilla.org/docs/Web/HTTP/Status/200), suponiendo que no haya ninguna excepción no controlada. Las excepciones no controladas se convierten en errores 5xx.

Los tipos de valores devueltos  `ActionResult`  pueden representar una gama amplia de códigos de estado HTTP. Por ejemplo,  `GetTodoItem`  puede devolver dos valores de estado diferentes:

-   Si no hay ningún elemento que coincida con el identificador solicitado, el método devolverá un código de error de  [estado 404](https://developer.mozilla.org/docs/Web/HTTP/Status/404)  [NotFound](https://docs.microsoft.com/es-es/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.notfound).
-   En caso contrario, el método devuelve 200 con un cuerpo de respuesta JSON. Devolver  `item`  genera una respuesta HTTP 200.

## 12. Método PutTodoItem

Examine el método  `PutTodoItem`:

C#Copiar

```
// PUT: api/TodoItems/5
[HttpPut("{id}")]
public async Task<IActionResult> PutTodoItem(long id, TodoItem todoItem)
{
    if (id != todoItem.Id)
    {
        return BadRequest();
    }

    _context.Entry(todoItem).State = EntityState.Modified;

    try
    {
        await _context.SaveChangesAsync();
    }
    catch (DbUpdateConcurrencyException)
    {
        if (!TodoItemExists(id))
        {
            return NotFound();
        }
        else
        {
            throw;
        }
    }

    return NoContent();
}

```

`PutTodoItem`  es similar a  `PostTodoItem`, salvo por el hecho de que usa HTTP PUT. La respuesta es  [204 Sin contenido](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html). Según la especificación HTTP, una solicitud PUT requiere que el cliente envíe toda la entidad actualizada, no solo los cambios. Para admitir actualizaciones parciales, use  [HTTP PATCH](https://docs.microsoft.com/es-es/dotnet/api/microsoft.aspnetcore.mvc.httppatchattribute).

Si recibe un error al llamar a  `PutTodoItem`, llame a  `GET`  para asegurarse de que hay un elemento en la base de datos.

### Prueba del método PutTodoItem

En este ejemplo se usa una base de datos en memoria que se debe inicializar cada vez que se inicia la aplicación. Debe haber un elemento en la base de datos antes de que realice una llamada PUT. Llame a GET para asegurarse de que hay un elemento en la base de datos antes de realizar una llamada PUT.

Actualice el elemento to-do que tiene el Id = 1 y establezca su nombre en  `"feed fish"`:

```json
  {
    "Id":1,
    "name":"feed fish",
    "isComplete":true
  }

```

En la imagen siguiente, se muestra la actualización de Postman:

![Consola de Postman que muestra la respuesta 204 (Sin contenido)](https://docs.microsoft.com/es-es/aspnet/core/tutorials/first-web-api/_static/3/pmcput.png?view=aspnetcore-5.0)

## 13. Método DeleteTodoItem

Examine el método  `DeleteTodoItem`:
``` csharp
// DELETE: api/TodoItems/5
[HttpDelete("{id}")]
public async Task<IActionResult> DeleteTodoItem(long id)
{
    var todoItem = await _context.TodoItems.FindAsync(id);
    if (todoItem == null)
    {
        return NotFound();
    }

    _context.TodoItems.Remove(todoItem);
    await _context.SaveChangesAsync();

    return NoContent();
}

```

### Prueba del método DeleteTodoItem

Use Postman para eliminar una tarea pendiente:

-   Establezca el método en  `DELETE`.
-   Establezca el URI del objeto que quiera eliminar (por ejemplo,  `https://localhost:5001/api/todoitems/1`).
-   Seleccione  **Enviar**.


Fuente: https://docs.microsoft.com/es-es/aspnet/core/tutorials/first-web-api?view=aspnetcore-5.0&tabs=visual-studio-code
