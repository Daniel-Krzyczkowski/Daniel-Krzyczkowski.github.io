---
title: "Event Sourcing with Azure SQL and Entity Framework Core"
excerpt: "This article presents how to implement event sourcing with Azure SQL database and Entity Framework Core"
header:
  image: /images/devisland/article38/assets/EventSourcingWithEfCoreAndAzureSql1.png
---

<p align="center">
<img src="/images/devisland/article38/assets/EventSourcingWithEfCoreAndAzureSql1.png?raw=true" alt="Event Sourcing with Azure SQL and Entity Framework Core"/>
</p>

# Introduction

Recently I had a chance to read many different articles related to Event Sourcing. This topic is becoming more and more popular especially when talking about microservices. In this article, I would like to present how to add Event Sourcing implementation but before that, just one short summary.

Event Sourcing is just the way to persist state of the entities. There is a lot of confusion nowadays especially when talking about microservices and CQRS. Just to clarify - Event Sourcing has nothing to do with microservices. We can use Event Sourcing in the monolithic systems too. CQRS is not a part of Event Sourcing. Event Sourcing stores each state mutation as a separate record called an event in opposite to state-oriented persistence that only keeps the latest version of the entity state.


# Solution architecture

To demonstrate how to use Azure SQL database together with Entity Framework Core ORM to implement Event Sourcing I decided to start the development of a sample solution called Cars Island. Source code is available on my GitHub.. This project is based on the eShop on containers solution which I found a bit heavy so that is why I decided to create my solution. In the Cars Island solution there are four microservices but we will focus on the *Catalog* microservice and event related to changing car price per day for specific car from the catalog:

<p align="center">
<img src="/images/devisland/article38/assets/EventSourcingWithEfCoreAndAzureSql2.png?raw=true" alt="Image not found"/>
</p>

## Event Log implementation

In this section we will discuss implementation of th class responsible for storing events information. This class implements from the *ICatalogIntegrationEventService* interface:

```csharp
    public interface ICatalogIntegrationEventService
    {
        Task PublishEventsThroughEventBusAsync(IntegrationEvent @event);
        Task AddAndSaveEventAsync(IntegrationEvent @event);
    }
```

