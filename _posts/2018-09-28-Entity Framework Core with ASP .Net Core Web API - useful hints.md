---
title: "Entity Framework Core with ASP .Net Core Web API - useful hints"
excerpt: "In this, second article I would like to share some hints connected with Entity Framework Core."
---

<p align="center">
<img src="/images/devisland/article13/assets/efh1.png?raw=true" alt="Entity Framework Core with ASP .Net Core Web API - useful hints"/>
</p>

<h3><strong>Short introduction</strong></h3>
In my previous article available <a href="https://devislandblog.wordpress.com/2018/09/01/entity-framework-core-with-asp-net-core-web-api-jump-start/" target="_blank" rel="noopener">here</a> article I tried to briefly introduce you to Entity Framework Core object-relational mapper used together with ASP .NET Core Web API. I tried to include code samples, some best practices and some common pitfalls. In this, second article I would like to share some hints connected with Entity Framework Core.

I assume that you are familiar with the first article and you already have previous Visual Studio project we created. If not you can download it from my <a href="https://github.com/Daniel-Krzyczkowski/NetCsharp/tree/master/EntityFrameworkCoreJumpStart" target="_blank" rel="noopener">GitHub</a>. In this article we will extend this sample with some additional code.

&nbsp;
<h3><strong>IQueryable&lt;T&gt; vs IEnumerable&lt;T&gt;</strong></h3>
Please look at the lines below - we will use them in our code:

```csharp
IEnumerable<Car> enumerableOwners = _applicationDbContext.Cars;

IQueryable<Car> queryableOwners = _applicationDbContext.Cars;
```

Both lines of code are doing the same thing. Get all records from Cars table so at the end below SQL query will be executed:

<em>SELECT * FROM Cars</em>

Now look at the lines below. If we add some query to above example difference will be huge:

```csharp
var enumerableCar = _applicationDbContext.Cars.ToList().Where(x => x.Id == id).SingleOrDefault();
var queryableCar = _applicationDbContext.Cars.Where(x => x.Id == id).SingleOrDefault();
```

In the first line we applied <strong>local query</strong>. It means that all rows from Cars table will be loaded to the memory and then query will be executed (where condition). Here we should remember that for big collections it can be harmful and has big impact related with performance.

In the second line of above code we applied <strong>interpreted query</strong>. It means that Cars table will be filtered on the database side and only specific records will be returned for presented condition.

To sum up when working with big collections in the database we should always try to use IQueryable. I added each case in the code sample:

```csharp
        public async Task<Car> GetAsync(Guid id)
        {
            // All cars will be loaded to the memory first and then Where condition will be applied:
            var enumerableCar = _applicationDbContext.Cars.ToList().Where(x => x.Id == id).SingleOrDefault();

            //Cars table will be filtered on the database side and then specific car record will be returned:
            var queryableCar = _applicationDbContext.Cars.Where(x => x.Id == id).SingleOrDefault();

            // We can use below code to do filtering on the database side:
            var car = await _applicationDbContext.Cars
               .Where(x => x.Id == id)
               .SingleOrDefaultAsync();

            // We can also shorten above query and move condition from Where to SingleOrDefault method:
            return await _applicationDbContext.Cars
               .SingleOrDefaultAsync(x => x.Id == id);
        }
```

<h3></h3>
<h3><strong>Using Linq Experssions</strong></h3>
Besides Entity Framework it is good to know a little bit about Linq.Expressions. This is why I recommend you to first read great article -  <a href="https://blogs.msdn.microsoft.com/charlie/2008/01/31/expression-tree-basics/" target="_blank" rel="noopener">Expression Tree Basics</a>.

Sometimes you would like to transform model returned from the database to the Data Transfer Object returned from your Web API. Lets look at the above example. I extended "IEntity" interafce with additional properties:
<ul>
 	<li>CreatedBy</li>
 	<li>MreatedOn</li>
 	<li>ModifiedOn</li>
 	<li>ModifiedBy</li>
</ul>

