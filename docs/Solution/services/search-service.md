
[[_TOC_]]

## Domain

(see [Domain Overview](search-service/domain.md))

The Search Service performs searches.

## Endpoints

(see: [Endpoints](search-etl/endpoints.md))

## Data Providers

(see: [Data Providers](search-etl/data-providers.md))
The Search Service combines a number of services in order to provide complete search results. It does not maintain its
own repository at this time.

## Architectural Overview
### Structure
<pre>
.
├── Configuration
|   ├── App.cs
|   └── Configuration
|       ├── DataProviderConfiguration.cs
|       ├── EndpointConfiguration.cs
|       ├── EventBusConfiguration.cs
|       └── UseCaseConfiguration.cs
├── <a href="search-service/data-providers.md">DataProviders</a>
|   ├── <a href="search-service/data-providers.md#repository">CognitiveSearch</a>
|   |   └── Database.cs
|   ├── <a href="search-service/data-providers.md#inventory-service">InventoryService</a>
|   |   └── Entities
|   |       └── Inventory.cs
├── <a href="search-service/domain.md">Domain</a>
|   ├── <a href="search-service/domain.md#entities">Entities</a>
|   |   └── SearchDocument.cs
|   └── <a href="search-service/domain.md#use-cases">UseCases</a>
|       └── <a href="search-service/domain.md#upsert">Search</a>
|           ├── ISearchEngine.cs
|           ├── IInventoryService.cs
|           ├── Request.cs
|           ├── Response.cs
|           └── UseCase
└── <a href="search-service/endpoints.md">Endpoints</a>
    └── HTTP
        └── Search
            ├── Request.cs
            └── Response.cs
</pre>



Data flow:

![how-a-search-flows.png](..%2F.diagrams%2Fhow-a-search-flows.png)
