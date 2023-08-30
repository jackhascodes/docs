# Search Service: Data Providers

## Azure Cognitive Search

(See also: [Azure Cognitive Search REST API](https://learn.microsoft.com/en-us/rest/api/searchservice/search-documents))

Azure Cognitive Search (ACS) is the search engine component. It is populated by [SearchETL](../search-etl.md).

Queries made to ACS will send at minimum:

* The search term
* fields to be returned (i.e. all fields except *Inventory*)
* The inventory criteria:

  `$filter=Inventory/any(i: search.in(i/Key, '${access-list}') and i/Available neq -1)`
* If searching only available, the inventory filters must use"

  `$filter=Inventory/any(i: search.in(i/Key, '${access-list}') and i/Available eq 1)`
* Further parameters such as facets, etc, may be sent.

### Schema

### Azure Cognitive Search Response

For a more complete explanation of the response from Cognitive Search, please
see [here](https://learn.microsoft.com/en-us/rest/api/searchservice/search-documents#response)

| Field                      | Type | Notes                                                                                                                       |
|----------------------------|------|-----------------------------------------------------------------------------------------------------------------------------|
| @odata.count               |      | (if $count=true was provided in the query)                                                                                  |
| @search.coverage           |      | (if minimumCoverage was provided in the query)                                                                              |
| @search.facets             |      | (if faceting was specified in the query)                                                                                    |
| facet_field                |      | [Facet](#facet)[]                                                                                                           |
| @search.nextPageParameters |      | [NextPageParams](#nextpageparams)                                                                                           |
| value                      |      | [SearchResult](#searchResult)[]                                                                                             |
| @odata.nextLink            |      | (URL to fetch the next page of results if not all results could be returned in this response; Applies  to both GET and POST |

#### SearchResult

| Field                | Type                      | Notes                                                                                                                                            |
|----------------------|---------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| @search.score        | float                     | document_score (if a text query was provided)                                                                                                    |
| @search.highlights   | n/a                       | can be ignored for this iteration of search                                                                                                      |
| @search.features     | n/a                       | can be ignored for this iteration of search, but holds metadata about why the record was returned                                                |
| key_field_name       | string                    | ProductID                                                                                                                                        |
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

### Facet

| Field | Type | Notes                                    |
|-------|------|------------------------------------------|
| value |      | facet_entry_value (for non-range facets) |
| from  |      | facet_entry_value (for range facets)     |
| to    |      | facet_entry_value (for range facets)     |
| count |      | number_of_documents                      |

### NextPageParams

(request body to fetch the next page of results if not all results could be returned in
this response and Search was called with POST)

| Field             | Type | Notes                                                    |
|-------------------|------|----------------------------------------------------------|
| count             |      | ... (value from request body if present)                 |
| facets            |      | ... (value from request body if present)                 |
| filter            |      | ... (value from request body if present)                 |
| highlight         |      | ... (value from request body if present)                 |
| highlightPreTag   |      | ... (value from request body if present)                 |
| highlightPostTag  |      | ... (value from request body if present)                 |
| minimumCoverage   |      | ... (value from request body if present)                 |
| orderby           |      | ... (value from request body if present)                 |
| scoringParameters |      | ... (value from request body if present)                 |
| scoringProfile    |      | ... (value from request body if present)                 |
| scoringStatistics |      | ... (value from request body if present)                 |
| search            |      | ... (value from request body if present)                 |
| searchFields      |      | ... (value from request body if present)                 |
| searchMode        |      | ... (value from request body if present)                 |
| select            |      | ... (value from request body if present)                 |
| "sessionId"       |      | ... (value from request body if present)                 |
| skip              |      | ... (page size plus value from request body if present)  |
| top               |      | ... (value from request body if present minus page size) |

## Inventory Access Service

(see [Inventory Access Service](../inventory-access-service.md))

Inventory Access Service will be used to retrieve the available inventory IDs that a library may search against.

## Inventory Service

(see [InventoryService](../inventory-service.md))

Inventory Service will be used to retrieve the most applicable availability for a title given its consortium
availability and subscriptions.
