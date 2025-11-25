### 🛡️ Descripción de `ValidationFilter`

`ValidationFilter` es un filtro de acción para ASP.NET Core que realiza validaciones automáticas a los modelos recibidos en los controladores antes de ejecutar la acción correspondiente.  

Su propósito es evitar repetir lógica de validación dentro de los controladores y devolver respuestas consistentes cuando los datos enviados por el cliente no cumplen las reglas definidas con FluentValidation.

Este filtro:

- Detecta el parámetro del método que corresponde a un modelo que deba ser validado.
- Obtiene el validador correspondiente desde la inyección de dependencias.
- Ejecuta la validación usando FluentValidation.
- Si la validación falla:
  - Detiene la ejecución de la acción.
  - Devuelve un `400 Bad Request` con una lista de errores indicando:
    - El campo afectado.
    - El mensaje de error.

Si no hay errores, el filtro permite que la acción continúe normalmente.

Este enfoque centraliza la validación, reduce código repetido en los controladores y garantiza respuestas coherentes para el cliente.


``` C#
public class ValidationFilter : IAsyncActionFilter
{
    public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
    {
        var param = context.ActionArguments.SingleOrDefault(p =>
            p.Value is CreateCategoriaDTO || p.Value is UpdateCategoriaDTO
        );

        if (param.Value == null)
        {
            await next();
            return;
        }

        var validator = (IValidator) context.HttpContext.RequestServices.GetService(typeof(IValidator<>).MakeGenericType(param.Value.GetType()));
        if (validator == null)
        {
            await next();
            return;
        }

        var validationContext = new ValidationContext<object>(param.Value);
        var validationResult = await validator.ValidateAsync(validationContext);

        if (!validationResult.IsValid)
        {
            var errors = validationResult.Errors
                .Select(error => new
                {
                    Field = error.PropertyName,
                    Message = error.ErrorMessage
                });

            context.Result = new BadRequestObjectResult(new { statusCode = 400, validationErrorsMessage = errors });
            return;
        }

        await next();
    }
}