```csharp
    public class Car : IEntity
    {
        public Guid Id { get; set; }

        public string RegistrationNumber { get; set; }
        public string Brand { get; set; }
        public string Model { get; set; }
        public Guid OwnerId { get; set; }
        public DateTime CreatedOn { get; set; }
        public Guid? CreatedBy { get; set; }
        public DateTime ModifiedOn { get; set; }
        public Guid? ModifiedBy { get; set; }
    }
```

Now I would like to return list of cars from my ASP .NET Core Web API. I do not want to include above four properties. I just want to return Id, registration number, brand, model and ownerId. Here we will use Linq.Expression to transform Car object to the CarDTO which will be returned from the API. "CarDTO" class looks like below:

```csharp
    public class CarDTO
    {
        public Guid Id { get; set; }
        public string RegistrationNumber { get; set; }
        public string Brand { get; set; }
        public string Model { get; set; }
        public Guid OwnerId { get; set; }
    }
```

Lets create mapper and use Linq.Expression. This mapper will be responsible for mapping "Car" instance to "CarDTO" instance:

```csharp
    public interface ICarsMapper
    {
        Expression<Func<Car, CarDTO>> MapToDTO { get; }
    }

    public class CarsMapper : ICarsMapper
    {
        public Expression<Func<Car, CarDTO>> MapToDTO { get; }

        public CarsMapper()
        {
            MapToDTO = c => new CarDTO
            {
                Id = c.Id,
                Brand = c.Brand,
                Model = c.Model,
                OwnerId = c.OwnerId,
                RegistrationNumber = c.RegistrationNumber
            };
        }
    }
```

As you can see we have Expression with Delegate which has "Car"instance as parameter and "CarDTO" as result. In the constructor of the mapper we are doing magic transformation.

Now lets get back to the Cars repository. To make it easier I will add one more method into it called "GetCarDTO" at the end of the class:

```csharp
        public async Task<CarDTO> GetCarDTO(Guid id)
        {
            var carDTO = await _applicationDbContext.Cars
              .Where(x => x.Id == id)
              .Select(_carsMapper.MapToDTO)
              .SingleOrDefaultAsync();

            return carDTO;
        }
```

We used "CarsMapper" to transform "Car" instance to the "CarDTO" instance in the single query - find car with specific id, then transform it to the "CarDTO" instance and return it.
This is just a small sample to present how Linq.Expressions can be used. This topic is enormous so I recommend you to read more about it.

&nbsp;
<h3><strong>Changes tracking</strong></h3>
It is possible to track changes when we are working on the data models in our code. For instance we can change phone number of the car owner. We can easily track changes and get all modified entities with "ChangeTracker" provided by Entity Framework Core. Please look at the below method located in the "ApplicationDbContext" class:

```csharp
        public override Task<int> SaveChangesAsync(bool acceptAllChangesOnSuccess, CancellationToken cancellationToken = default(CancellationToken))
        {
            var addedAuditedEntities = ChangeTracker.Entries<IEntity>()
                .Where(p => p.State == EntityState.Added)
                .Select(p => p.Entity);

            var modifiedAuditedEntities = ChangeTracker.Entries<IEntity>()
                .Where(p => p.State == EntityState.Modified)
                .Select(p => p.Entity);

            return base.SaveChangesAsync(acceptAllChangesOnSuccess, cancellationToken);
        }
```

As you can see you we have easy access to all added and edited entities. Important note here! No changes are send to the database until we invoke "SaveChangesAsync" method.

&nbsp;
<h3><strong>Automatic migrations</strong></h3>
As mentioned in the previous article we know that we should apply migration and update database to start using Entity Framework and queries. But its good to enable automatic migrations so once our ASP .NET Core Web API application is started it will apply all migrations if needed.

To enable automatic migrations add below code in the "Startup" class in the "ConfigureServices" method:

```csharp
services.BuildServiceProvider()
    .GetService<ApplicationDbContext>()
    .Database
    .Migrate();
```

Now once you start you ASP .NET Core app Entity Framework will apply all migrations if needed.

&nbsp;
<h3><strong>Fluent API vs Data Annotations</strong></h3>
<strong>Fluent API
</strong>

