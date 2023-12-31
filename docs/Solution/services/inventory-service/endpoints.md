# Inventory SERVICE: Endpoints

[[_TOC_]]

## Event Subscriptions

### availability.checkout.success

**Event Schema:** [Axis Inventory Checkout](../../events.md#axis-inventory-checkout)

**Use Case:** [Increment](domain.md#increment)

**Handler rules:**

* Message body MUST contain:
    * LibraryID
    * ProductID
* Increment call MUST use `1`

---

### availability.deleted.success

**Event Schema:** [Axis Inventory Delete](../../events.md#axis-inventory-delete)

**Use Case:** [Delete](domain.md#delete)

**Handler rules:**

* Message body MUST contain:
    * LibraryID
    * ProductID

---

### availability.quantity.changed.success

**Event Schema:** [Axis Checkout](../../events.md#axis-inventory-quantity-change)

**Use Case:** [Quantity Change](domain.md#quantity-change)

**Handler rules:**

* Message body MUST contain:
    * LibraryID
    * ProductID
    * A quantity value

---

### availability.return.success

**Event Schema:** [Axis Checkout](../../events.md#axis-inventory-return)

**Use Case:** [Increment](domain.md#increment)

**Handler rules:**

* Message body MUST contain:
    * LibraryID
    * ProductID
* Increment call MUST use `-1`

---

### availability.scope.changed.success

**Event Schema:** [Axis Checkout](../../events.md#axis-inventory-scope-change)

**Use Case:** [Scope](domain.md#scope)

**Handler rules:**

* Message body MUST contain:
    * LibraryID
    * ProductID
    * A boolean value

---

### availability.updated.success

**Event Schema:** [Axis Checkout](../../events.md#axis-inventory-upsert)

**Use Case:** [Upsert](domain.md#upsert)

---

## HTTP Handlers

**Headers**

All HTTP requests should carry the following headers

| Header        | Type                 | Reason                                                              |
|---------------|----------------------|---------------------------------------------------------------------|
| CorrelationID | UUID                 | Tracks the relationship between different events in the same chain. |
| Origin        | string               | Tracks the service which spawned the event.                         |                                            
| Timestamp     | int (unix Timestamp) | Tracks the sequencing (and in some cases applicability) of events.  |

Any event generated by a use case which publishes response events should forward the `CorrelationID` of the originally
consumed request.

The `Origin` header should be used to provide information on where a request came from. It should be a namespaced path.

---

### GET /{productId}/{clientId}

**Request Schema:**

| Field     | Type | Location  | Notes |
|-----------|------|-----------|-------|
| ProductID | int  | URL Param |       |
| ClientID  | int  | URL Param |       |

**Use Case:** [Get](domain.md#getbyavailableinventory) 

---

### GET /{productId}

**Request Schema:**

| Field     | Type | Location  | Notes |
|-----------|------|-----------|-------|
| ProductID | int  | URL Param |       |

**Use Case:** [Get By ProductID](domain.md#getbyproductid)