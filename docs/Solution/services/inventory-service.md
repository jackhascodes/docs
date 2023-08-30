# Inventory SERVICE

[[_TOC_]]

## A Note on Subscription Services
Subscription services such as PPC will be treated as their own set of inventory. Libraries who subscribe to these
services will search them using the same technique as a parent in a consortium.

Therefore, it will be helpful to think of inventories more as collections of titles available to a given entity. The
majority of these entities will be libraries, but may be other things such as subcription providers, or supra-library
entities. The term LibraryID is currently being used as an alias for all such entities, but this may change at some 
point.

Libraries are currently the only entity type capable of checking out titles, but for maximum flexibility, it would be
helpful to begin thinking of libraries which check out items from an inventory as clients of that inventory. (See also
[Inventory Access Service](inventory-access-service.md)

## Domain

(see [Domain Overview](inventory-service/domain.md))

The Inventory Service is intended to condense availability data for use in other services (initially, and primarily,
[Search ETL](search-etl.md)).

Inventory will take availability updates and minimize the representation of an available inventory item to a trinary
integer, where:

* -1: Scoped
* 0: Unavailable
* 1: Available

Other library-specific details regarding an inventory item include:

* Audience Code:

  This value is supplied at the title level, but may be overridden by a library who may believe a more suitable audience
  code applies

* Date Added:

  This value represents the date that the library took ownership of the title.

## Endpoints

(see: [Endpoints](inventory-service/endpoints.md))

Access to Inventory use cases is primarily event based, as it is intended to primarily be a transformer of data.
However, because of the ["notice event" pattern](../events/patterns.md) used by the majority of publishers, there must
be at least a GET endpoint.

Therefore, Inventory Service has Event endpoints and REST endpoints.

## Data Providers

Inventory Service will maintain its own persistence repository. To enable efficient and extensible storage of
things like BTKey aliasing of products, this should be a relational database.

## Considerations

Given the design for this and the title etl is serving more of a purpose than simple ETLs (see Get usecases)
this should be renamed inventory-service.

## Architectural Overview
### Structure
<pre>
.
├── Configuration
|   ├── App.cs
|   └── Configuration
|       ├── DatabaseConfiguration.cs
|       ├── EndpointConfiguration.cs
|       ├── EventBusConfiguration.cs
|       └── UseCaseConfiguration.cs
├── <a href="inventory-service/data-providers.md">DataProviders</a>
|   ├── <a href="inventory-service/data-providers.md#repository">Database</a>
|   |   ├── Entities
|   |   |   ├── Inventory.cs
|   |   |   ├── BTAlias.cs
|   |   ├── Database.cs
|   └── <a href="inventory-service/data-providers.md#eventbus">EventBus</a>
|       └── EventBus.cs
├── <a href="inventory-service/domain.md">Domain</a>
|   ├── <a href="inventory-service/domain.md#entities">Entities</a>
|   |   ├── Inventory.cs
|   |   └── InventoryShort.cs
|   └── <a href="inventory-service/domain.md#use-cases">UseCases</a>
|       ├── <a href="inventory-service/domain.md#delete">Delete</a>
|       |   ├── IRepo.cs
|       |   ├── IEventBus.cs
|       |   ├── Request.cs
|       |   ├── Response.cs
|       |   └── UseCase
|       ├── <a href="inventory-service/domain.md#get">Get</a>
|       |   ├── IRepo.cs
|       |   ├── Request.cs
|       |   ├── Response.cs
|       |   └── UseCase
|       ├── <a href="inventory-service/domain.md#getbyproductid">GetByProductID</a>
|       |   ├── IRepo.cs
|       |   ├── Request.cs
|       |   ├── Response.cs
|       |   └── UseCase
|       ├── <a href="inventory-service/domain.md#increment">Increment</a>
|       |   ├── IRepo.cs
|       |   ├── IEventBus.cs
|       |   ├── Request.cs
|       |   ├── Response.cs
|       |   └── UseCase
|       └── <a href="inventory-service/domain.md#upsert">Upsert</a>
|           ├── IRepo.cs
|           ├── IEventBus.cs
|           ├── Request.cs
|           ├── Response.cs
|           └── UseCase
├── <a href="inventory-service/endpoints.md">Endpoints</a>
    ├── Http
    |   ├── Get
    |   |   ├── Endpoint.cs
    |   |   └── ResponseDTO.cs
    |   └── GetByProductID
    |   |   ├── Endpoint.cs
    |   |   └── ResponseDTO.cs
    └── Subscribers
        ├── Delete
        |   ├── Event.cs
        |   └── SubscribeHandler.cs
        ├── Increment
        |   ├── Event.cs
        |   └── SubscribeHandler.cs
        └── Upsert
            ├── Event.cs
            └── SubscribeHandler.cs

</pre>

### Dependency and Data Flow
![Inventory SERVICE Diagram](./.diagrams/inventory-service-overview-simple.png)