If you did not have a chance to my previous article [Asynchronous messaging with Azure Service Bus](https://daniel-krzyczkowski.github.io/Asynchronous-Messaging-With-Azure-Service-Bus/) I encourage you to do it. There we used the above interface to send messages across microservices once there is a new event. In this article, we will focus on the *AddAndSaveEventAsync* method.

We will talk about events that occur in the *Catalog* microservice. In this case *CarPricePerDayChangedIntegrationEvent* which we will discuss in detail later in this article. Below we can see *CatalogIntegrationEventService* class that implements ** interface:

```csharp
    public class CatalogIntegrationEventService : ICatalogIntegrationEventService
    {
        private readonly CarCatalogDbContext _carCatalogDbContext;
        private readonly IEventBus _eventBus;
        private readonly IEventLogService _eventLogService;
        private readonly ILogger<CatalogIntegrationEventService> _logger;
        private readonly Func<DbConnection, IEventLogService> _integrationEventLogServiceFactory;

        public CatalogIntegrationEventService(CarCatalogDbContext carCatalogDbContext, Func<DbConnection, IEventLogService> integrationEventLogServiceFactory,
                                     IEventBus eventBus,
                                     ILogger<CatalogIntegrationEventService> logger)
        {
            _carCatalogDbContext = carCatalogDbContext ?? throw new ArgumentNullException(nameof(carCatalogDbContext));
            _integrationEventLogServiceFactory = integrationEventLogServiceFactory ?? throw new ArgumentNullException(nameof(integrationEventLogServiceFactory));
            _eventBus = eventBus ?? throw new ArgumentNullException(nameof(eventBus));
            _eventLogService = _integrationEventLogServiceFactory(_carCatalogDbContext.Database.GetDbConnection());
            _logger = logger ?? throw new ArgumentNullException(nameof(logger));
        }

        public async Task AddAndSaveEventAsync(IntegrationEvent @event)
        {
            await ResilientTransaction.CreateNew(_carCatalogDbContext).ExecuteAsync(async () =>
            {
                await _carCatalogDbContext.SaveChangesAsync();
                await _eventLogService.SaveEventAsync(@event, _carCatalogDbContext.Database.CurrentTransaction);
            });
        }

        public async Task PublishEventsThroughEventBusAsync(IntegrationEvent @event)
        {
            try
            {
                await _eventLogService.MarkEventAsInProgressAsync(@event.Id);
                await _eventBus.PublishAsync(@event);
                await _eventLogService.MarkEventAsPublishedAsync(@event.Id);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "ERROR publishing integration event: '{IntegrationEventId}'", @event.Id);

                await _eventLogService.MarkEventAsFailedAsync(@event.Id);
            }
        }
    }
```

Let's focus on the *AddAndSaveEventAsync* method:

```csharp
        public async Task AddAndSaveEventAsync(IntegrationEvent @event)
        {
            await ResilientTransaction.CreateNew(_carCatalogDbContext).ExecuteAsync(async () =>
            {
                await _carCatalogDbContext.SaveChangesAsync();
                await _eventLogService.SaveEventAsync(@event, _carCatalogDbContext.Database.CurrentTransaction);
            });
        }
```

As we can see there are two operations within the transaction:

1. We want to save changes in the catalog database - *await _carCatalogDbContext.SaveChangesAsync()*
2. We want to save information about the event that occurred and was handled in our microservice - *await _eventLogService.SaveEventAsync(@event, _carCatalogDbContext.Database.CurrentTransaction)*

You probably notice that we are using Entity Framework Core transaction here and code responsible for this is located in the *ResilientTransaction* class:

```csharp
    public class ResilientTransaction
    {
        private readonly DbContext _context;
        private ResilientTransaction(DbContext context) =>
            _context = context ?? throw new ArgumentNullException(nameof(context));

        public static ResilientTransaction CreateNew(DbContext context) =>
            new ResilientTransaction(context);

        public async Task ExecuteAsync(Func<Task> action)
        {
            var strategy = _context.Database.CreateExecutionStrategy();
            await strategy.ExecuteAsync(async () =>
            {
                using (var transaction = _context.Database.BeginTransaction())
                {
                    await action();
                    transaction.Commit();
                }
            });
        }
    }
```

In the *CreateNew* static method we are passing *DbContext* parameter so we can use it later to begin the transaction. We can pass the operation that has to be executed using *action* parameter in the *ExecuteAsync* method. I encourage you to visit my [Cars Island project on GitHub](https://github.com/Daniel-Krzyczkowski/MicrosoftAzure/tree/master/asp-net-core-microservices-with-azure-and-docker) to see implementation details.


## CarCatalogDbContext and EventLogContext

The question is how events are stored and where? Just a short reminded - each microservice in our sample solution has its own database. In our case, we are talking about *Catalog* microservice in this specific example. There is *catalog-db* database and inside this database, there are two tables:

1. Cars
2. IntegrationEventLog

<p align="center">
<img src="/images/devisland/article38/assets/EventSourcingWithEfCoreAndAzureSql3.PNG?raw=true" alt="Image not found"/>
</p>

In the source code there are two separate *DbContext* instances used, one is *CarCatalogDbContext* used to manage cars data and second, for events is *EventLogContext*:

```csharp
    public class CarCatalogDbContext : DbContext
    {
        public CarCatalogDbContext(DbContextOptions<CarCatalogDbContext> options)
                                                              : base(options)
        {
        }

        public DbSet<Car> Cars { get; set; }


        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);

            modelBuilder.Entity<Car>().HasData(
                    new Car
                    {
                        Id = Guid.NewGuid(),
                        Brand = "BMW",
                        Model = "320",
                        AvailableForRent = true,
                        PricePerDay = 200
                    },
                    new Car
                    {
                        Id = Guid.NewGuid(),
                        Brand = "Audi",
                        Model = "A1",
                        AvailableForRent = true,
                        PricePerDay = 120
                    },
                    new Car
                    {
                        Id = Guid.NewGuid(),
                        Brand = "Mercedes",
                        Model = "E200",
                        AvailableForRent = true,
                        PricePerDay = 250
                    },
                    new Car
                    {
                        Id = Guid.NewGuid(),
                        Brand = "Ford",
                        Model = "Focus",
                        AvailableForRent = true,
                        PricePerDay = 90
                    }
                );
        }
    }
```

```csharp
    public class EventLogContext : DbContext
    {
        public EventLogContext(DbContextOptions<EventLogContext> options) : base(options)
        {
        }

        public DbSet<IntegrationEventLogEntry> IntegrationEventLogs { get; set; }

        protected override void OnModelCreating(ModelBuilder builder)
        {
            builder.Entity<IntegrationEventLogEntry>(ConfigureIntegrationEventLogEntry);
        }

        private void ConfigureIntegrationEventLogEntry(EntityTypeBuilder<IntegrationEventLogEntry> builder)
        {
            builder.ToTable("IntegrationEventLog");

            builder.HasKey(e => e.EventId);

            builder.Property(e => e.EventId)
                .IsRequired();

            builder.Property(e => e.Content)
                .IsRequired();

            builder.Property(e => e.CreationTime)
                .IsRequired();

            builder.Property(e => e.State)
                .IsRequired();

            builder.Property(e => e.TimesSent)
                .IsRequired();

            builder.Property(e => e.EventTypeName)
                .IsRequired();

        }
    }
```

Now below we can move forward and see how car price per day change event is handled.


## Car price per day changed event persistence

Now once we know that *CatalogIntegrationEventService* class is responsible for handling events and storing them in the database in a separate table, it is good to show some examples. In this case we will focus on the *CarPricePerDayChangedIntegrationEvent* event.

This is the cars table in the database with existing cars:

<p align="center">
<img src="/images/devisland/article38/assets/EventSourcingWithEfCoreAndAzureSql4.PNG?raw=true" alt="Image not found"/>
</p>


Let's open *CatlogController* located in the *Catalog* microservice:

```csharp
        public async Task<ActionResult> UpdateCarAsync([FromBody] Car carToUpdate)
        {
            var existingCarFromTheCatalog = await _carCatalogDbContext.Cars.SingleOrDefaultAsync(i => i.Id == carToUpdate.Id);

            if (existingCarFromTheCatalog == null)
            {
                return NotFound(new { Message = $"Car with id {carToUpdate.Id} not found." });
            }

            else
            {
                var oldPricePerDay = existingCarFromTheCatalog.PricePerDay;
                var hasPricePerDayChanged = existingCarFromTheCatalog.PricePerDay != carToUpdate.PricePerDay;
                existingCarFromTheCatalog.PricePerDay = carToUpdate.PricePerDay;

                _carCatalogDbContext.Cars.Update(existingCarFromTheCatalog);

                if (hasPricePerDayChanged)
                {
                    var pricePerDayChangedEvent = new CarPricePerDayChangedIntegrationEvent(existingCarFromTheCatalog.Id,
                                                                                            existingCarFromTheCatalog.PricePerDay,
                                                                                            oldPricePerDay);

                    await _catalogIntegrationEventService.AddAndSaveEventAsync(pricePerDayChangedEvent);
                    await _catalogIntegrationEventService.PublishEventsThroughEventBusAsync(pricePerDayChangedEvent);
                }

                else
                {
                    await _carCatalogDbContext.SaveChangesAsync();
                }
                return NoContent();
            }
        }
```

As we can see this is the method responsible for handling updates related to cars located in the catalog. In this simple scenario, we would like to change the car price per day for the specific car and save this change in the event store. We can see above that if the price per day has changed, its value is updated using *_carCatalogDbContext.Cars.Update(existingCarFromTheCatalog)* code. Short reminder - this will not save changes to the database yet. There is only information for the Entity Framework Core Change Tracker that there was an update to the entity.

Then we are creating *CarPricePerDayChangedIntegrationEvent* instance with information about car identity, the old price per day, the new price per day, and some additional properties. This is the class implementation:

```csharp
    public class CarPricePerDayChangedIntegrationEvent : IntegrationEvent
    {
        public Guid CarId { get; private set; }

        public decimal NewPricePerDay { get; private set; }

        public decimal OldPricePerDay { get; private set; }

        public CarPricePerDayChangedIntegrationEvent(Guid carId, decimal newPricePerDay, decimal oldPricePerDay)
        {
            CarId = carId;
            NewPricePerDay = newPricePerDay;
            OldPricePerDay = oldPricePerDay;
        }
    }
```

Then we are ready to call integration service using *await _catalogIntegrationEventService.AddAndSaveEventAsync(pricePerDayChangedEvent)* code. Once data is saved in the database we can look at it using data explorer:

<p align="center">
<img src="/images/devisland/article38/assets/EventSourcingWithEfCoreAndAzureSql5.PNG?raw=true" alt="Image not found"/>
</p>

Here is the value stored under the *Content* column:

```csharp
{"CarId":"2605c956-1ab4-4ce9-ad90-559978d754dd","NewPricePerDay":300.0,"OldPricePerDay":200.00,"Id":"9ef1f6ec-ef2e-4ad2-9a15-15284fc94ca7","CreationDate":"2020-07-19T11:22:18.9525308Z"}
```

As we can see there is information about car identity, old and new price per day, the identity of the event together with the creation date. We can also see that there are other columns like *EventTypeName* or *TimesSend*. We can of course adjust the columns according to our needs and requirements.


# Summary

In this article, I presented how to implement Event Sourcing using the Azure SQL database and Entity Framework Core ORM. As we saw Event Sourcing is not so scary. Of course, it depends how complex our solution is but it is good to understand the basics - Event Sourcing is just the way to persist state of the entities. I encourage you to check the source code for the whole solution on [my GitHub.](https://github.com/Daniel-Krzyczkowski/MicrosoftAzure/tree/master/asp-net-core-microservices-with-azure-and-docker)
