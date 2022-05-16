# generic

## .NET 6 and EntityFramework
- References
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
          return MyTests.FromSqlRaw($"EXEC ListTests").ToListAsync();
      }
  }
  ```
- `Startup`:
  ```csharp
  builder.Services.AddSqlServer<TestsEntityFrameworkContext>(builder.Configuration.GetConnectionString("DefaultConnection"));
  ```
- `appSettings.Development.json`
  ```xml
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=TestsEntityFramework;User Id=sa;Password=Pa55w0rd;"
  }
  ```
