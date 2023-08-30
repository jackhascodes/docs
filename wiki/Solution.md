# Overview
This is a basic overview of the various parts of the solution. For more detail on any of these sections, please refer
to the "See also" links in each section.
 
## A Note on Subscription Services
Subscription services such as PPC will be treated as their own set of inventory. Libraries who subscribe to these 
services will search them using the same technique as a parent in a consortium.

Therefore, it will be helpful to think of inventories more as collections of titles available to a given entity. The 
majority of these entities will be libraries, but may be other things such as subcription providers, or supra-library 
entities. 

## Services
(see also: [Services](Solution/services.md))

To handle the state changes, and keep the index up to date

## Injection Points
(see also: [Injection Points](Solution/injection-points.md))

Changes to existing services will need to be made in order to generate events which are consumed by new services.

## Data Structures
(see also: [Data Structures](Solution/data-structures.md))

Search records in ACS will be stored in a nested document structure composed of title data, with libraries having access
to a title being an array of inventory data inside the title document. This meets the requirement to minimize index size
and maintenance.
```yaml
TitleRecord:
  ISBN: 1234567890123
  Title: My Book
  # ...etc... 
  Libraries: 
      - LibraryID: 1234
        Availability: 1 # available
      - LibraryID: 3456
        Availability: 0 # not available
      - LibraryID: 4567
        Availability: -1 # scoped
      # ...etc...
```

Because ACS has a hard limit of 3000 complex collections per document, and we may have titles which are owned by more 
than 3000 libraries, this solution needs a way to partition the documents such that every library which owns a title may
be represented. To that end, we have decided to implement the concept of "regions". (see also: 
[Data Structures/Regions](Solution/data-structures.md#regions))

Please note that because we may have other complex fields in the title document, we cannot guarantee 3000 slots for 
library inventory alone. Therefore, we may allocate only 2500 libraries to a title.


## Events
(see also: [Events](Solution/events.md))

In order to keep the index up to date, we must be able to handle state changes for inventory and titles.

The following state changes have been identified.
* Availability/Access changes:
    * A library purchases a title
    * A library over-rides the title's audience key
    * A library deletes a title
    * A change in available quantity of a title (check-in or check-out)
    * A change in scoping
    * A library subscribes to an available set of titles (PPC, bundles, etc)
* Title Changes:
    * A title is created
    * A title is updated
    * A title is deleted
* Library Changes:
    * A library is created
    * A library is updated
    * A library is deleted

## Infrastructure
(see also: [Infrastructure](Solution/infrastructure.md))

## Migration
Each service described in these docs outlines its own migration. However, the majority are reliant on the Axis Poller 
service to bulk dump relevant data and process in a queued asynchronous process which can be rolled back/forward at 
will. 