# Start Here
**NOTE: This documentation is currently a living design document and subject to major changes.**

"Search" is a combination of services used to input and extract data from Azure Cognitive Search (ACS).

## Background
We have titles which may belong to multiple libraries.

A Library may not have access to a title directly and still have access via their parent.

A Library and their parent may both independently have access to a title.

Where a library checks out from their own access, the available quantity for that library decrements.

Where a library checks out from their parent's access, the available quantity for that parent decrements.

Some titles are always available and belong to a collection.

There are titles which may be always available as part of a collection and to which child-libraries
are subscribed by their parent.

There are titles which may be available via a parent library, but which a child library has opted not to supply (scoped 
out).

Note that "access" is different from "availability". A library may have access to a Title but have no availability due
to all copies being checked out.

There are effectively 2 kinds of update which may affect data:
* A Title's details are updated.
* A Library's access or availability is updated.

## Requirements Overview
Full documentation of requirements may be found [here](Requirements.md).

## Solution Overview
Full documentation of the solution design may be found [here](Solution.md). These documents are intended to 
describe the data structures, service relationships and other implementation design details used to build Search.

## Delivery Checklist
A brief document to keep track of components required for delivery may be found [here](Delivery.md)