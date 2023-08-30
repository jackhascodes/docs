# Inventory SERVICE: Data Providers

The Inventory Access service will maintain its own persistence repository.

## Repository Schema

### InventoryAccess

| Field        | Type        | Notes                |
|--------------|-------------|----------------------|
| InventoryID  | int         |                      |
| InventoryRef | varchar(36) |                      |
| ClientRef    | varchar(36) |                      |
| Blocked      | bool        |                      |
| UpdatedAt    | TIMESTAMP   |                      |
| DeletedAt    | TIMESTAMP   | Allows soft deletion |


## Required Methods

* Get(string ClientRef, string InventoryRef): [InventoryAccess](domain.md#inventoryaccess)
* GetByClient(string ClientRef): [InventoryAccess](domain.md#inventoryaccess)
* Save([InventoryAccess](domain.md#inventoryaccess) inventoryAccess): bool