Perfect! Let’s build a **production-ready ASP.NET Core example** showing both `HttpClient` and `Refit` for a location API. I’ll make it clean, DI-friendly, and ready for unit testing.

---

## **1. Project Setup**

* .NET 8 (or 7+) Web API.
* Install NuGet packages:

```bash
dotnet add package Refit
dotnet add package Microsoft.Extensions.Http.Polly
```

---

## **2. Models**

```csharp
public class LocationResult
{
    public string City { get; set; } = default!;
    public string Country { get; set; } = default!;
    public double Lat { get; set; }
    public double Lng { get; set; }
}
```

---

## **3. HttpClient Service**

```csharp
using System.Net.Http;
using System.Net.Http.Json;
using System.Threading.Tasks;

public interface ILocationHttpClientService
{
    Task<LocationResult?> GetLocationAsync(string address);
}

public class LocationHttpClientService : ILocationHttpClientService
{
    private readonly HttpClient _httpClient;
    private readonly string _apiKey;

    public LocationHttpClientService(HttpClient httpClient, IConfiguration config)
    {
        _httpClient = httpClient;
        _apiKey = config["LocationApi:ApiKey"] ?? throw new ArgumentNullException("API key missing");
    }

    public async Task<LocationResult?> GetLocationAsync(string address)
    {
        try
        {
            var url = $"geocode?address={Uri.EscapeDataString(address)}&key={_apiKey}";
            return await _httpClient.GetFromJsonAsync<LocationResult>(url);
        }
        catch (HttpRequestException ex)
        {
            Console.WriteLine($"HTTP Error: {ex.Message}");
            return null;
        }
    }
}
```

---

## **4. Refit Service**

```csharp
using Refit;
using System.Threading.Tasks;

public interface ILocationApi
{
    [Get("/geocode")]
    Task<LocationResult> GetLocationAsync([Query] string address, [Query] string key);
}
```

---

## **5. DI Setup (Program.cs)**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Configuration for API key
builder.Configuration.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true);

// HttpClient with Polly retry
builder.Services.AddHttpClient<ILocationHttpClientService, LocationHttpClientService>(client =>
{
    client.BaseAddress = new Uri("https://api.location.com/");
})
.AddTransientHttpErrorPolicy(p => p.RetryAsync(3)); // retry 3 times on failure

// Refit client
builder.Services.AddRefitClient<ILocationApi>()
    .ConfigureHttpClient(c => c.BaseAddress = new Uri("https://api.location.com/"));

var app = builder.Build();

app.MapGet("/location/httpclient", async (ILocationHttpClientService service, string address) =>
{
    var result = await service.GetLocationAsync(address);
    return result is null ? Results.NotFound() : Results.Ok(result);
});

app.MapGet("/location/refit", async (ILocationApi api, IConfiguration config, string address) =>
{
    var apiKey = config["LocationApi:ApiKey"]!;
    var result = await api.GetLocationAsync(address, apiKey);
    return Results.Ok(result);
});

app.Run();
```

---

## **6. appsettings.json**

```json
{
  "LocationApi": {
    "ApiKey": "YOUR_API_KEY"
  }
}
```