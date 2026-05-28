=========================================================
MINI PORADNIK - MINIMUM DO ZALICZENIA WEB API (DATABASE FIRST)
=========================================================

KROK 1: NIEZBĘDNE PACZKI NUGET
Zainstaluj te 3 paczki przez terminal lub okno NuGet, aby móc połączyć się z bazą:
1. Microsoft.EntityFrameworkCore.SqlServer
2. Microsoft.EntityFrameworkCore.Design
3. Microsoft.EntityFrameworkCore.Tools

---------------------------------------------------------

KROK 2: WYGENEROWANIE MODELI Z BAZY (SCAFFOLDING)
Otwórz terminal w Riderze i wklej tę komendę (jeśli masz użyć bazy 'master'):
dotnet ef dbcontext scaffold "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=master;Integrated Security=True;TrustServerCertificate=True" Microsoft.EntityFrameworkCore.SqlServer -o Models -c HospitalDbContext

---------------------------------------------------------

KROK 3: KONFIGURACJA appsettings.json
Dodaj sekcję "ConnectionStrings" z podwójnymi ukośnikami (\\):

{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=(localdb)\\MSSQLLocalDB;Initial Catalog=master;Integrated Security=True;TrustServerCertificate=True"
  }
}

---------------------------------------------------------

KROK 4: KONFIGURACJA Program.cs (Z TRIKIEM NA ZAPĘTLENIA)
Podmień zawartość Program.cs na to (zmień 'TwojaNazwaProjektu'):

using TwojaNazwaProjektu.Models;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Dodanie kontrolerów + TRIK: Ignorowanie cykli, żeby można było zwracać encje prosto z bazy bez DTO
builder.Services.AddControllers().AddJsonOptions(options =>
{
    options.JsonSerializerOptions.ReferenceHandler = System.Text.Json.Serialization.ReferenceHandler.IgnoreCycles;
});

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Rejestracja bazy danych
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
builder.Services.AddDbContext<HospitalDbContext>(options =>
    options.UseSqlServer(connectionString));

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.MapControllers();
app.Run();

---------------------------------------------------------

KROK 5: MINIMALNY DZIAŁAJĄCY KONTROLER
Dodaj plik MinimalController.cs w folderze Controllers:

using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using TwojaNazwaProjektu.Models; // Zmień na swoją nazwę

namespace TwojaNazwaProjektu.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class MinimalController : ControllerBase
    {
        private readonly HospitalDbContext _context;

        public MinimalController(HospitalDbContext context)
        {
            _context = context;
        }

        [HttpGet]
        public async Task<IActionResult> Get()
        {
            // Zwraca wszystko z tabeli bez pisania DTO (działa dzięki trikowi w Program.cs)
            return Ok(await _context.Patients.ToListAsync());
        }
    }
}
# abcd

{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=(localdb)\\MSSQLLocalDB;Initial Catalog=[ZMIEN_TO_NAZWA_BAZY];Integrated Security=True;TrustServerCertificate=True"
  }
}






using [ZMIEN_TO_NAZWA_PROJEKTU].Models; 
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Dodanie kontrolerów i Swaggera
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Podłączenie bazy danych z użyciem EF Core
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
builder.Services.AddDbContext<[ZMIEN_TO_NAZWA_KONTEKSTU]>(options =>
    options.UseSqlServer(connectionString));

var app = builder.Build();

// Uruchomienie Swaggera
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.MapControllers();
app.Run();


namespace [ZMIEN_TO_NAZWA_PROJEKTU].DTOs
{
    public class [ZMIEN_TO_NAZWA_MODELU]Dto
    {
        // Tutaj wpisujesz tylko te właściwości, które mają być zwrócone w odpowiedzi (np. Id, Name)
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
    }
}







using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using [ZMIEN_TO_NAZWA_PROJEKTU].Models;
using [ZMIEN_TO_NAZWA_PROJEKTU].DTOs;

namespace [ZMIEN_TO_NAZWA_PROJEKTU].Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class [ZMIEN_TO_NAZWA]Controller : ControllerBase
    {
        private readonly [ZMIEN_TO_NAZWA_KONTEKSTU] _context;

        public [ZMIEN_TO_NAZWA]Controller([ZMIEN_TO_NAZWA_KONTEKSTU] context)
        {
            _context = context;
        }

        // --- PRZYKŁAD GET (Pobieranie listy z filtrowaniem) ---
        [HttpGet]
        public async Task<IActionResult> GetAll([FromQuery] string? searchFilter)
        {
            var query = _context.[ZMIEN_TO_TABELA_W_BAZIE].AsQueryable();

            // Opcjonalne filtrowanie
            if (!string.IsNullOrWhiteSpace(searchFilter))
            {
                query = query.Where(x => x.Name.Contains(searchFilter)); 
            }

            // Mapowanie na DTO i pobranie
            var result = await query.Select(x => new [ZMIEN_TO_NAZWA_MODELU]Dto
            {
                Id = x.Id,
                Name = x.Name
            }).ToListAsync();

            return Ok(result);
        }

        // --- PRZYKŁAD POST (Dodawanie nowego rekordu) ---
        [HttpPost]
        public async Task<IActionResult> Create([FromBody] [ZMIEN_TO_NAZWA_MODELU]Dto request)
        {
            // Opcjonalna walidacja biznesowa (np. sprawdzenie, czy już istnieje)
            var exists = await _context.[ZMIEN_TO_TABELA_W_BAZIE].AnyAsync(x => x.Name == request.Name);
            if (exists)
            {
                return Conflict("Obiekt o podanej nazwie już istnieje.");
            }

            // Tworzenie nowej encji
            var newEntity = new [ZMIEN_TO_MODEL_Z_BAZY]
            {
                Name = request.Name
            };

            _context.[ZMIEN_TO_TABELA_W_BAZIE].Add(newEntity);
            await _context.SaveChangesAsync();

            return Created("", newEntity.Id);
        }
    }
}
