# ✅ Logging request duration, route & status code with Serilog.AspNetCore

---

# 1. Install Packages

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
```

(If you want correlation IDs)

```bash
dotnet add package Serilog.Enrichers.CorrelationId
```

---

# 2. Configure Serilog in `Program.cs` (Production-grade)

```csharp
using Serilog;
using Serilog.Events;
using Serilog.Formatting.Json;

Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .Enrich.FromLogContext()
    .WriteTo.Console()
    .WriteTo.File(
        new JsonFormatter(),
        "logs/requests-.json",
        rollingInterval: RollingInterval.Day,
        retainedFileCountLimit: 7)
    .CreateLogger();

builder.Host.UseSerilog();
```

✅ Logs as structured JSON
✅ Files are rotated daily
✅ Good for Seq / Elastic / Splunk

---

# 3. Enable HTTP Request Logging

This is where the magic happens.

In `Program.cs`:

```csharp
app.UseSerilogRequestLogging(options =>
{
    options.MessageTemplate =
        "HTTP {RequestMethod} {RequestPath} responded {StatusCode} in {Elapsed:0.0000} ms";

    options.EnrichDiagnosticContext = (diagnosticContext, httpContext) =>
    {
        diagnosticContext.Set("RequestHost", httpContext.Request.Host.Value);
        diagnosticContext.Set("Scheme", httpContext.Request.Scheme);
        diagnosticContext.Set("UserAgent", httpContext.Request.Headers["User-Agent"].ToString());
        diagnosticContext.Set("ClientIP", httpContext.Connection.RemoteIpAddress?.ToString());
        diagnosticContext.Set("Endpoint", httpContext.GetEndpoint()?.DisplayName);
    };
});
```

✅ Measures request time automatically
✅ Logs route, status code, and duration
✅ Attaches metadata

---

# 4. Sample Output (Real log)

Console log:

```text
HTTP GET /api/qrcode/12 responded 200 in 48.21 ms
```

JSON file log:

```json
{
  "RequestMethod": "GET",
  "RequestPath": "/api/qrcode/12",
  "StatusCode": 200,
  "Elapsed": 48.21,
  "ClientIP": "192.168.1.10",
  "Endpoint": "QRController.GetQRCode",
  "Timestamp": "2025-02-23T12:10:00Z"
}
```

---

# 5. Add Correlation ID (Optional but Pro)

Very useful when debugging distributed systems.

---

## Middleware for correlation ID

```csharp
app.Use(async (context, next) =>
{
    var correlationId = context.Request.Headers["X-Correlation-ID"].FirstOrDefault() 
                        ?? Guid.NewGuid().ToString();

    context.Response.Headers["X-Correlation-ID"] = correlationId;

    using (Serilog.Context.LogContext.PushProperty("CorrelationId", correlationId))
    {
        await next();
    }
});
```

Now every log entry contains:

```json
"CorrelationId": "21c0c2b9-0f1a-4f63-8fa4-9c2ea488f12a"
```

---

# 6. Manual Logging Correlates Automatically

Inside controllers:

```csharp
_logger.LogInformation("QR generated for user {UserId}", userId);
```

This log now includes:
✅ CorrelationId
✅ HTTP context info
✅ Request path & status

---

# 7. Logging level strategy (teach this)

| Level       | Use for                    |
| ----------- | -------------------------- |
| Debug       | Development noise          |
| Information | Requests, state changes    |
| Warning     | Unexpected but recoverable |
| Error       | Failures                   |
| Fatal       | App crashes                |

---

# 8. Bonus: Exclude noisy routes (e.g. /health)

```csharp
options.GetLevel = (context, elapsed, ex) =>
{
    if (context.Request.Path == "/health")
        return LogEventLevel.Verbose;

    return LogEventLevel.Information;
};
```

---

# 9. Minimum checklist for a production API

✅ Structured JSON logs
✅ Request timing
✅ Correlation ID
✅ File rotation
✅ Controlled verbosity
✅ No secrets in logs

---

# 10. Common Mistakes (warn students early)

❌ Logging passwords or tokens
❌ Logging full request bodies without filtering
❌ Writing logs only to console in production
✅ Solution → structured logging & filtering