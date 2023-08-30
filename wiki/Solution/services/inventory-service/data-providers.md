# Inventory SERVICE: Data Providers

[[_TOC_]]

Question: Does ItemAvailability in mongo cover this service?
Can we include listIDs to get lists from cogsearch?

Inventory Service will maintain its own persistence repository. To enable efficient and extensible storage of
things like BTKey aliasing of products, this should be a relational database.

## Repository Schema

### Inventory

| Field             | Type      | Notes                              |
|-------------------|-----------|------------------------------------|
| ProductID         | INT       | composite primary with inventoryID |
| InventoryID       | INT       | composite primary with productID   |
| AudienceCode      | VARCHAR   |                                    |
| DateAdded         | TIMESTAMP |                                    |
| AvailableQuantity | INT       |                                    |
| Scoped            | TINYINT   |                                    |
| UpdatedAt         | TIMESTAMP |                                    |
| DeletedAt         | TIMESTAMP | Allows soft deletion               |

### BT Alias

| Field       | Type    | Notes                    |
|-------------|---------|--------------------------|
| InventoryID | INT     | Foreign Key to Inventory |
| BTKey       | VARCHAR |                          |
| Format      | VARCHAR |                          |

## Required Methods

*Note that any SQL in this document is an approximation intended to demonstrate, not the final intended code.*

### Delete(int ProductID, int InventoryID): bool

This should be a soft delete, updating the relevant record's DeletedAt timestamp to now.

### Get(int ProductID, int InventoryID): [Inventory](domain.md#inventory)

Retrieve a single inventory record with the relevant primary key

### GetByProductID(int ProductID): [Inventory](domain.md#inventory)[]

This should retrieve all matching inventory items regardless of availability or scoping.

  ```
  SELECT
      available,
      btkey,
      format,
      etc,
  from inventory i
  join btkeys b on i.id = b.inventory_id and deleted_at is null
      where ISBN = 123
  ```

Resultset should look like:

| isbn | inventoryID | available | btkey   | format | audienceCode | dateAdded  |
|------|-------------|-----------|---------|--------|--------------|------------|
| 123  | 1           | 1         | abc-def | EPUB   | g            | 0000-00-00 |
| 123  | 1           | 0         | def-ghi | XPS    | g            | 0000-00-00 |
| 123  | 3           | 1         | abc-def | EPUB   | m            | 0000-00-00 |

### GetByAvailableInventory(int[] ProductID, int[] InventoryIDList): [Inventory](domain.md#inventory)

This should retrieve the most available state (excluding scoped) for products in a given set of inventories.

  ```
  SELECT
      isbn,
      available,
      btkey,
      format,
      etc,
      rank () over (
          partition by isbn
          order by 
              available DESC,     -- available is either 1 or 0 
              priority ASC        -- should take care of subscriptions being used only if library does not have.
      ) available_priority
  from inventory i
  join btkeys b on i.id = b.inventory_id and deleted_at is null 
      where ISBN in (a,b,c) 
      inventoryID in (x,y,z)
      and available >= 0
      and available_priority = 1
  ```

Resultset should look like:

| isbn | inventoryID | available | btkey   | format | audienceCode | dateAdded  |
|------|-------------|-----------|---------|--------|--------------|------------|
| 123  | 1           | 1         | abc-def | EPUB   | g            | 0000-00-00 |
| 456  | 2           | 0         | ghi-jkl | EPUB   | g            | 0000-00-00 |
| 789  | 1           | 1         | mno-pqr | EPUB   | g            | 0000-00-00 |

### Save([Inventory](domain.md#inventory) inventory): bool

This should be an upsert with inventoryID and productID as a composite primary key.