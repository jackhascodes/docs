# Inventory SERVICE: Domain Overview

[[_TOC_]]

## Use Cases

**Conventions:**

* Dates will be represented as yyyymmdd integers (e.g. Jan 3 2020 would be 20200103).
* All requests should have:
    * Correlation ID: string - a unique identifier used to trace a request across services.
    * Origin: string - an identifier for the source of the request.
    * Timestamp: int - a unix timestamp for when the request was *made*.
* All responses should forward the Correlation ID supplied in the request
* All responses should set a new origin (i.e. `inventory-service`)
* All responses should set a new timestamp (i.e. `now()`)
* Use cases with Publish Topics should publish:
    * `foo.bar.success` when a valid response is generated.
    * `foo.bar.fail` when an exception occurs.

  to the event bus.

---

### Delete

Make a single inventory record unavailable for retrieval.

**Publish Topic**: `inventory.delete.[success|fail]`

**Actors:** availability delete events

**Ports:**

* *Repo:*
    * Delete(int LibraryID, int ProductID): bool
* *EventBus:*
    * Publish(string Topic, data any)

* *Request:*

| name            | type | notes                               |
 |-----------------|------|-------------------------------------|
| standard params |      | *Correlation ID, Origin, Timestamp* |
| Product ID      | int  |                                     |
| Library Key     | int  |                                     |

* *Response:*

  | field         | type   | notes             |
      |---------------|--------|-------------------|
  | CorrelationID | string | from the request  |
  | Origin        | string | `"inventory_etl"` |
  | Timestamp     | int    | `now()`           |
  | EventType     | string | `"notice"`        |
  | Product ID    | int    |                   |
  | Library Key   | int    |                   |

**Rules:**

* Actor MUST have permission
* Inventory MUST EXIST
* Product ID MUST become unavailable for this Library Key

**Except:**

* Not Authorized
* Ignored because old

---

### GetByProductId

Return inventory records from inventories which hold this product.

**Publish Topic:** DO NOT PUBLISH

**Actors:**

* Other services
* Users

**Ports:**

*Repo:*

