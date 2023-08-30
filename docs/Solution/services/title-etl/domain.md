# Title ETL: Domain Overview

[[_TOC_]]

## Title ETL Use Cases

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

### Delete

**Publish Topic**: `title.delete.[success|fail]`

**Actors:** title delete events

**Ports:**

*Repo:*

* Delete(int ProductID): bool

*EventBus:*

* Publish(string Topic, data any)

*Request:*

| name            | type | notes                               |
|-----------------|------|-------------------------------------|
| standard params |      | *Correlation ID, Origin, Timestamp* |
| Product ID      | int  |                                     |

*Response:*

| field         | type   | notes            |
|---------------|--------|------------------|
| CorrelationID | string | from the request |
| Origin        | string | `"title_etl"`    |
| Timestamp     | int    | `now()`          |
| EventType     | string | `"notice"`       |
| Product ID    | int    |                  |

**Rules:**

* Actor MUST have permission
* Title MUST EXIST

**Except:**

* Not Authorized
* Ignored because old

---

### Get

**Publish Topic:** DO NOT PUBLISH

**Actors:**

* Other services
* Users

**Ports:**

*Repo:*

* Get(int ProductID): [Title](#title)

*Request:*

| name            | type | notes                               |
|-----------------|------|-------------------------------------|
| standard params |      | *Correlation ID, Origin, Timestamp* |
| Product ID      | int  |                                     |

*Response:*

| field         | type                      | notes            |
|---------------|---------------------------|------------------|
| CorrelationID | string                    | from the request |
| Origin        | string                    | `title_etl`      |
| Timestamp     | int                       | `now()`          |
| Title         | [TitleShort](#titleshort) |                  |

**Rules:**

* Actor MUST have permission
* Use Case MUST return a Title record if it exists.

**Except:**

* Not Authorized
* Not found

---

### Upsert

**Publish Topic:** `title.update.[success|fail]`

**Actors:**

* availability update events

*Repo:*

* Get(int ProductID): ?[Title](#title)
* Save([Title](#title)): bool

*EventBus:*

* Publish(string Topic, data any)

*Request:*

| name            | type   | notes                               |
|-----------------|--------|-------------------------------------|
| standard params |        | *Correlation ID, Origin, Timestamp* |
| Product ID      | int    |                                     |
| Audience Code   | string |                                     |
| DateAdded       | int    | (yyyymmdd)                          |

*Response:*

| field         | type   | notes            |
|---------------|--------|------------------|
| CorrelationID | string | from the request |
| Origin        | string | `"title_etl"`    |
| Timestamp     | int    | `now()`          |
| EventType     | string | `"notice"`       |
| Product ID    | int    |                  |

**Rules:**

* Actor MUST have permission
* Title MAY EXIST

**Except:**

* Not Authorized
* Ignored because old

## Entities

---

### Title

Respresents the full detail of a title item.

#### Fields

| Field                | Type     | Description                                                                                  |
|----------------------|----------|----------------------------------------------------------------------------------------------|
| ID                   | int      | a primary key                                                                                |
| ProductKey           | int      | reference to the title this title item refers to                                             |
| AudienceCode         | string   |                                                                                              |
| DateAdded            | int      | an integer encoded date (format `yyyymmdd`) defining when the title was purchased (0 = null) |
| UpdatedAt            | int      | a timestamp indicating when this record was last updated                                     |
| Series               | string   |                                                                                              |
| Author               | string   |                                                                                              |
| Narrator             | string   |
| SupplierDesc         | string   |                                                                                              |
| ImprintDesc          | string   |                                                                                              |
| ESupplierLiteral     | string   |                                                                                              |
| EFormatLiteral       | string[] |                                                                                              |
| BTKeys               | string[] |                                                                                              |
| PubDate              | int      | a yyyymmdd formatted date as an integer                                                      |
| LanguageDesc         | string   |                                                                                              |
| BISAC                | string[] |                                                                                              |
| Genre                | string   |                                                                                              |                                                                                            |
| AudienceDesc         | string   |                                                                                              |
| UpdateDate           | int      | a yyyymmdd formatted date as an integer                                                      |
| Annotation           | string   |                                                                                              |
| AvgRating            | int      |                                                                                              |
| ReadyToVend          | bool     |                                                                                              |
| PurchaseOption       | string   |                                                                                              |
| Edition              | string   |                                                                                              |
| JacketImageInd       | bool     |                                                                                              |
| SubTitle             | string   |                                                                                              |
| ShortTitle           | string   |                                                                                              |
| AudioSampleAvailable | string   |                                                                                              |
| AxisAttribute        | string   |                                                                                              |
| RunTime              | int      |                                                                                              |
| PageCount            | int      |                                                                                              |
| Lexile               | string   |                                                                                              |
| PopularityScore      | int      |                                                                                              |
| DeletedDate          | int      | a yyyymmdd formatted date as an integer. 0 == not deleted.                                   |

#### Business