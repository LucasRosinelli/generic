# Generic

## .NET 6 and EntityFramework
- References
  - https://www.c-sharpcorner.com/article/asp-net-core-entity-framework-call-store-procedure/
  - https://referbruv.com/blog/working-with-stored-procedures-in-aspnet-core-ef-core/
  - https://www.c-sharpcorner.com/article/apiasp-net-core-web-api-entity-framewor-call-stored-procedure-part-ii/
  - https://docs.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/work-with-data-in-asp-net-core-apps
  - https://stackoverflow.com/questions/20901419/how-to-call-stored-procedure-in-entity-framework-6-code-first
- NuGet packages
  - Microsoft.EntityFrameworkCore.SqlServer
  - Microsoft.EntityFrameworkCore.Tools
  - Microsoft.EntityFrameworkCore.SqlServer.Design
- Package manager console
  - `Scaffold-DbContext "THE_CONNECTION_STRING" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models/DB`
- `DbContext`:
  ```csharp
  namespace Cadagil.PoC.TestsEntityFramework.Api.Infrastructure;

  public partial class TestsEntityFrameworkContext : DbContext
  {
      public TestsEntityFrameworkContext()
      {
      }

      public TestsEntityFrameworkContext(DbContextOptions<TestsEntityFrameworkContext> options)
          : base(options)
      {
      }
   
      public virtual DbSet<MyTest> MyTests { get; set; } = null!;
  
      protected override void OnModelCreating(ModelBuilder modelBuilder)
      {
          modelBuilder.Entity<MyTest>(entity =>
          {
              entity.Property(e => e.Description)
                  .HasMaxLength(120)
                  .IsUnicode(false);

              entity.Property(e => e.Name)
                  .HasMaxLength(20)
                  .IsUnicode(false);
          });

          OnModelCreatingPartial(modelBuilder);
      }

      partial void OnModelCreatingPartial(ModelBuilder modelBuilder);

      public Task<List<MyTest>> ListAllAsync()
      {
          return MyTests.FromSqlRaw("EXEC ListTests").ToListAsync();
      }

      public async Task<MyTest?> GetByIdAsync(int id)
      {
          var idParam = new SqlParameter("@Id", SqlDbType.Int);
          idParam.Value = id;

          var result = await MyTests.FromSqlRaw("EXEC ListTestsById @Id", idParam).ToListAsync();
          return result.FirstOrDefault();
      }
  }
  ```
- `Startup`:
  ```csharp
  builder.Services.AddSqlServer<TestsEntityFrameworkContext>(builder.Configuration.GetConnectionString("DefaultConnection"));
  ```
- `appSettings.json`:
  ```json
  {
    "Logging": {
      "LogLevel": {
        "Default": "Information",
        "Microsoft.AspNetCore": "Warning"
      }
    },
    "AllowedHosts": "*"
  }
  ```
- `appSettings.Development.json`
  ```json
  {
    "Logging": {
      "LogLevel": {
        "Default": "Information",
        "Microsoft.AspNetCore": "Warning"
      }
    },
    "ConnectionStrings": {
      "DefaultConnection": "Server=localhost;Database=TestsEntityFramework;User Id=sa;Password=Pa55w0rd;"
    }
  }
  ```
- `Properties/launchSettings.json`:
  ```json
  {
    "$schema": "https://json.schemastore.org/launchsettings.json",
    "iisSettings": {
      "windowsAuthentication": false,
      "anonymousAuthentication": true,
      "iisExpress": {
        "applicationUrl": "http://localhost:9517",
        "sslPort": 44313
      }
    },
    "profiles": {
      "Cadagil.PoC.TestsEntityFramework.Api": {
        "commandName": "Project",
        "dotnetRunMessages": true,
        "launchBrowser": true,
        "launchUrl": "swagger",
        "applicationUrl": "https://localhost:7039;http://localhost:5039",
        "environmentVariables": {
          "ASPNETCORE_ENVIRONMENT": "Development"
        }
      },
      "IIS Express": {
        "commandName": "IISExpress",
        "launchBrowser": true,
        "launchUrl": "swagger",
        "environmentVariables": {
          "ASPNETCORE_ENVIRONMENT": "Development"
        }
      }
    }
  }
  ```
