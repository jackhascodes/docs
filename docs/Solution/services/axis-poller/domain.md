# Axis Poller: Domain Overview

[[_TOC_]]

## Axis Poller Use Cases

### Poll
Poll should:
* create a unique reservation ID
* reserve a number of records in the EventQueue
* retrieve those records
* if no records were retrieved:
  * sleep an amount of time.
  * start again.
* if records were retrieved:
  * publish the events described by those records
  * keep track of any failures to publish
  * update the status of the queue items as either true or false, depending on whether
    the event published successfully or not.
  * start again

**Publish Topic**: see [LIUD Database Injections](../../injection-points/LIUD-databse.md)

**Actors:** Self running

**Ports:**

*Repo:*

* Reserve(uuid ID, int amount): bool
* Get(uuid ID): [QueueJob](queuejob)
* UpdateByJobID(int ID, bool status): bool
* UpdateByReservationID(uuid ID, bool status): bool


*EventBus:*

* Publish(Event event)

---

## Entities

### QueueJob
Object:

| Field     | Type      | Notes       |
|-----------|-----------|-------------|
| ID        | int       | Primary Key |
| Topic     | string    |             |
| Type      | EventType |             |
| Body      | string    |             |
| ReserveID | ?uuid     |             |
| Status    | ?bool     |             |

### Event
Object:

| Field   | Type        | Notes |
|---------|-------------|-------|
| Topic   | string      |       |
| Body    | string      |       |
| Headers | EventHeader |       |

### EventType

Enum:
* "notice"
* "delta"
* "document"

### EventHeaders
Object:

| Field         | Type                         | Notes          |
|---------------|------------------------------|----------------|
| CorrelationID | uuid                         |                |
| Origin        | string                       | "axis-poller"  |
| Timestamp     | int                          | unix Timestamp |
| Type          | enum [notice/delta/document] |                |
