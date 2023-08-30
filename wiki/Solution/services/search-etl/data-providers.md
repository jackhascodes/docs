# Search ETL: Data Providers

[[_TOC_]]

Search ETL will persist data in CosmosDB, which will be set up as a datasource for Cognitive Search.

## Repository Schema

### Inventory

| Field                | Type                      | Notes                                                                                                                                            |
|----------------------|---------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| ProductID            | int                       |                                                                                                                                                  |
| Annotation           | string                    |                                                                                                                                                  |
| AudienceDesc         | string                    |                                                                                                                                                  |
| AudioSampleAvailable | string                    |                                                                                                                                                  |
| Author               | string                    |                                                                                                                                                  |
| AvgRating            | int                       |                                                                                                                                                  |
| AxisAttribute        | string                    |                                                                                                                                                  |
| BISAC                | string[]                  |                                                                                                                                                  |
| DeletedAt            | int                       | a yyyymmdd formatted date as an integer. 0 == not deleted.                                                                                       |
| EFormatLiteral       | string                    |                                                                                                                                                  |
| ESupplierLiteral     | string                    |                                                                                                                                                  |
| Edition              | string                    |                                                                                                                                                  |
| Genre                | string                    |                                                                                                                                                  |
| ImprintDesc          | string                    |                                                                                                                                                  |
| Inventory            | [Inventory](#inventory)[] | An array of Inventory items                                                                                                                      |
| JacketImageInd       | bool                      |                                                                                                                                                  |
| LanguageDesc         | string                    |                                                                                                                                                  |
| Lexile               | string                    |                                                                                                                                                  |
| Narrator             | string                    |                                                                                                                                                  |
| PageCount            | int                       |                                                                                                                                                  |
| PopularityScore      | int                       |                                                                                                                                                  |
| PubDate              | int                       | a yyyymmdd formatted date as an integer                                                                                                          |
| PurchaseOption       | string                    |                                                                                                                                                  |
| ReadyToVend          | bool                      |                                                                                                                                                  |
| RunTime              | int                       |                                                                                                                                                  |
| Series               | string                    |                                                                                                                                                  |
| ShortTitle           | string                    |                                                                                                                                                  |
| SubTitle             | string                    |                                                                                                                                                  |
| SupplierDesc         | string                    |                                                                                                                                                  |
| UpdatedAt            | int                       | a yyyymmdd formatted date as an integer for the date that triggered an update to this document                                                   |
| _ts                  | timestamp                 | a timestamp of when this document was updated (see [Cosmos Indexing](https://learn.microsoft.com/en-us/azure/search/search-howto-index-cosmosdb) |

### Inventory

| Field        | Type   | Notes |
|--------------|--------|-------|
| InventoryID  | INT    |       |
| AudienceCode | string |       |
| Available    | bool   |       |

## Required Methods

### Save(doc)

This should be a whole document upsert.