* Get(intProductID): [Inventory](#inventory)

*EventBus:*

* Publish(string Topic, data any)

*Request:*

| name            | type | notes                               |
|-----------------|------|-------------------------------------|
| standard params |      | *Correlation ID, Origin, Timestamp* |
| Product ID      | int  |                                     |

*Response:*

| field         | type                              | notes            |
|---------------|-----------------------------------|------------------|
| CorrelationID | string                            | from the request |
| Origin        | string                            | `inventory_etl`  |
| Timestamp     | int                               | `now()`          |
| Inventory     | [InventoryShort](#inventoryshort) |                  |

**Rules:**

* Actor MUST have permission
* Use Case MUST return an Inventory record if it exists.

**Except:**

* Not Authorized
* Not found

---

### GetByAvailableInventory

On null result from repo, should return an Inventory object with InventoryID = ClientID, ProductID = productID,
Availability = 0, BTKey = null. In downstream systems, BTKey = null should allow a distinction to be made between
`unavailable no access`, or `unavailable no quantity`. This would cover the PPC out of budget problem, where a title
*should* be represented, but unavailable, since we can't directly cause PPC inventory to be unavailable on its own.

**Publish Topic:** DO NOT PUBLISH

**Actors:**

* Search ETL
* Users

**Ports:**

*Inventory Access:*

* GetInventories(int clientID): []InventoryIDList

*Repo:*

* GetByProduct(int ProductID): [Inventory](#inventory)[]

*EventBus:*

* Publish(string Topic, data any)

*Request:*

| name            | type | notes                               |
|-----------------|------|-------------------------------------|
| standard params |      | *Correlation ID, Origin, Timestamp* |
| Product ID      | int  |                                     |
| ClientID        | int  | The requesting library              |

*Response:*

| field         | type                           | notes                           |
|---------------|--------------------------------|---------------------------------|
| CorrelationID | string                         | from the request                |
| Origin        | string                         | `inventory_etl`                 |
| Timestamp     | int                            | `now()`                         |
| Inventory     | [Inventory](#inventoryshort)[] | array of full inventory records |

**Rules:**

* Actor MUST have permission.
* Use Case MUST return an array.
* Use Case MUST return an array
    * The Array MAY contain InventoryShort DTOs when the Product ID has related inventory items.
    * The Array MAY be empty if the Product ID has no related inventory items.

**Except:**

* Not Authorized

---

### Increment

**Publish Topic:** `inventory.update.[success|fail]`
**Actors:**

* availability increment events

**Ports:**

*Repo:*

* Get(int InventoryID, int ProductID): ?[Inventory](#inventory)
* Save([Inventory](#inventory)): bool

*EventBus:*

* Publish(string Topic, data any)

*Request:*

| name            | type | notes                               |
|-----------------|------|-------------------------------------|
| standard params |      | *Correlation ID, Origin, Timestamp* |
| ProductID       | int  |                                     |
| InventoryID     | int  |                                     |
| Increment       | int  | one of [-1/1]                       |

*Response:*

| field         | type   | notes             |
|---------------|--------|-------------------|
| CorrelationID | string | from the request  |
| Origin        | string | `"inventory_etl"` |
| Timestamp     | int    | `now()`           |
| EventType     | string | `"notice"`        |
| Product ID    | int    |                   |
| Library Key   | int    |                   |

**Rules:**

* Actor MUST have permission
* Inventory MUST EXIST
* Increment MAY be EITHER `-1` OR `1`
* Inventory.AvailableQuantity MUST NOT be decremented below `0`
* When Increment IS `-1` AND Increment.AvailableQuantity IS GREATER THAN `0`:
    * Inventory.AvailableQuantity MAY reduce by 1
* When Increment IS `1`:
    * Inventory.AvailableQuantity MUST increase by 1

**Except:**

* Not Authorized

---

### Quantity Change

**Publish Topic:** `inventory.update.[success|fail]`

**Actors:**

* availability quantity changed events

**Ports:**

*Repo:*

* Get(int LibraryID, int ProductID): ?[Inventory](#inventory)
* Save([Inventory](#inventory)): bool

*EventBus:*

* Publish(string Topic, data any)

*Request:*

| name              | type | notes                               |
|-------------------|------|-------------------------------------|
| standard params   |      | *Correlation ID, Origin, Timestamp* |
| Product ID        | int  |                                     |
| Library Key       | int  |                                     |
| AvailableQuantity | int  |                                     |

**Response:**

| field         | type   | notes             |
|---------------|--------|-------------------|
| CorrelationID | string | from the request  |
| Origin        | string | `"inventory_etl"` |
| Timestamp     | int    | `now()`           |
| EventType     | string | `"notice"`        |
| Product ID    | int    |                   |
| Library Key   | int    |                   |

**Rules:**

* Actor MUST have permission
* Inventory MAY EXIST
* When Inventory DOES NOT EXIST:
    * Use Case MUST create a new Inventory record.
    * `Inventory.Scope` MUST be set to `false`
    * `Inventory.AvailableQuantity` MUST be set to `request.AvailableQuantity`
    * `Inventory.DateAdded` MUST be set to `0`
    * `Inventory.AudienceCode` MUST be set to `""`
* When Inventory EXISTS:
    * When request.timestamp <= existing.updatedAt (i.e. request is too old):
        * Use Case MUST NOT update
        * Use Case MUST ignore the request
    * When request.timestamp > existing.updatedAt:
        * `Inventory.Scope` MUST NOT be changed
        * `Inventory.AvailableQuantity` MUST be set to `request.AvailableQuantity`
        * `Inventory.DateAdded` MUST NOT be changed
        * `Inventory.AudienceCode` MUST NOT be changed

**Except:**

* Not Authorized
* Ignored because old

---

### Scope

**Publish Topic:** `inventory.update.[success|fail]`

**Actors:**

* availability scope events

**Ports:**

*Repo:*

* Get(int LibraryID, int ProductID): ?[Inventory](#inventory)
* Save([Inventory](#inventory)): bool

*EventBus:*

* Publish(string Topic, data any)

*Request:*

| name            | type | notes                               |
|-----------------|------|-------------------------------------|
| standard params |      | *Correlation ID, Origin, Timestamp* |
| Product ID      | int  |                                     |
| Library Key     | int  |                                     |
| Scoped          | bool |                                     |

*Response:*

| field         | type   | notes             |
|---------------|--------|-------------------|
| CorrelationID | string | from the request  |
| Origin        | string | `"inventory_etl"` |
| Timestamp     | int    | `now()`           |
| EventType     | string | `"notice"`        |
| Product ID    | int    |                   |
| Library Key   | int    |                   |

**Rules:**

* Actor MUST have permission
* Inventory MAY EXIST
* When Inventory DOES NOT EXIST:
    * Use Case MUST create a new Inventory record.
    * `Inventory.Scope` MUST be set to `request.Scoped`
    * `Inventory.AvailableQuantity` MUST be set to `0`
    * `Inventory.DateAdded` MUST be set to `0`
    * `Inventory.AudienceCode` MUST be set to `""`
* When Inventory EXISTS:
    * When request.timestamp <= existing.updatedAt (i.e. request is too old):
        * Use Case MUST NOT update
        * Use Case MUST ignore the request
    * When request.timestamp > existing.updatedAt:
        * `Inventory.Scope` MUST be set to `request.Scoped`
        * `Inventory.AvailableQuantity` MUST NOT be changed
        * `Inventory.DateAdded` MUST NOT be changed
        * `Inventory.AudienceCode` MUST NOT be changed

**Except:**

* Not Authorized
* Ignored because old

---

### Upsert

**Publish Topic:** `inventory.update.[success|fail]`

**Actors:**

* availability update events

*Repo:*

* Get(int LibraryID, int ProductID): ?[Inventory](#inventory)
* Save([Inventory](#inventory)): bool

*EventBus:*

* Publish(string Topic, data any)

*Request:*

| name              | type   | notes                               |
|-------------------|--------|-------------------------------------|
| standard params   |        | *Correlation ID, Origin, Timestamp* |
| Product ID        | int    |                                     |
| Library Key       | int    |                                     |
| Audience Code     | string |                                     |
| DateAdded         | int    | (yyyymmdd)                          |
| AvailableQuantity | int    |                                     |

*Response:*

| field         | type   | notes             |
|---------------|--------|-------------------|
| CorrelationID | string | from the request  |
| Origin        | string | `"inventory_etl"` |
| Timestamp     | int    | `now()`           |
| EventType     | string | `"notice"`        |
| Product ID    | int    |                   |
| Library Key   | int    |                   |

**Rules:**

* Actor MUST have permission
* Inventory MAY EXIST
* When Inventory DOES NOT EXIST:
    * Use Case MUST create a new Inventory record.
    * `Inventory.Scope` MUST be set to `false`
    * `Inventory.AvailableQuantity` MUST be set to `request.AvailableQuantity`
    * `Inventory.DateAdded` MUST be set to `request.DateAdded`
    * `Inventory.AudienceCode` MUST be set to `request.AudienceCode`
* When Inventory EXISTS:
    * When request.timestamp <= existing.updatedAt (i.e. request is too old):
        * Use Case MUST NOT update
        * Use Case MUST ignore the request
    * When request.timestamp > existing.updatedAt:
        * `Inventory.Scope` MUST NOT be changed
        * `Inventory.AvailableQuantity` MUST be set to `request.AvailableQuantity`
        * `Inventory.DateAdded` MUST be set to `request.DateAdded`
        * `Inventory.AudienceCode` MUST be set to `request.AudienceCode`

**Except:**

* Not Authorized
* Ignored because old

## Entities

---

### Inventory

Respresents the full detail of an inventory item.

---

#### Fields

| Field             | Type    | Description                                                                                               |
|-------------------|---------|-----------------------------------------------------------------------------------------------------------|
| ID                | int     | a primary key                                                                                             |
| InventoryID       | int     | reference to the library who owns this inventory item                                                     |
| ProductID         | int     | reference to the title this inventory item refers to                                                      |
| AudienceCode      | string? | nullable, defines an override of audience as compared to the title's definition                           |
| DateAdded         | int     | an integer encoded date (format `yyyymmdd`) defining when the title was purchased (0 = null)              |
| Available         | enum    | one of `-1`,`0`,`1`. Where `-1` = scoped, `0` = unavailable, `1` = available                              |
| AvailableQuantity | int     | how many copies of this item are available for checkout                                                   |
| Scoped            | bool    | whether this item has been [scoped out](../../../Glossary.md#scopedscoped-out) (`true`) or not (`false`). |
| BTKey             | string  | An identifier for the library's ownership terms                                                           |
| UpdatedAt         | int     | a timestamp indicating when this record was last updated                                                  |
| BTKey             | string  |                                                                                                           |
| Format            | string  |                                                                                                           |

---

#### Business

* `Inventory.Available` is determined by `AvailableQuantity` and `Scoped` values.
* When `Scoped` IS `true`, `Available` is ALWAYS `-1`
* When `Scoped` IS `false` AND `AvailableQuantity` IS `0`, `Available` is `0`
* When `Scoped` IS `false` AND `AvailableQuantity` IS greater than `0`, `Available` is `1`
* `Inventory` should make available a condensed version of these fields (see below [InventoryShort](#inventoryshort))

---

### InventoryShort

Represents a condensed version of an inventory item for direct reference in search documents.

| Field        | Type    | Description                                                                                  |
|--------------|---------|----------------------------------------------------------------------------------------------|
| ID           | int     | a primary key                                                                                |
| InventoryID  | int     | reference to the library who owns this inventory item                                        |
| ProductID    | int     | reference to the title this inventory item refers to                                         |
| AudienceCode | string? | nullable, defines an override of audience as compared to the title's definition              |
| DateAdded    | int     | an integer encoded date (format `yyyymmdd`) defining when the title was purchased (0 = null) |
| Available    | enum    | one of `-1`,`0`,`1`. Where `-1` = scoped, `0` = unavailable, `1` = available                 |
| BTKey        | string  |                                                                                              |
| Format       | string  |                                                                                              |