# Inventory Clients Service

## Use Cases

**Conventions:**

* Dates will be represented as yyyymmdd integers (e.g. Jan 3 2020 would be 20200103).
* All requests should have:
    * Correlation ID: string - a unique identifier used to trace a request across services.
    * Origin: string - an identifier for the source of the request.
    * Timestamp: int - a unix timestamp for when the request was *made*.
* All responses should forward the Correlation ID supplied in the request
* All responses should set a new origin (i.e. `inventory-access`)
* All responses should set a new timestamp (i.e. `now()`)
* Use cases with Publish Topics should publish:
    * `foo.bar.success` when a valid response is generated.
    * `foo.bar.fail` when an exception occurs.

  to the event bus.

### Upsert
**Publish Topic:** `inventory.access.update.[success|fail]`

**Actors:**

* ppc budget events
* consortia change events

**Ports:**

*Repo:*
* Get(string ClientRef, string InventoryRef): ?[InventoryAccess](#inventoryaccess)
* Save([InventoryAccess](#inventoryaccess)): bool

*EventBus:*
* Publish(string Topic, data any)

*Request:*

| name            | type   | notes                                                                                                     |
|-----------------|--------|-----------------------------------------------------------------------------------------------------------|
| standard params |        | *Correlation ID, Origin, Timestamp*                                                                       |
| ClientRef       | string |                                                                                                           |
| InventoryRef    | string |                                                                                                           |
| Blocked         | bool   | A client may view this inventory, but items in this inventory are effectively unavailable to that client. |

Update or insert a relationship between an inventory and a client.

### Get Client Access
Retrieve a list of InventoryIDs for which the requested client has access (i.e. is linked and not blocked).

**Publish Topic:** DO NOT PUBLISH

**Actors:**

* other services

**Ports:**

*Repo:*
* GetActiveInventories(string ClientRef): [InventoryAccess[]](#inventoryaccess)

*Request:*

| name            | type   | notes                               |
|-----------------|--------|-------------------------------------|
| standard params |        | *Correlation ID, Origin, Timestamp* |
| ClientRef       | string |                                     |

*Response:*
string[] of Inventory IDs 

## Entities

### InventoryAccess

| Field        | Type   | Notes |
|--------------|--------|-------|
| InventoryID  | int    |       |
| InventoryRef | string |       |
| ClientRef    | string |       |
| Blocked      | bool   |       |