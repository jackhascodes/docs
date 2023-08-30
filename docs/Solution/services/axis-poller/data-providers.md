[[_TOC_]]
# Axis Poller: Data Providers

## Repo
Uses the exsting [LIUD](../../infrastructure#liud) database in [SQL Server](../../infrastructure#sql-server)

### Schema
The Axis Poller will require a new table in LIUD, from which it will pull awaiting jobs.

#### EventQueue

| Field      | Type             | Notes                                           |
|------------|------------------|-------------------------------------------------|
| ID         | INT              | Primary Key                                     |
| Topic      | VARCHAR(50)      |                                                 |
| Type       | VARCHAR(8)       | Should always be one of [notice/delta/document] |
| Message    | TEXT             |                                                 |
| ReserveID  | UNIQUEIDENTIFIER | Nullable, default null                          |
| Status     | TINYINT          | Nullable, default null, 0 - failed, 1 - handled |
| CreatedAt  | TIMESTAMP        |                                                 |
| ReservedAt | TIMESTAMP        | Nullable, default null                          |

### Methods

#### Reserve(uuid id, int amount)
Reserve a number of records. 

#### Get(uuid ID): [QueueJob](domain.md#queuejob)
Retrieve all records with a given reservation ID.

#### UpdateByEventID(int ID, bool status)
Set the status of a given event

#### UpdateByReservationID(uuid ID, bool status)
Set the status of all events with a given reservation ID.

## EventBus
Uses [Azure EventHub](../../infrastructure#azure-eventhub)
### Methods
#### Publish([Event](domain.md#event) event): bool
Send the event to [EventHub](../../infrastructure#azure-eventhub)
