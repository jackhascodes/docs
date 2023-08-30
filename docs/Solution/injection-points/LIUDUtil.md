[[_TOC_]]

# Code Injections in LIUDUtil (dotnet component)
Given that LIUDUtil is a common component between VendorAPI and Axis360/Boundless, this is where the lightest touch on
persistent events may be handled. While it is true that the functions served by LIUDUtil may not always route through 
LIUDUtil, the replacement of these functions will be in future microservices which replace chunks of VendorAPI and/or
Axis. Therefore, LIUDUtil makes the most sense to contain the event injections, as the point at which the relevant
events for Search are persisted is when the event can really be said to have "happened".

It also means that there is a single consistent point in the current architecture to initiate these events, ensuring 
consistency regardless of origin... (TODO: mayyyybe reconsider this, as it would be helpful to know whether events 
originated from VAPI or Axis, unless we can circumvent that by calling app settings or similar from within LIUDUtil.)

We will need to introduce some new Classes to more easily handle Messages.

Doing these changes separately from the injections in LIUD-database enables us to have more "real-time" representation
of the event, rather than waiting for the database poller to pick up the event.

## Checkout
At BusinessObjects/CheckOutTransactionBO.cs:323
If we have reached this line, then no exceptions have been thrown across the whole of the preceeding process.
```
EventBus.Publish(
    "availability.checkout.success", 
    new AvailabilityNoticeMessage(
        checkoutTran.LibraryID, 
        checkoutTran.TitleID
    )
)
```
At BusinessObjects/CheckOutTransactionBO.cs:329
And BusinessObjects/CheckOutTransactionBO.cs:335
(NOTE: `ex` below simply refers to the exception variable being caught.)
```
EventBus.Publish(
    "availability.checkout.fail", 
    new AvailabilityErrorMessage(
        checkoutTran.TitleID,
        checkoutTran.Format, 
        ex.Message()
    )
)
```

## Early Check-in

Kind of a problem here, for now. The early check in only takes a transaction ID - updating the checkout trans status by
that ID - and does not expose the library id or title id being checked in.

TODO: Clarify whether we want to do a get to be able to populate the real time event.

Both `LIUDUtilService.EarlyCheckinTitle()` and `LIUDUtilService.EarlyCheckinItem()` call 
`EarlyCheckinTitleBO.EarlyCheckinTitle()` only with different flags in the params, so we will make our changes in
`EarlyCheckinTitleBO`

At BusinessObjects/EarlyCheckinTitleBO.cs:105
NOTE: predictable 'failures' here are not thrown forward, but returned in a stable object. Therefore, we need to if/else
to check success.
```
if (statuscode !== LIUDConstant.CODE_SUCCESS) {
    EventBus.Publish(
        "availability.checkin.fail", 
        new AvailabilityErrorMessage(
            transactionID,
            statuscode, 
            statusMessage
        )
    )
} else {
    EventBus.Publish(
        "availability.checkout.success", 
        new AvailabilityNoticeMessage(
            checkoutTran.LibraryID, 
            checkoutTran.TitleID
        )
    )
}
```
There is a generic catch block at :106 which should have
```
EventBus.Publish(
        "availability.checkout.success", 
        new AvailabilityNoticeMessage(
            checkoutTran.LibraryID, 
            checkoutTran.TitleID
        )
    )
```