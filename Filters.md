# ASP.NET Core Exception Handling & Filters – Complete Guide

---

# 🧨 Production Issue: Unhandled Exceptions

## ❓ Problem

A production application is crashing due to unhandled exceptions.

## ✅ Solution Approach

Use **both middleware and filters**, but for different purposes.

---

# 🌍 Global Exception Handling in ASP.NET Core

## ✅ Middleware (Primary Solution)

Middleware is the **best place for global exception handling**.

### ✔ Why Middleware?

* Handles **entire request pipeline**
* Works for:

  * Controllers (MVC / Web API)
  * Minimal APIs
  * Non-MVC components
* Centralized logging & response handling
* Clean and scalable

---

## 🧩 Example: Global Exception Middleware

```csharp
public class GlobalExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<GlobalExceptionMiddleware> _logger;

    public GlobalExceptionMiddleware(RequestDelegate next, ILogger<GlobalExceptionMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task Invoke(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unhandled exception occurred");

            context.Response.StatusCode = 500;
            context.Response.ContentType = "application/json";

            var response = new
            {
                message = "An unexpected error occurred.",
                detail = ex.Message // Hide in production if needed
            };

            await context.Response.WriteAsJsonAsync(response);
        }
    }
}
```

---

## ⚙️ Register Middleware

```csharp
app.UseMiddleware<GlobalExceptionMiddleware>();
```

### Or built-in handler:

```csharp
app.UseExceptionHandler("/error");
```

---

## 🎯 Filters (Secondary, Scoped Use)

Filters are **limited to MVC only**.

### ✔ When to Use Filters:

* Controller-specific handling
* Business/domain exceptions
* Need access to MVC context (ModelState, etc.)

---

## 🧩 Example: Exception Filter

```csharp
public class GlobalExceptionFilter : IExceptionFilter
{
    private readonly ILogger<GlobalExceptionFilter> _logger;

    public GlobalExceptionFilter(ILogger<GlobalExceptionFilter> logger)
    {
        _logger = logger;
    }

    public void OnException(ExceptionContext context)
    {
        _logger.LogError(context.Exception, "Unhandled exception");

        context.Result = new ObjectResult(new
        {
            message = "An error occurred"
        })
        {
            StatusCode = 500
        };

        context.ExceptionHandled = true;
    }
}
```

---

## ⚙️ Register Filter

```csharp
services.AddControllers(options =>
{
    options.Filters.Add<GlobalExceptionFilter>();
});
```

---

## 🏁 Final Recommendation

* ✅ Use **middleware** for global exception handling
* ✅ Use **filters** for MVC-specific scenarios
* 🚫 Do NOT rely only on filters for global handling

---

# 🧠 ASP.NET Core Filters – Complete Guide

---

# 🔍 What are Filters?

Filters allow you to run code:

* Before or after execution stages
* Around **MVC controller actions**

---

# 🔄 Request Pipeline Position

```
Middleware → Routing → Filters → Controller → Filters → Response
```

### Key Idea:

* Middleware = Global
* Filters = MVC only

---

# 🎯 Types of Filters

---

## 1️⃣ Authorization Filters

* Run first
* Used for authentication/authorization

```csharp
public class CustomAuthFilter : IAuthorizationFilter
{
    public void OnAuthorization(AuthorizationFilterContext context)
    {
        if (!context.HttpContext.User.Identity.IsAuthenticated)
        {
            context.Result = new UnauthorizedResult();
        }
    }
}
```

---

## 2️⃣ Resource Filters

* Run before & after model binding
* Used for caching, performance

```csharp
public class ResourceFilter : IResourceFilter
{
    public void OnResourceExecuting(ResourceExecutingContext context)
    {
        // Before model binding
    }

    public void OnResourceExecuted(ResourceExecutedContext context)
    {
        // After execution
    }
}
```

---

## 3️⃣ Action Filters ⭐

* Most commonly used
* Run before & after controller actions

```csharp
public class LoggingActionFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        Console.WriteLine("Before action");
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        Console.WriteLine("After action");
    }
}
```

---

## 4️⃣ Exception Filters

* Handle exceptions inside controllers only

```csharp
public class ExceptionFilter : IExceptionFilter
{
    public void OnException(ExceptionContext context)
    {
        context.Result = new ObjectResult("Something went wrong")
        {
            StatusCode = 500
        };

        context.ExceptionHandled = true;
    }
}
```

---

## 5️⃣ Result Filters

* Run before & after result execution

```csharp
public class ResultFilter : IResultFilter
{
    public void OnResultExecuting(ResultExecutingContext context)
    {
        // Before response
    }

    public void OnResultExecuted(ResultExecutedContext context)
    {
        // After response
    }
}
```

---

# 📍 Filter Scopes

---

## 🌐 Global

```csharp
services.AddControllers(options =>
{
    options.Filters.Add<LoggingActionFilter>();
});
```

---

## 🏢 Controller Level

```csharp
[ServiceFilter(typeof(LoggingActionFilter))]
public class HomeController : Controller
{
}
```

---

## 🎯 Action Level

```csharp
[ServiceFilter(typeof(LoggingActionFilter))]
public IActionResult Index()
{
    return View();
}
```

---

# ⚙️ Filter Implementation Types

---

## 1. Interface-Based

* IActionFilter
* IExceptionFilter
* etc.

---

## 2. Attribute-Based

```csharp
public class MyFilterAttribute : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        // Logic here
    }
}
```

---

# 🔁 Execution Order

```
Authorization → Resource → Action → Result → Exception
```

Filters wrap around each other like layers.

---

# ⚖️ Filters vs Middleware

| Feature            | Filters            | Middleware      |
| ------------------ | ------------------ | --------------- |
| Scope              | MVC only           | Entire pipeline |
| Use case           | Action-level logic | Global logic    |
| Exception handling | Limited            | Best            |
| MVC context access | Yes                | No              |

---

# 💡 When to Use Filters

* Logging at action level
* Validation
* Modifying inputs/outputs
* Handling known exceptions

---

# 🚫 When NOT to Use Filters

* Global exception handling
* Authentication (use built-in system)
* Non-MVC logic

---

# 🧪 Real-World Setup

* Middleware → Global exception handling
* Filters → Logging, validation
* Exception filters → Business exceptions

---

# 🚀 Mental Model

* Middleware = Outer layer (global)
* Filters = Inner layer (MVC)
* Controller = Core logic

---

# ✅ Summary

| Concern               | Recommended Approach   |
| --------------------- | ---------------------- |
| Global errors         | Middleware             |
| MVC-specific handling | Filters                |
| Best practice         | Use both appropriately |

---

**End of Document**
