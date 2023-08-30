# Search ETL: Endpoints

[[_TOC_]]

## Event Subscriptions

### title.updated.success

**Event Schema:** [Search Title Updated](../../events.md#search-title-update)

**Use Case:** [Update Title](domain.md#update-title)

**Handler rules:**

* Message body MUST contain:
    * ProductID

---

### inventory.updated.success

**Event Schema:** [Search Inventory Updated](../../events.md#search-inventory-update)

**Use Case:** [Update Inventory](domain.md#update-inventory)

**Handler rules:**

* Message body MUST contain:
    * LibraryID
    * ProductID

---

## HTTP Handlers

This service does not require HTTP handlers at this time.