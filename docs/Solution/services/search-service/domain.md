# Search Service

## Use Cases

### Search
Call out to inventory access service to get inventory IDs for clientID, then add these IDs to the inventoryIDs field of
the search request.

On response, call out to inventory service for most available inventory items for given inventories and product keys in
resultset.

**Publish Topic:** DO NOT PUBLISH

**Actors:**

* VendorAPI, Axis360/Boundless Web

**Ports:**

*InventoryService*
* GetInventoryAccess(string ClientRef): string[]
* GetIDsByBTKey(string[] BTKeys): string[]
* GetInventoryItems(string[] ProductIDs, string[] InventoryIDs)

*SearchEngine*
* Search(string searchTerm, Filter[] filters, Facet[] facets, int page, int limit, OrderBy orderby, string[] inventories)

*Request:*

| name            | type                | notes                               |
|-----------------|---------------------|-------------------------------------|
| standard params |                     | *Correlation ID, Origin, Timestamp* |
| ClientID        | string              |                                     |
| search          | string              |                                     |
| filters         | [Filter](#filter)[] |                                     |
| facets          | [Facet](#facet)[]   |                                     |
| page            | int                 |                                     |
| limit           | int                 |                                     |
| orderBy         | [OrderBy](#orderby) |                                     |

*Response:*

## Entities

### SearchRequest

| Field        | Type    | Notes |
|--------------|---------|-------|
| inventoryIDs | int[]   |       |
| search       | string  |       |
| filter       | Filter  |       |
| facets       | Facet[] |       |
| page         | int     |       |
| limit        | int     |       |
| orderBy      | OrderBy |       |

### SearchResult

| Field           | Type                      | Notes |
|-----------------|---------------------------|-------|
| Count           |                           |       |
| AvailableFacets | [Facet](#facet)[]         |       |
| ResultSet       | [ResultSet](#resultSet)[] |       |

### ResultSet

| Field                | Type     | Notes                                                                                          |
|----------------------|----------|------------------------------------------------------------------------------------------------|
| Score                | float    | document_score (if a text query was provided)                                                  |
| ProductID            | int      |                                                                                                |
| Annotation           | string   |                                                                                                |
| AudienceDesc         | string   |                                                                                                |
| AudioSampleAvailable | string   |                                                                                                |
| Author               | string   |                                                                                                |
| AvgRating            | int      |                                                                                                |
| AxisAttribute        | string   |                                                                                                |
| BISAC                | string[] |                                                                                                |
| DeletedAt            | int      | a yyyymmdd formatted date as an integer. 0 == not deleted.                                     |
| EFormatLiteral       | string[] |                                                                                              |
| BTKeys               | string[] |                                                                                              |
| ESupplierLiteral     | string   |                                                                                                |
| Edition              | string   |                                                                                                |
| Genre                | string   |                                                                                                |
| ImprintDesc          | string   |                                                                                                |
| JacketImageInd       | bool     |                                                                                                |
| LanguageDesc         | string   |                                                                                                |
| Lexile               | string   |                                                                                                |
| Narrator             | string   |                                                                                                |
| PageCount            | int      |                                                                                                |
| PopularityScore      | int      |                                                                                                |
| PubDate              | int      | a yyyymmdd formatted date as an integer                                                        |
| PurchaseOption       | string   |                                                                                                |
| ReadyToVend          | bool     |                                                                                                |
| RunTime              | int      |                                                                                                |
| Series               | string   |                                                                                                |
| ShortTitle           | string   |                                                                                                |
| SubTitle             | string   |                                                                                                |
| SupplierDesc         | string   |                                                                                                |
| UpdatedAt            | int      | a yyyymmdd formatted date as an integer for the date that triggered an update to this document |

### Filter

| Field     | Type   | Notes |
|-----------|--------|-------|
| FieldName | string |       |
| Value     | string |       |
| Operator  | string |       |

### Facet

| Field     | Type     | Notes |
|-----------|----------|-------|
| FacetName | string   |       |
| Values    | string[] |       |

### OrderBy

| Field          | Type   | Notes |
|----------------|--------|-------|
| orderField     | string |       |
| orderDirection | string |       |
