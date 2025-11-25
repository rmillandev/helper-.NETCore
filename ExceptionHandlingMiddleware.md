### 🧰 Descripción de `ExceptionHandlingMiddleware`

`ExceptionHandlingMiddleware` es un middleware para ASP.NET Core que centraliza el manejo de excepciones no controladas dentro de la aplicación. Su propósito es capturar cualquier error que ocurra durante el procesamiento de una petición HTTP, registrarlo y devolver una respuesta uniforme y clara al cliente.

Este middleware:

- Envuelve la ejecución de la siguiente etapa del pipeline usando un bloque `try/catch`.
- Si ocurre una excepción, la registra usando `ILogger`.
- Devuelve una respuesta en formato JSON con:
  - Código HTTP correspondiente.
  - Un mensaje de error entendible.
- Permite mapear excepciones específicas a códigos HTTP adecuados.

De esta forma, la aplicación logra un manejo de errores consistente, evita fugas de excepciones sin procesar y mejora la experiencia del cliente y la mantenibilidad del código.


``` C#
public class ExceptionHandlingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ExceptionHandlingMiddleware> _logger;

    public ExceptionHandlingMiddleware(RequestDelegate next, ILogger<ExceptionHandlingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext httpContext)
    {
        try
        {
            await _next(httpContext);
        } catch (Exception ex)
        {
            _logger.LogError(ex, "Unhandled exception occurred: {Message}", ex.Message);
            await HandleExceptionAsync(httpContext, ex);
        }
    }

    private static Task HandleExceptionAsync(HttpContext httpContext, Exception ex)
    {
        httpContext.Response.ContentType = "application/json";
        var statusCode = HttpStatusCode.InternalServerError;
        var message = "An unexpected error occurred.";

        switch (ex)
        {
            case KeyNotFoundException _:
                statusCode = HttpStatusCode.NotFound;
                message = ex.Message;
                break;
            case InvalidOperationException _:
                statusCode = HttpStatusCode.Conflict;
                message = ex.Message;
                break;


            default:
                break;
        }

        httpContext.Response.StatusCode = (int)statusCode;

        var result = JsonSerializer.Serialize(new
        {
            statusCode = httpContext.Response.StatusCode,
            errorMessage = message
        });

        return httpContext.Response.WriteAsync(result);
    }
}