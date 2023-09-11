# AddSingleton() vs AddScoped() vs AddTransient()

### AddSingleton
`AddSingleton() `- our 2 services point to the same Singleton
- one gets mutated -> all do.

### AddScoped
`AddScoped()` - The Services gets created (then destroyed) for EVERY HTTP Request
- so no persistence between requests.
- BUT service1 == service2 both again point to the same 'singleton'-ish for that lifetime.
- We could make the `private static string[] _stages` **static** to persists the service.instance lifetimes.
     - then the Plants would increase agai
### AddTransient
`AddTransient()` - our 2 services are Separate
- again no persistence between requests. Lifetime coupled to Http-Request.
- one gets mutated then destroyed without touching the other.

## Example
- `Programm.cs`
```cs
// inject our Services:
// Singleton() - our 2 services point to the same Singleton
// - one gets mutated -> all do.
// builder.Services.AddSingleton<IPlantService, PlantService>();

// Scoped() - The Services gets created (then destroyed) for EVERY HTTP Request
// - so no persistence between requests.
// - BUT service1 == service2 both again point to the same 'singleton'-ish for that lifetime.
// - We could make the `private static string[] _stages` **static** to persists the service.instance lifetimes.
//      - then the Plants would increase again.
// builder.Services.AddScoped<IPlantService, PlantService>();

// Transient() - our 2 services are Separate
// - again no persistence between requests. Lifetime coupled to Http-Request.
// - one gets mutated then destroyed without touching the other.
builder.Services.AddTransient<IPlantService, PlantService>();

```

- `Controllers/PlantController.cs`
```cs
using Microsoft.AspNetCore.Mvc;
using tutorials.Models;
using tutorials.Services.PlantService;

namespace tutorials.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class PlantController : ControllerBase
    {
        private readonly IPlantService _plantService1;
        private readonly IPlantService _plantService2;

        public PlantController(IPlantService s1, IPlantService s2)
        {
            _plantService1 = s1;
            _plantService2 = s2;
        }

        [HttpGet("count")]
        public ActionResult<int> GetPlantsCount()
        {
            return _plantService2.GetPlantsCount();
        }

        [HttpGet]
        public ActionResult<List<Plant>> GetPlantsList()
        {
            return _plantService2.GetPlantsList();
        }

        [HttpPost]
        public ActionResult<List<Plant>> Grow()
        {
            _plantService1.GrowRandomPlant();
            return _plantService2.GetPlantsList();
        }
    }
}
```

-  `Services/PlantService/PlantService.cs `

```cs
using tutorials.Models;

namespace tutorials.Services.PlantService;

/// <summary>
/// Some basic Service that grows Trees and returns data about the Trees
/// </summary>
public class PlantService : IPlantService
{
    private static string[] _stages = { "Seed", "Germination", "Sapling", "Mature Tree", "Dying Tree", "Fire Wood", };

    public List<Plant> PlantsList = new()
    {
        new Plant(){ Stage = _stages[0]},
        new Plant(){ Stage = _stages[0]},
        new Plant(){ Stage = _stages[2]},
    };

    public int GetPlantsCount()=> PlantsList.Count;
    
    public List<Plant> GetPlantsList() => PlantsList;

    public void GrowRandomPlant()
    {
        PlantsList.Add(new Plant
        {
            Stage = _stages[new Random().Next(_stages.Length)]
        });
    }
}
```

-  `Services/PlantService/IPlantService.cs `
```cs
using tutorials.Models;
namespace tutorials.Services.PlantService;

// interface because we want to inject it to show different ways of doing so
public interface IPlantService
{
    int GetPlantsCount();
    List<Plant> GetPlantsList();
    void GrowRandomPlant();
}
```