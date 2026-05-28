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
