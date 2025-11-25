### 🔌 Descripción de `MiddlewareExtensions`

`MiddlewareExtensions` es una clase de extensión que simplifica la forma de registrar el middleware de manejo de excepciones (`ExceptionHandlingMiddleware`) dentro del pipeline de ASP.NET Core.  

Proporciona el método `UseExceptionHandling()`, que permite agregar el middleware a la aplicación de manera limpia y expresiva desde `Program.cs` o `Startup.cs`, evitando tener que usar `UseMiddleware<ExceptionHandlingMiddleware>()` directamente.

Este enfoque mejora la legibilidad del código y sigue la convención estándar de ASP.NET Core para registrar middlewares mediante métodos de extensión.


```
  public static class MiddlewareExtensions
  {
      public static IApplicationBuilder UseExceptionHandling(this IApplicationBuilder builder)
      {
          return builder.UseMiddleware<ExceptionHandlingMiddleware>();
      }
  }