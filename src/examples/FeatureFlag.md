# Feature Flag in Csharp
This provides a way to toggle Features without re-compiling or even **restarting** the service.

## Add the package
Install `Microsoft.FeatureManagement.AspNetCore`

## The base min-api-template:
```cs
using Microsoft.FeatureManagement;
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// inject our FeatureManager
builder.Services.AddFeatureManagement();

var app = builder.Build();
if (app.Environment.IsDevelopment()) {
    app.UseSwagger();
    app.UseSwaggerUI();
}
app.UseHttpsRedirection();
var summaries = new[] {
    "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
};

// we pass the injected featureManager to the handler:
app.MapGet("/weatherforecast", async (IFeatureManager featureManager) =>
    {
        // first we check for the current state of our flag:
        var isFlag10Weathers = await featureManager.IsEnabledAsync(FeatureFlags.TenWeathers);
        
        // if isFlag we return 10 weathers, otherwise the default 5
        var forecast = Enumerable.Range(1, isFlag10Weathers ? 10 : 5).Select(index =>
                new WeatherForecast
                (
                    DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
                    Random.Shared.Next(-20, 55),
                    summaries[Random.Shared.Next(summaries.Length)]
                ))
            .ToArray();
        return forecast;
    })
    .WithName("GetWeatherForecast")
    .WithOpenApi();

app.Run();
internal record WeatherForecast(DateOnly Date, int TemperatureC, string? Summary) {
    public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
}

// we could pass plain strings into .IsEnabledAsync("TenWeathers")
// - but cleaner to have them all in one Class:
public class FeatureFlags {
    public const string TenWeathers = "Tenweathers";
}
```

- `appsettings.json`

The default configuration provider is `appsettings.json`. We just add the current value for the flag: 
```json
{
  "FeatureManagement": {
    "TenWeathers": false
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

### More Examples
We can toggle certain middleware with those 2 methods:
- `app.UseForFeature()` 
- `app.UseMiddleWareForFeature<>`

Minimal Apis can also use toggleable EndpointFilters.
```csharp
```

### Example for MVC Controllers
```cs
// you can toggle Existence of whole Controllers
[FeatureGate("isThisControllerActive")]
[ApiController]
public class ExampleController: ControllerBase
{
    // but also for single Actions
    [FeatureGate("isThisActionActive")]
    public IActionResult Action()
    {
        return Ok();
    }
}
```

### Endpoint Filter Example
- `FeatureFlagEndpointFilters.cs`
```cs
using Microsoft.FeatureManagement;

namespace minApiFeatureFlags;

// Extension to make registration of Filters easier:
public static class FeatureFlagEndpointFilterExtensions
{
    public static TBuilder WithFeatureFlag<TBuilder>(
        this TBuilder builder, string endpointName) 
        where TBuilder : IEndpointConventionBuilder
    {
        builder.AddEndpointFilter(new FeatureFlagEndpointFilters(endpointName));
        return builder;
    }
}

// Container for all our EndpointFilters:
public class FeatureFlagEndpointFilters : IEndpointFilter
{
    // to differentiate between endpoint-filters we pass down a identifier string:
    private readonly string _endpointName;

    public FeatureFlagEndpointFilters(string endpointName)
    {
        _endpointName = endpointName;
    }

    // if Flags are set we will pass context down the Pipeline
    // otherwise we return a generic 404
    public async ValueTask<object?> InvokeAsync(
        EndpointFilterInvocationContext context, 
        EndpointFilterDelegate next)
    {

        var featureManager = context.HttpContext
            .RequestServices.GetRequiredService<IFeatureManager>();
        
        var isEnabled = await featureManager
            .IsEnabledAsync($"Endpoints_{_endpointName}");

        if (!isEnabled)
        {
            return Results.NotFound();
        }

        return await next(context);
    }
}
```
- we create a new endpoint we want to hide with our filter implementation:
```cs
app.MapGet("/weatherforecastslim", (IFeatureManager featureManager) =>
    {
        return  Enumerable.Range(1, 5).Select(index =>
                new WeatherForecastSlim
                (
                    DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
                    Random.Shared.Next(-20, 55)
                ))
            .ToArray();
    })
    .WithFeatureFlag("GetWeatherForecastSlim")
    .WithName("GetWeatherForecastSlim")
    .WithOpenApi();

internal record WeatherForecastSlim(DateOnly Date, int TemperatureC) {
    public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
}
```


- and now we can toggle the flag in our appsettings:
```json
  "FeatureManagement": {
    "Endpoints_GetWeatherForecastSlim": false
  },
```

## More Advanced Filtering techniques

### Canary with a Percantagefilter
Canary ish functionality. Immagine we want to test a feature on only a percantage of requests. While monitoring closely if we break something (or if performance improves etc.)

```cs
builder.Services.AddFeatureManagement()
    .AddFeatureFilter<Percantagefilter>();
//    .AddSessionManager<>              // for more features
```
- now we could specify to only enable the new slim variant for 50% of all requests:
```json
  "FeatureManagement": {
    "Endpoints_GetWeatherForecastSlim": {
        "EnabledFor": [
            {
                "Name": "Percantage",
                "Parameters":{
                    "Value": 50
                }
            }
        ]
    }
  },
```

### TargetingFilter
- example only target non subscription users. Or Only users that are in the platform for over 2 years. To access those features.
```cs
builder.Services.AddFeatureManagement().AddFeatureFilter<TargetingFilter>();
```

### TimeWindowFilter
- Feature is only available for a limited time period.
```cs
builder.Services.AddFeatureManagement().AddFeatureFilter<TimeWindowFilter>();
```