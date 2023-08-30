# Search ETL: Domain Overview

[[_TOC_]]

## Search ETL Use Cases

### Conventions:

* Dates will be represented as yyyymmdd integers (e.g. Jan 3 2020 would be 20200103).
* All requests should have:
    * Correlation ID: string - a unique identifier used to trace a request across services.
    * Origin: string - an identifier for the source of the request.
    * Timestamp: int - a unix timestamp for when the request was *made*.
* All responses should forward the Correlation ID supplied in the request
* All responses should set a new origin (i.e. `title-etl`)
* All responses should set a new timestamp (i.e. `now()`)
* Use cases with Publish Topics should publish:
    * `foo.bar.success` when a valid response is generated.
    * `foo.bar.fail` when an exception occurs.

  to the event bus.

---

## Update Inventory

When an inventory item is updated, we only need to update the title document in the region(s)
defined by the inventory item.

* Upsert [Inventory](#inventory)
* Get [Regions](#region) by `LibraryKey`
* For each region [Rebuild Title for Region](#rebuild-title-for-region) by `ProductID`

**Actors:** inventory update events

**Ports:**
*Search Repo:*

* Get(int regionID, int productID)
* Upsert(TitleDocument doc)

*Inventory Repo:*

* Get(int productID, int libraryID)

*Request:*

| Field        | Type   | Notes                                                                                             |
|--------------|--------|---------------------------------------------------------------------------------------------------|
| ProductID    | int    |                                                                                                   |
| LibraryKey   | int    |                                                                                                   |
| AudienceCode | string |                                                                                                   |
| DateAdded    | int    | a yyyymmdd formatted date as an integer                                                           |
| Available    | int    | one of [-1/0/1] where -1 is scoped, 0 is unavailable, and 1 is available, regardless of quantity. |
| DeletedDate  | int    | a yyyymmdd formatted date as an integer. 0 == not deleted.                                        |

*Response:*

| Field      | Type | Notes |
|------------|------|-------|
| ProductID  | int  |       |
| LibraryKey | int  |       |

---

## Update Title

When a title is updated, we must rebuild the title document in ALL regions.

* Get Title by ProductID
* Get all inventory (ProductID, LibraryKey, AudienceCode, DateAdded, Available) where:
    * ProductID == Title.ProductID &&
    * Region == Inventory.Region &&
    * DeletedDate == 0
* Set Title.Inventory to result
* Set Title._ts to "now"
* Update Title

**Actors:** inventory update events

**Ports:**
*Search Repo:*

* Get(int regionID, int productID)
* Upsert(TitleDocument doc)

*Inventory Repo:*

* Get(int productID)

*Request:*

| Field                | Type     | Notes                                                      |
|----------------------|----------|------------------------------------------------------------|
| ProductID            | int      |                                                            |
| Annotation           | string   |                                                            |
| AudienceDesc         | string   |                                                            |
| AudioSampleAvailable | string   |                                                            |
| Author               | string   |                                                            |
| AvgRating            | int      |                                                            |
| AxisAttribute        | string   |                                                            |
| BISAC                | string[] |                                                            |
| DeletedAt            | int      | a yyyymmdd formatted date as an integer. 0 == not deleted. |
| EFormatLiteral       | string   |                                                            |
| ESupplierLiteral     | string   |                                                            |
| Edition              | string   |                                                            |
| Genre                | string   |                                                            |
| ImprintDesc          | string   |                                                            |
| JacketImageInd       | bool     |                                                            |
| LanguageDesc         | string   |                                                            |
| Lexile               | string   |                                                            |
| Narrator             | string   |                                                            |
| PageCount            | int      |                                                            |
| PopularityScore      | int      |                                                            |
| PubDate              | int      | a yyyymmdd formatted date as an integer                    |
| PurchaseOption       | string   |                                                            |
| ReadyToVend          | bool     |                                                            |
| RunTime              | int      |                                                            |
| Series               | string   |                                                            |
| ShortTitle           | string   |                                                            |
| SubTitle             | string   |                                                            |
| SupplierDesc         | string   |                                                            |
| UpdatedAt            | int      | a yyyymmdd formatted date as an integer                    |

*Response:*

| Field      | Type | Notes |
|------------|------|-------|
| ProductID  | int  |       |
| LibraryKey | int  |       |

## Entities

### Title

| Field                | Type        | Notes                                                              |
|----------------------|-------------|--------------------------------------------------------------------|
| _ts                  | datetime    | timestamp used by ACS to determine what to pull. Defaults to "now" |
| ProductID            | int         |                                                                    |
| Series               | string      |                                                                    |
| Author               | string      |                                                                    |
| Narrator             | string      |                                                                    |
| SupplierDesc         | string      |                                                                    |
| ImprintDesc          | string      |                                                                    |
| ESupplierLiteral     | string      |                                                                    |
| EFormatLiteral       | string[]    |                                                                    |
| BTKeys               | string[]    |                                                                    |
| PubDate              | int         | a yyyymmdd formatted date as an integer                            |
| LanguageDesc         | string      |                                                                    |
| BISAC                | stringp[]   |
| Genre                | string      |                                                                    |
| AudienceDesc         | string      |                                                                    |
| UpdateDate           | int         | a yyyymmdd formatted date as an integer                            |
| Annotation           | string      |                                                                    |
| AvgRating            | int         |                                                                    |
| ReadyToVend          | bool        |                                                                    |
| PurchaseOption       | string      |                                                                    |
| Edition              | string      |                                                                    |
| JacketImageInd       | bool        |                                                                    |
| SubTitle             | string      |                                                                    |
| ShortTitle           | string      |                                                                    |
| AudioSampleAvailable | string      |                                                                    |
| AxisAttribute        | string      |                                                                    |
| RunTime              | int         |                                                                    |
| PageCount            | int         |                                                                    |
| Lexile               | string      |                                                                    |
| PopularityScore      | int         |                                                                    |
| DeletedDate          | int         | a yyyymmdd formatted date as an integer. 0 == not deleted.         |
| Inventory            | Inventory[] |                                                                    |

### Inventory

| Field        | Type   | Notes                                                                                             |
|--------------|--------|---------------------------------------------------------------------------------------------------|
| LibraryKey   | int    |                                                                                                   |
| AudienceCode | string |                                                                                                   |
| DateAdded    | int    | a yyyymmdd formatted date as an integer                                                           |
| Available    | int    | one of [-1/0/1] where -1 is scoped, 0 is unavailable, and 1 is available, regardless of quantity. |

### Region

| Field      | Type | Notes |
|------------|------|-------|
| RegionKey  | int  |       |
| LibraryKey | int  |       |
