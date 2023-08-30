# Event Patterns
## Notice Pattern
Where an event is published with an entity ID as the only data in the message body.

The message topic and some common headers may be used to provide context.

The subscriber uses the context and ID to reach out to the entity's service for the current state of the entity.

This allows for out of sequence subsrciptions to always have access to the latest state of an entity, while also enabling
a tracking of events as needed.

However, subscribers would generally need to reach out to an entity's service in order to get more information, unless
the topic is informative enough to do work with. This could cause unnecessary network traffic, but should be minor.

Examples:

`availability.checkout.success` contains the composite key for an inventory item, and the topic allows us to know that 
quantity has decreased without needing to retrieve the originating data from anywhere.

`inventory.updated.success`, on the other hand, requires that a subscriber reach back to the inventory service to find 
out what changed.

## Delta Pattern
Where an event is published with an entity ID and only the fields which have been affected by an event.

This allows for "thinner" messages, and reduces the amount of callbacks required to retrieve state.

The subscriber will be required to figure out what to do with the differential data.

Examples:

`availability.quantity.updated.success` contains the composite inventory key and a quantity value, but does not hold 
information about scoping state or audience code, whereas `availability.scope.updated.success` would hold the scoping 
state, but not quantity or audience code. This kind of differential would be published from events that don't actually 
know or have anything to do with each other's relevant data. Any subscribers would need to be able to know how to handle
the different message bodies appropriately.

## Document Pattern
Where an event is published with the state of an entity represented in full.

An event may have access to the full dataset of an entity, because the cause was functionally equivalent to an update 
where everything changed, or a new entity was created.

This allows the subscriber to have access to the entire data set without requiring any further communication, but does 
result in "heavier" messages.

Examples:

`omni.updated.success` contains the entirety of a title's information, because all fields are handled in the event cause
regardless of whether a title is new or simply being updated. Because these changes are relatively infrequent, it can be
assumed "safe" to send the whole state without worrying about out-of-sequence data.


## Applicability

`notice` events should be preferred, except where it is not possible to run a retrieval callback (e.g. events from
Axis or LIUD).

`document` events should be preferred when a `notice` event is not possible due to a lack of retrieval endpoints, except
where a complete document is not available by the publisher of an event

`delta` events may be used, where `notice` and `document` events are not possible, provided sufficient data is provided
for a subscriber to work with.

Example:
> A scope event may come from LIUD, but scoping data does not include quantity data, and so we cannot provide a whole
> document which represents an inventory item. Therefore we would publish a `delta` event, with LibraryKey, BTKey and
> the scope state.
>
> The same applies for a quantity update, insofar as we cannot provide a whole document. Quantity updates do not include
> scope data. So, again, we would publish a `delta` event, this time with with LibraryKey, BTKey and the Quantity data.

Do note that `document` events may be treated as `delta` events by any subscriber so motivated.

`delta` events may also be treated as `document` events by subscribers, but due to the potential variability of the
message body, this is not recommended.
 