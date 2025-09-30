# Entity Framework Core – Complete Guide + Example

This guide explains Entity Framework Core in a **practical, complete way**:  
- Required NuGet packages  
- Foreign key rules (implicit vs explicit)  
- Loading strategies (Eager / Explicit / Lazy)  
- Fluent API essentials  
- Full working `AppDbContext.cs` example  

---

## 1. NuGet Packages

Install the following NuGet packages:

- **Microsoft.EntityFrameworkCore**  
  Core runtime (`DbContext`, `DbSet<T>`, LINQ provider).

- **Database provider** (pick one):  
  - `Microsoft.EntityFrameworkCore.Sqlite` – SQLite (lightweight, file-based).  
  - `Microsoft.EntityFrameworkCore.SqlServer` – SQL Server / Azure SQL.  
  - `Npgsql.EntityFrameworkCore.PostgreSQL` – PostgreSQL.  
  - `Pomelo.EntityFrameworkCore.MySql` – MySQL / MariaDB.  

- **Microsoft.EntityFrameworkCore.Tools**  
  Enables `dotnet ef` commands for migrations and schema updates.

- **Microsoft.EntityFrameworkCore.Design**  
  Supports design-time tasks (scaffolding, migration generation).

---

## 2. Core Concepts

- **Entity** → C# class mapped to a database table.  
- **DbSet<TEntity>** → Represents a table (`db.Users`).  
- **DbContext** → Database session; builds queries and tracks changes.  
- **Primary Key** → Required for every entity (convention: `Id` or `EntityNameId`).  
- **Relationships**:
  - One-to-Many (`Author` → `Books`)  
  - Many-to-Many (`Student` ↔ `Course`)  
  - One-to-One (`Person` ↔ `Passport`)  
- **Owned Types** → Value objects bound to an entity (e.g., `Address`).  
- **Shadow Properties** → Exist in DB, not in C# (e.g., `IsDeleted`).  
- **Global Query Filters** → Auto-applied conditions (e.g., exclude soft-deleted).  
- **Change Tracking** → EF monitors entity states (Added, Modified, Deleted, Unchanged).  
- **Migrations** → Database schema evolution.  
  ```bash
  dotnet ef migrations add InitialCreate
  dotnet ef database update
  ```

---

## 3. Foreign Keys (FKs)

### Option 1 – Navigation only (implicit FK)

```csharp
public class Script
{
    public int Id { get; set; }
    public string Title { get; set; } = "";
    public Project Project { get; set; } = null!; // navigation only
}
```

- EF creates a `ProjectId` column automatically as a **shadow property**.  
- Query by navigation:
  ```csharp
  var scripts = db.Scripts.Where(s => s.Project.Id == 5);
  ```

### Option 2 – Navigation + explicit FK (recommended)

```csharp
public class Script
{
    public int Id { get; set; }
    public string Title { get; set; } = "";
    public int ProjectId { get; set; }       // explicit FK
    public Project Project { get; set; } = null!;
}
```

✅ Benefits of explicit FK:
- Easier queries (`s.ProjectId == 5`)  
- Better performance (no need to load navigation)  
- Clearer schema  

👉 **Rule of thumb**: For real apps, define FKs explicitly.

---

## 4. Loading Related Data

When you access related entities, EF can load them in 3 ways:

### 4.1 Eager Loading
Load related data immediately using `.Include()`:

```csharp
var projects = db.Projects
    .Include(p => p.Scripts)
    .ThenInclude(s => s.HistoryGenerates)
    .ToList();
```

- ✅ One query  
- ❌ Might load unnecessary data  

---

### 4.2 Explicit Loading
Load related data only when requested:

```csharp
var project = db.Projects.First(p => p.Id == 1);
db.Entry(project).Collection(p => p.Scripts).Load();
```

- ✅ Full control  
- ❌ Multiple queries  

---

### 4.3 Lazy Loading
Load related data automatically **when accessed**.

1. Install `Microsoft.EntityFrameworkCore.Proxies`  
2. Enable proxies:
   ```csharp
   options.UseLazyLoadingProxies().UseSqlite("Data Source=app.db");
   ```
3. Mark navigations as `virtual`.

```csharp
public virtual ICollection<Script> Scripts { get; set; }
```

Usage:
```csharp
var project = db.Projects.First();
Console.WriteLine(project.Scripts.Count); // triggers SQL query
```

- ✅ Very convenient  
- ❌ Risk of N+1 query problem  

---

## 5. Fluent API Essentials

Configure entities in `OnModelCreating(ModelBuilder b)`.

- **Tables & Keys**
  ```csharp
  cfg.ToTable("TableName");
  cfg.HasKey(e => e.Id);
  ```

- **Properties**
  ```csharp
  cfg.Property(e => e.Name).IsRequired().HasMaxLength(200);
  cfg.Property<DateTime>("CreatedUtc").HasDefaultValueSql("CURRENT_TIMESTAMP");
  ```

