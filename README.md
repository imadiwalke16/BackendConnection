1. Install Npgsql Package
Npgsql is the .NET provider for PostgreSQL.

Run the following command in your project directory:

sh
Copy
Edit
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
2. Configure appsettings.json
Modify your appsettings.json to include the PostgreSQL connection string:

json
Copy
Edit
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Port=5432;Database=mydb;Username=myuser;Password=mypassword"
  }
}
Replace:

localhost with your database server address
mydb with your database name
myuser and mypassword with your PostgreSQL credentials
3. Configure the Database Context (ApplicationDbContext.cs)
Create a new class that represents the database context:

csharp
Copy
Edit
using Microsoft.EntityFrameworkCore;

public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options) { }

    // Define your tables here
    public DbSet<User> Users { get; set; }
}

public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
}
4. Register Database in Program.cs
Modify the Program.cs file to add PostgreSQL support:

csharp
Copy
Edit
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;

var builder = WebApplication.CreateBuilder(args);

// Add PostgreSQL database context
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

app.UseSwagger();
app.UseSwaggerUI();

app.UseAuthorization();

app.MapControllers();

app.Run();
5. Create a Migration and Apply It
Run the following commands to create and apply database migrations:

sh
Copy
Edit
dotnet ef migrations add InitialCreate
dotnet ef database update
6. Create an API Controller
Generate a controller for CRUD operations:

csharp
Copy
Edit
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

[Route("api/[controller]")]
[ApiController]
public class UsersController : ControllerBase
{
    private readonly ApplicationDbContext _context;

    public UsersController(ApplicationDbContext context)
    {
        _context = context;
    }

    [HttpGet]
    public async Task<ActionResult<IEnumerable<User>>> GetUsers()
    {
        return await _context.Users.ToListAsync();
    }

    [HttpPost]
    public async Task<ActionResult<User>> CreateUser(User user)
    {
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
        return CreatedAtAction(nameof(GetUsers), new { id = user.Id }, user);
    }
}
7. Test Your API
Run the project:

sh
Copy
Edit
dotnet run
Then, test it using Postman or Swagger (https://localhost:5001/swagger).

8. Troubleshooting
Ensure PostgreSQL is running (systemctl status postgresql on Linux).
Check firewall and authentication settings.
Ensure the Npgsql package matches your EF Core version.