The term Fluent API refers to a pattern of programming where method calls are chained together with the end result. You can read more about FluentInterface on <a href="http://Martin Flower's blog" target="_blank" rel="noopener">https://martinfowler.com/bliki/FluentInterface.html</a>.

General idea is to write code like below using Fluent API to configure the table that a type maps to:

```csharp
   modelBuilder.Entity<Car>()
       .HasOne<Owner>()
       .WithMany()
       .HasForeignKey(a => a.OwnerId);
```

"DbContext" class has a method called "OnModelCreating" that takes an instance of "ModelBuilder" as a parameter. In our example "OnModelCreating" method:

```csharp
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Car>()
                .HasOne<Owner>()
                .WithMany()
                .HasForeignKey(a => a.OwnerId);

            modelBuilder.Entity<Car>()
                .Property(b => b.RegistrationNumber)
                .HasMaxLength(7);
        }
```

We can setup relation between Cars and Onwers. Car has one owner (with unique ID) which can have many cars.

<strong>Data Annotations</strong>

We do not have to use Fluent API - there is alternative called Data Annotations. Then constraints from above code will look like below:

```csharp
    public class Car : IEntity
    {
        public Guid Id { get; set; }
        [MaxLength(7)]
        public string RegistrationNumber { get; set; }
        public string Brand { get; set; }
        public string Model { get; set; }
        [ForeignKey("Owner")]
        public Guid OwnerId { get; set; }
        public Owner Owner { get; set; }
    }
```

&nbsp;

<strong>Data Annotations versus Fluent API</strong>

Both approaches are acceptable. Data Annotations is more intrusive way because you are contaminating your model with data annotations related to the infrastructure database.
With Fluent API model stays cleaner and we manage most conventions and mappings within one place. Final decision always depends on the project and architecture.

&nbsp;
<h3><strong>IEntityTypeConfiguration</strong></h3>
Once we described difference between Fluent API and Data Annotatons we can move to the part related with "IEntityTypeConfiguration" interface. We mentioned that "DbContext" class has a method called "OnModelCreating" where we can setup relationships and conditions for different types. Entity Framework provides "IEntityTypeConfiguration" interface so we can create separate class which implements it and inside this class we can set all the constraints.

Lets look at below example to make it easier to understand:

```csharp
    public class CarEntityConfiguration : IEntityTypeConfiguration<Car>
    {
        public void Configure(EntityTypeBuilder<Car> builder)
        {
            builder
                .HasOne<Owner>()
                .WithMany()
                .HasForeignKey(a => a.OwnerId);

            builder
               .Property(b => b.RegistrationNumber)
               .HasMaxLength(7);
        }
    }
```

"CarEntityConfiguration" class has one method implemented from the "IEntityTypeConfiguration" interface called "Configure". Inside this method we can setup all the constrains which we added earlier in the "OnModelCreating" method inside "ApplicationDbContext" class. Now we can replace the code inside this method with one single line:

```csharp
 protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            //modelBuilder.Entity<Car>()
            //    .HasOne<Owner>()
            //    .WithMany()
            //    .HasForeignKey(a => a.OwnerId);

            //modelBuilder.Entity<Car>()
            //    .Property(b => b.RegistrationNumber)
            //    .HasMaxLength(7);

            modelBuilder.ApplyConfiguration(new CarEntityConfiguration());
        }
```

&nbsp;
<h3><strong>Wrapping up</strong></h3>
In this second article (previous one you can find <a href="https://devislandblog.wordpress.com/2018/09/01/entity-framework-core-with-asp-net-core-web-api-jump-start/" target="_blank" rel="noopener">here</a>) I presented some hints related with Entity Framework. Creating queries, tracking changes or using Linq.Expressions are some of them. I recommend you to download second sample I prepared available on <a href="https://github.com/Daniel-Krzyczkowski/NetCsharp/tree/master/EntityFrameworkCoreHints" target="_blank" rel="noopener">GitHub</a> to experiment and practise. I hope this article will be helpful!