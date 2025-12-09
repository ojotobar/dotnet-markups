```csharp
var options = new JsonSerializerOptions
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    PropertyNameCaseInsensitive = true,
    DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull,
};

// Refit settings using System.Text.Json with the options above
var refitSettings = new RefitSettings
{
    ContentSerializer = new SystemTextJsonContentSerializer(options)
};

services.AddRefitClient<ILocationClientService>(refitSettings)
    .ConfigureHttpClient(c =>
    { 
        c.BaseAddress = new Uri(servers.ExternalLocationClient);
        c.Timeout = TimeSpan.FromSeconds(120);
    }).AddHttpMessageHandler(() => new ServiceRefitWakeUpHandler());

return services;

 /// <summary>
 /// Gets user by id
 /// </summary>
 /// <param name="id"></param>
 /// <returns></returns>
 [Get("/api/v1/user/{id}")]
 Task<ApiResponse<ApiResult<UserDto>>> GetUserById(string id);

 /// <summary>
 /// Gets list of users by ids
 /// </summary>
 /// <param name="ids"></param>
 /// <returns></returns>
 [Post("/api/v1/user")]
 Task<ApiResponse<ApiResult<List<UserDto>>>> GetUsersByIds([Body] List<string> ids);
```
---

```csharp
 public static void ConfigureHttpClient(this IServiceCollection services,
    IConfiguration configuration)
{
    var httpClient = new HttpClient
    {
        BaseAddress = new Uri(configuration["CSAPI:BaseAddress"]!),
        Timeout = TimeSpan.FromMinutes(5)
    };

    httpClient.DefaultRequestHeaders.Add("X-CSCAPI-KEY", configuration["CSAPI:APIKEY"]);
    services.AddSingleton(httpClient);
}

private readonly HttpClient _httpClient = httpClient;
private readonly ILogger<LocationFromClientRepository> _logger = logger;

public async Task<(List<CountryFromClient> Countries, bool Success, HttpStatusCode Code)> GetCountriesAsync()
{
    var response = await _httpClient.GetAsync("");
    if (response.IsSuccessStatusCode)
    {
        var countries = await response.Content.ReadFromJsonAsync<List<CountryFromClient>>();
        return (countries ?? [], true, HttpStatusCode.OK);
    }
    else
    {
        string errorMessage = await response.Content.ReadAsStringAsync();
        var error = $"Error: {response.StatusCode} - {errorMessage}";
        _logger.LogError(error);
        return ([], false, response.StatusCode);
    }
}

public async Task<(CountryFromClient? Country, bool Success, HttpStatusCode Code)> GetCountryAsync(string countryIso2)
{
    var response = await _httpClient.GetAsync($"{countryIso2}");
    if (response.IsSuccessStatusCode)
    {
        var country = await response.Content.ReadFromJsonAsync<CountryFromClient>();
        return (country, true, HttpStatusCode.OK);
    }
    else
    {
        string errorMessage = await response.Content.ReadAsStringAsync();
        var error = $"Error: {response.StatusCode} - {errorMessage}";
        _logger.LogError(error);
        return (null, false, response.StatusCode);
    }
}
```

--- 

```csharp

"CSAPI": {
  "APIKEY": "TjNXcHRXVWxkMWVyeVZSZTBsSGxxSWFnd3VETmF1b2lBazlxYnJZQw==",
  "BaseAddress": "https://api.countrystatecity.in/v1/countries/"
}
services.AddHttpClient("LocationService", client =>
{
    client.BaseAddress = new Uri(configuration["ExternalServices:LocationMarker"]!);
    client.Timeout = TimeSpan.FromMinutes(5);
    client.DefaultRequestHeaders.Add("X-CSCAPI-KEY", configuration["CSAPI:APIKEY"]);
});//.AddTransientHttpErrorPolicy(policy => policy.WaitAndRetryAsync(3, attempt => TimeSpan.FromSeconds(10)));

 services.AddHttpClient("LocationService", client =>
 {
     client.BaseAddress = new Uri(configuration["ExternalServices:LocationMarker"]!);
     client.Timeout = TimeSpan.FromMinutes(5);
 }).AddTransientHttpErrorPolicy(policy => policy.WaitAndRetryAsync(3, attempt => TimeSpan.FromSeconds(10)));


 private readonly IHttpClientFactory _httpClientFactory;
private readonly ILoggerManager _logger;

public LocationServiceClient(IHttpClientFactory httpClientFactory,
                         ILoggerManager logger)
{
    _httpClientFactory = httpClientFactory;
    _logger = logger;
}

public async Task<ApiBaseResponse> GetStatesForCountryV1Async(Guid countryId)
{
    var client = _httpClientFactory.CreateClient("LocationService");
    var response = await client.GetAsync($"/api/v1/location/country/{countryId}/states");
    if (response == null || !response.IsSuccessStatusCode)
    {
        return new BadRequestResponse($"Response with status code: {response?.StatusCode ?? 0}");
    }

    var states = await response.ReadFromJsonAsync<List<KwikNestaState>>();
    if (states == null)
    {
        return new NotFoundResponse("No state data found");
    }

    return new OkResponse<List<KwikNestaState>>(states);
}


```