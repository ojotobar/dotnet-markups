"BaseUrl": "https://api.weatherapi.com",
"ApiKey": "1a134b3c4f20405aa33230938250710"

builder.Services.AddHttpClient("WeatherForecastService", client =>
{
    client.BaseAddress = new Uri(builder.Configuration["WeatherApi:BaseUrl"]!);
    client.Timeout = TimeSpan.FromMinutes(2);
});

builder.Services.AddSingleton<IWeatherForecastService, WeatherForecastService>();

 private readonly IHttpClientFactory _clientFactory;
 private readonly string _apiKey;

 public WeatherForecastService(IHttpClientFactory clientFactory, IConfiguration configuration)
 {
     _clientFactory = clientFactory;
     _apiKey = configuration["WeatherApi:ApiKey"] ?? 
         throw new ArgumentNullException(nameof(configuration));
 }

 public async Task<WeatherDto?> GetForecast(string? location)
 {
     var client = _clientFactory.CreateClient("WeatherForecastService");
     var response = await client.GetAsync($"/v1/current.json?key={_apiKey}&q={location}");
     if (response == null || !response.IsSuccessStatusCode)
     {
         return null;
     }

     var weatherForecast = await response.Content.ReadFromJsonAsync<WeatherForecast>();
     if (weatherForecast == null)
     {
         return null;
     }

     return WeatherForecast.ToWeatherDto(weatherForecast);
 }

 public WeatherLocation? Location { get; set; }
public CurrentWeather? Current { get; set; }

public static WeatherDto? ToWeatherDto(WeatherForecast weatherForecast)
{
    return new WeatherDto
    {
        Current = weatherForecast.Current != null ? new CurrentWeatherDto
        {
            FeelsLikeC = weatherForecast.Current.FeelsLikeC,
            FeelsLikeF = weatherForecast.Current.FeelsLikeF,
            TemperatureC = weatherForecast.Current.TemperatureC,
            TemperatureF = weatherForecast.Current.TemperatureF,
            Humidity = weatherForecast.Current.Humidity,
            WindMPH = weatherForecast.Current.WindMPH,
            WindKPH = weatherForecast.Current.WindKPH,
            WindDirection = weatherForecast.Current.WindDirection,
            UV = weatherForecast.Current.UV,
            IsDayTime = weatherForecast.Current.IsDayTime,
            LastUpdated = weatherForecast.Current.LastUpdated
        } : null,
        Location = weatherForecast.Location != null ? new WeatherLocationDto
        {
            Name = weatherForecast.Location.Name,
            Country = weatherForecast.Location.Country,
            Latitude = weatherForecast.Location.Latitude,
            LocalTime = weatherForecast.Location.LocalTime,
            Longitude = weatherForecast.Location.Longitude,
            TimeZoneId = weatherForecast.Location.TimeZoneId
        } : null
    };
}

public class WeatherLocation
{
    public string Name { get; set; } = string.Empty;
    public string Country { get; set; } = string.Empty;
    [JsonPropertyName("lon")]
    public double Longitude { get; set; }
    [JsonPropertyName("lat")]
    public double Latitude { get; set; }
    [JsonPropertyName("tz_id")]
    public string TimeZoneId { get; set; } = string.Empty;
    public string LocalTime { get; set; } = string.Empty;
}

 public class CurrentWeather
 {
     [JsonPropertyName("temp_c")]
     public double TemperatureC { get; set; }
     [JsonPropertyName("temp_f")]
     public double TemperatureF { get; set; }
     [JsonIgnore]
     [JsonPropertyName("is_day")]
     public byte IsDay { get; set; }
     public bool IsDayTime =>
         IsDay > 0 ? true : false;
     [JsonPropertyName("wind_mph")]
     public double WindMPH { get; set; }
     [JsonPropertyName("wind_kph")]
     public double WindKPH { get; set; }
     [JsonPropertyName("wind_dir")]
     public string WindDirection { get; set; } = string.Empty;
     public double Humidity { get; set; }
     [JsonPropertyName("feelslike_f")]
     public double FeelsLikeF { get; set; }
     [JsonPropertyName("feelslike_c")]
     public double FeelsLikeC { get; set; }
     public double UV { get; set; }
     [JsonPropertyName("last_updated")]
     public string LastUpdated { get; set; } = string.Empty;
 }