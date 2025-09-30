# Entity Framework Core – Complete Guide + Example

This document is a compact but complete guide to **Entity Framework Core**.  
It covers required NuGet packages, EF concepts, Fluent API, and a full working example (`AppDbContext.cs`).

---

## 1. NuGet Packages

Install the following packages depending on your project:

- **Microsoft.EntityFrameworkCore**  
  Core runtime. Defines `DbContext`, `DbSet<T>`, change tracking, LINQ translation.

- **Database providers** (pick one):
  - `Microsoft.EntityFrameworkCore.Sqlite` – SQLite support (great for demos/tests).
  - `Microsoft.EntityFrameworkCore.SqlServer` – SQL Server & Azure SQL.
  - `Npgsql.EntityFrameworkCore.PostgreSQL` – PostgreSQL.
  - `Pomelo.EntityFrameworkCore.MySql` – MySQL/MariaDB.

- **Microsoft.EntityFrameworkCore.Tools**  
  Command-line tooling (`dotnet ef migrations add`, `dotnet ef database update`).

- **Microsoft.EntityFrameworkCore.Design**  
  Design-time support (used by Visual Studio and CLI scaffolding/migrations).

---

## 2. Core EF Core Concepts

- **Entity** → A class mapped to a table.  
- **DbSet<TEntity>** → Represents a table (`db.Books`).  
- **DbContext** → Database session; creates queries and saves changes.  
- **Primary Key** → Every entity needs one (`Id` is conventional).  
- **Relationships**:  
  - One-to-Many (`Author` → `Books`)  
  - Many-to-Many (`Books` ↔ `Tags`)  
  - One-to-One (rare, but supported)  
- **Owned Types** → Value objects without their own identity, tied to an entity (e.g. `Address`).  
- **Shadow Properties** → Properties stored in DB but not on the class (e.g. `IsDeleted`).  
- **Global Query Filters** → Automatic filters for queries (e.g. exclude soft-deleted rows).  
- **Migrations** → Version control for schema.  
  ```bash
  dotnet ef migrations add InitialCreate
  dotnet ef database update
  ```  
- **Change Tracking** → EF monitors entity state: Added, Modified, Deleted, Unchanged.  
- **LINQ Queries** → Translate into SQL.  
- **Concurrency Tokens** → Detect conflicting updates (`RowVersion`).  
- **Transactions** → Ensure multiple operations succeed/fail together.

---

## 3. Fluent API Quick Reference

Configure entities inside `OnModelCreating(ModelBuilder b)`.

### Tables & Keys
```csharp
cfg.ToTable("TableName");
cfg.HasKey(e => e.Id);
```

### Properties
```csharp
cfg.Property(e => e.Name).IsRequired().HasMaxLength(200);
cfg.Property<DateTime>("CreatedUtc").HasDefaultValueSql("CURRENT_TIMESTAMP");
cfg.Property<byte[]>("RowVersion").IsRowVersion().IsConcurrencyToken();
```

### Relationships
- **One-to-Many**
```csharp
cfg.HasOne(e => e.Author)
   .WithMany(a => a.Books)
   .HasForeignKey(e => e.AuthorId)
   .OnDelete(DeleteBehavior.Cascade);
```

- **Many-to-Many**
```csharp
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
    owned.Property<int>("Id");
    owned.HasKey("Id");
});
```

### Indexes
```csharp
cfg.HasIndex(e => e.Title).HasDatabaseName("IX_Books_Title");
```

### Global Filters
```csharp
cfg.HasQueryFilter(e => EF.Property<bool>(e, "IsDeleted") == false);
```

### Seeding
```csharp
cfg.HasData(new Book { Id = 1, Title = "1984", AuthorId = 1 });
```

---

## 4. Full Example: `AppDbContext.cs`