- **Relationships**
  ```csharp
  // One-to-Many
  cfg.HasOne(e => e.Project)
     .WithMany(p => p.Scripts)
     .HasForeignKey(e => e.ProjectId);

  // Many-to-Many
  cfg.HasMany(e => e.Tags)
     .WithMany(t => t.Books)
     .UsingEntity("BookTags");
  ```

- **Owned Types**
  ```csharp
  cfg.OwnsMany(a => a.Addresses, owned =>
  {
      owned.ToTable("AuthorAddresses");
      owned.WithOwner().HasForeignKey("AuthorId");
  });
  ```

- **Global Filters**
  ```csharp
  cfg.HasQueryFilter(e => EF.Property<bool>(e, "IsDeleted") == false);
  ```

- **Seeding**
  ```csharp
  cfg.HasData(new Book { Id = 1, Title = "1984", AuthorId = 1 });
  ```

---

## 6. Full Example: `AppDbContext.cs`

```csharp
// AppDbContext.cs
using Microsoft.EntityFrameworkCore;
using System;
using System.Collections.Generic;

public enum BookStatus { Draft = 0, Published = 1 }

public class Author
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public ICollection<Address> Addresses { get; set; } = new List<Address>();
    public ICollection<Book> Books { get; set; } = new List<Book>();
}

public class Address
{
    public string City { get; set; } = "";
    public string Country { get; set; } = "";
}

public class Book
{
    public int Id { get; set; }
    public string Title { get; set; } = "";
    public BookStatus Status { get; set; }
    public DateTime CreatedUtc { get; set; }
    public byte[] RowVersion { get; set; } = Array.Empty<byte>();

    public int AuthorId { get; set; }            // explicit FK
    public Author Author { get; set; } = null!;

    public ICollection<Tag> Tags { get; set; } = new List<Tag>();
}

public class Tag
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public ICollection<Book> Books { get; set; } = new List<Book>();
}

public class AppDbContext : DbContext
{
    public DbSet<Author> Authors => Set<Author>();
    public DbSet<Book> Books => Set<Book>();
    public DbSet<Tag> Tags => Set<Tag>();

    protected override void OnConfiguring(DbContextOptionsBuilder options)
        => options.UseSqlite("Data Source=efmini.db");

    protected override void OnModelCreating(ModelBuilder b)
    {
        // Author
        b.Entity<Author>(cfg =>
        {
            cfg.ToTable("Authors");
            cfg.HasKey(a => a.Id);
            cfg.Property(a => a.Name).IsRequired().HasMaxLength(200);

            cfg.OwnsMany(a => a.Addresses, owned =>
            {
                owned.ToTable("AuthorAddresses");
                owned.WithOwner().HasForeignKey("AuthorId");
                owned.Property<int>("Id");
                owned.HasKey("Id");
            });

            cfg.HasData(
                new Author { Id = 1, Name = "George Orwell" },
                new Author { Id = 2, Name = "Aldous Huxley" }
            );
        });

        // Book
        b.Entity<Book>(cfg =>
        {
            cfg.ToTable("Books");
            cfg.HasKey(x => x.Id);
            cfg.HasIndex(x => x.Title);

            cfg.HasOne(x => x.Author)
               .WithMany(a => a.Books)
               .HasForeignKey(x => x.AuthorId);

            cfg.Property(x => x.Status).HasConversion<string>();
            cfg.Property(x => x.CreatedUtc).HasDefaultValueSql("CURRENT_TIMESTAMP");
            cfg.Property(x => x.RowVersion).IsRowVersion().IsConcurrencyToken();
            cfg.Property<bool>("IsDeleted").HasDefaultValue(false);
            cfg.HasQueryFilter(x => EF.Property<bool>(x, "IsDeleted") == false);

            cfg.HasData(
                new Book { Id = 1, Title = "1984", AuthorId = 1, Status = BookStatus.Published },
                new Book { Id = 2, Title = "Animal Farm", AuthorId = 1, Status = BookStatus.Published },
                new Book { Id = 3, Title = "Brave New World", AuthorId = 2, Status = BookStatus.Published }
            );
        });

        // Tag
        b.Entity<Tag>(cfg =>
        {
            cfg.ToTable("Tags");
            cfg.HasKey(t => t.Id);
            cfg.Property(t => t.Name).IsRequired();

            cfg.HasMany(t => t.Books)
              .WithMany(bk => bk.Tags)
              .UsingEntity("BookTags");

            cfg.HasData(
                new Tag { Id = 1, Name = "Dystopia" },
                new Tag { Id = 2, Name = "Classic" }
            );
        });
    }
}
```

---

## 7. Example Usage

```csharp
using var db = new AppDbContext();
db.Database.Migrate();

// Create
db.Authors.Add(new Author { Name = "Ray Bradbury" });
db.SaveChanges();

// Eager load
var books = db.Books.Include(b => b.Author).Include(b => b.Tags).ToList();

// Explicit load
var author = db.Authors.First();
db.Entry(author).Collection(a => a.Books).Load();

// Lazy load (requires proxies + virtual)
var book = db.Books.First();
Console.WriteLine(book.Author.Name);
```