```csharp
// AppDbContext.cs
using Microsoft.EntityFrameworkCore;
using System;
using System.Collections.Generic;

public enum BookStatus { Draft = 0, Published = 1 }

// ----- Domain -----
public class Author
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public ICollection<Address> Addresses { get; set; } = new List<Address>(); // many addresses
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
    public int AuthorId { get; set; }
    public Author Author { get; set; } = default!;
    public ICollection<Tag> Tags { get; set; } = new List<Tag>();
}
public class Tag
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public ICollection<Book> Books { get; set; } = new List<Book>();
}

// ----- DbContext -----
public class AppDbContext : DbContext
{
    public DbSet<Author> Authors => Set<Author>();
    public DbSet<Book> Books => Set<Book>();
    public DbSet<Tag> Tags => Set<Tag>();

    protected override void OnConfiguring(DbContextOptionsBuilder options)
        => options.UseSqlite("Data Source=efmini.db");

    protected override void OnModelCreating(ModelBuilder b)
    {
        // -------- Author --------
        b.Entity<Author>(cfg =>
        {
            cfg.ToTable("Authors");
            cfg.HasKey(a => a.Id);
            cfg.Property(a => a.Name).IsRequired().HasMaxLength(200);

            // OwnsMany: Author has many Addresses
            cfg.OwnsMany(a => a.Addresses, owned =>
            {
                owned.ToTable("AuthorAddresses");
                owned.WithOwner().HasForeignKey("AuthorId");
                owned.Property<int>("Id");
                owned.HasKey("Id");
                owned.Property(p => p.City).HasMaxLength(100).IsRequired(false);
                owned.Property(p => p.Country).HasMaxLength(100).IsRequired(false);

                owned.HasData(
                    new { Id = 1, AuthorId = 1, City = "Motihari", Country = "India" },
                    new { Id = 2, AuthorId = 2, City = "Godalming", Country = "UK" }
                );
            });

            cfg.HasData(
                new Author { Id = 1, Name = "George Orwell" },
                new Author { Id = 2, Name = "Aldous Huxley" }
            );
        });

        // -------- Book --------
        b.Entity<Book>(cfg =>
        {
            cfg.ToTable("Books");
            cfg.HasKey(x => x.Id);
            cfg.HasIndex(x => x.Title).HasDatabaseName("IX_Books_Title");

            cfg.HasOne(x => x.Author)
               .WithMany(a => a.Books)
               .HasForeignKey(x => x.AuthorId)
               .OnDelete(DeleteBehavior.Cascade);

            cfg.Property(x => x.Status)
               .HasConversion<string>()
               .HasMaxLength(32);

            cfg.Property(x => x.CreatedUtc)
               .HasDefaultValueSql("CURRENT_TIMESTAMP");

            cfg.Property(x => x.RowVersion)
               .IsRowVersion()
               .IsConcurrencyToken();

            cfg.Property<bool>("IsDeleted").HasDefaultValue(false);
            cfg.HasQueryFilter(x => EF.Property<bool>(x, "IsDeleted") == false);

            cfg.HasData(
                new Book { Id = 1, Title = "1984", AuthorId = 1, Status = BookStatus.Published },
                new Book { Id = 2, Title = "Animal Farm", AuthorId = 1, Status = BookStatus.Published },
                new Book { Id = 3, Title = "Brave New World", AuthorId = 2, Status = BookStatus.Published }
            );
        });

        // -------- Tag --------
        b.Entity<Tag>(cfg =>
        {
            cfg.ToTable("Tags");
            cfg.HasKey(t => t.Id);
            cfg.Property(t => t.Name).IsRequired().HasMaxLength(64);

            cfg.HasData(
                new Tag { Id = 1, Name = "Dystopia" },
                new Tag { Id = 2, Name = "Classic" }
            );

            cfg.HasMany(t => t.Books)
              .WithMany(bk => bk.Tags)
              .UsingEntity<Dictionary<string, object>>(
                  "BookTags",
                  right => right.HasOne<Book>().WithMany().HasForeignKey("BookId").OnDelete(DeleteBehavior.Cascade),
                  left  => left.HasOne<Tag>().WithMany().HasForeignKey("TagId").OnDelete(DeleteBehavior.Cascade),
                  join =>
                  {
                      join.ToTable("BookTags");
                      join.HasKey("BookId", "TagId");
                      join.HasData(
                          new { BookId = 1, TagId = 1 },
                          new { BookId = 1, TagId = 2 },
                          new { BookId = 3, TagId = 1 }
                      );
                  });
        });
    }
}
```

---

## 5. Usage Example

```csharp
using var db = new AppDbContext();
db.Database.Migrate();

// Create
db.Authors.Add(new Author { Name = "Ray Bradbury", Addresses = { new Address { City="LA", Country="USA" } } });
db.SaveChanges();

// Read with Include
var books = db.Books.Include(b => b.Author).Include(b => b.Tags).ToList();

// Update
var book = db.Books.First();
book.Title = "Edited Title";
db.SaveChanges();

// Soft delete
db.Entry(book).Property("IsDeleted").CurrentValue = true;
db.SaveChanges();
```
