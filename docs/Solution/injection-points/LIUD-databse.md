    NOTE UPFRONT: The insertions here are not accurate, as we need to figure out how to get a Parent Library ID if the 
    checkin/checkout is being made from the parent. 

[[_TOC_]]

# Code Injections in LIUD (Database)

To facilitate moving "events" out of OMNE, current batch stored procedures will require additional insertions of their
results into the new event_queue table (see also: [Axis Poller](../services/axis-poller.md))

Prefer `+`
over `CONCAT()` ([see here](https://stevestedman.com/2021/01/performance-string-concatenation-in-sql-server/))

## dbo.AddNewLibrary

```sql
INSERT INTO EventQueue (Topic, Type, Message)
VALUES ('availability.access.updated.success',
        'delta',
        '{"ClientID":"' + @newLibraryID + '","InventoryRef":"' + @newLibraryID + ',"Blocked":0}')
```

## dbo.AddNewVendorLibraryAccess

```sql
INSERT INTO EventQueue (Topic, Type, Message)
VALUES ('availability.access.updated.success',
        'delta',
        '{"ClientID":"' + @LibraryID + '","InventoryRef":"' + @VendorID + ',"Blocked":0}')
```

## dbo.UpdateTablePPCLibraryBudget

Note, this doesn't appear to be called from anywhere inside LIUD, and isn't referenced in LIUDUtils or VAPI... Where is
it actually called from, since that may be a better injection point, long-term.

```sql
INSERT INTO EventQueue (Topic, Type, Message)
VALUES ('availability.access.updated.success',
        'delta',
        '{"ClientID":"' + @libraryKey + '","InventoryRef":"?PPC?","Blocked":@todaysBudgetDisabled}')
```

## dbo.UpdateLibraryRolloverBudget

Note, this doesn't appear to be called from anywhere inside LIUD, and isn't referenced in LIUDUtils or VAPI... Where is
it actually called from, since that may be a better injection point, long-term.

```sql
INSERT INTO EventQueue (Topic, Type, Message)
VALUES ('availability.access.updated.success',
        'delta',
        '{"ClientID":"' + @libraryKey + '","InventoryRef":"?PPC?","Blocked":0}')
```

## dbo.AutoReturnProcess

```sql
INSERT INTO EventQueue (Topic, Type, Message)
SELECT 'availability.return.success',
       'notice',
       '{"ItemID":"' + E.ItemID + '","InventoryRef":"' + E.LibraryID + '"}'
FROM #ExpiredItem E
```

## dbo.AutoRemoveExpiredHold

```sql
INSERT INTO EventQueue (Topic, Type, Message)
SELECT 'availability.return.success',
       'notice',
       '{"ItemID":"' + E.ItemID + '","InventoryRef":"' + E.LibraryID + '"}'
FROM #ExpiredHold E
```

## dbo.AutoRemoveExpiredCheckoutList

    QUESTION: Is this still a thing? PatronCheckoutList doesn't appear to be a table or view (at least in dev)

(before delete)

```sql
INSERT INTO EventQueue (Topic, Type, Message)
SELECT 'availability.return.success',
       'notice',
       '{"ItemID":"' + P.ItemID + '","InventoryRef":"' + P.LibraryID + '"}'
FROM PatronCheckoutList P
WHERE DATEDIFF(minute, AddedDate, @currentTime) >= 15;

```

## dbo.AutoReserveProcess

    QUESTION: Despite not performing a checkout, does this affect actual availability?

    I.E. Is it functionally equivalent to a checkout where availability is concerned?

```sql
INSERT INTO EventQueue (Topic, Type, Message)
SELECT 'availability.checkout.success',
       'notice',
       '{"ItemID":"' + H.ItemID + '","InventoryRef":"' + H.LibraryID + '"}'
FROM #Holds H
```

## dbo.SOPImportInsertGeneralOrders

@line 133 before second END (i.e. LI has been either updated or inserted, but not ignored.)

```sql
INSERT INTO EventQueue (Topic, Type, Message)
VALUES ('availability.updated.success',
        'delta',
        '{
            "ItemID":"' + @most_common_ItemID + '", 
        "InventoryRef":"' + @libraryid + '",
        "AvailableQuantity":"' + @shipped_quantity + '"
        "CreateDate":"' + NOW() + '",
        "UpdatedDate":"' + NOW() + '",
        "HasCirculationLimit":"' + @HasCirculationLimit + '", 
        "RTVFormats":"' + @rtvFormats + '",
        "ISBN":"' + @isbn + '"
	}')
```

## dbo.SOPImportUpdateItemInDeletedItemTable

* @ line 28 before END
  child in a consortium is removing their inventory of this item -- self owned? TODO: Check this is true.

  Could be that when @consortiaType is 'C', that doesn't apply. Child libraries are also their own parents?
* @ line 38 before END
  parent in a consortium is removing their inventory of this item. Def self owned, and don't need
  to worry about children, so use the libraryID param.

```sql
INSERT INTO EventQueue (Topic, Type, Message)
VALUES ('availability.updated.success',
        'delta',
        '{
            "ItemID":"' + @replacedBTKey + '", 
        "InventoryRef":"' + @libraryID + '",
 	}')
```

## dbo.FillBooksToOMNI

@ line 255
TODO: Check if this ODSFeedStaging is cleared after this process?

```sql
INSERT INTO EventQueue (Topic, Type, Message)
SELECT 'omni.updated.success',
       'document',
       '{
           "BTKey":"' + TT.BTKey + '",
        "ISBN":"' + TT.ISBN + '",
        "Title":"' + TT.Title + '",
        "SubTitle":"' + TT.SubTitle + '",
        "ShortTitle ":"' + TT.ShortTitle + '",
        "Series":"' + TT.Series + '",
        "Author":"' + TT.Author + '",
        "SupplierDesc":"' + TT.SupplierDesc + '",
        "ESupplierLiteral":"' + TT.ESupplierLiteral + '",
        "EFormatLiteral":"' + TT.EFormatLiteral + '",
        "PubDate":"' + COALESCE(TT.OnSaleDate, TT.PubDate) + '",
        "LanguageDesc":"' + TT.LanguageDesc + '",
        "BisacDesc":"' + TT.BisacDesc + '",
        "AudienceDesc":"' + TT.AudienceDesc + '",
        "Annotation":"' + dbo.udf_RemoveHTMLTags(TT.Annotation) + '",
        "Created":"' + TT.Created + '",
        "Updated":"' + TT.Updated + '",
        "ImprintDesc":"' + TT.ImprintDesc + '",
        "ReadyToVend":"' + TT.ReadyToVendFlag + '",
        "Narrator":"' + TT.Narrator + '",
        "Edition":"' + TT.Edition + '",
        "RunTime":"' + TT.RunTime + '",
        "FileSize":"' + TT.FileSize + '",
        "Illustrator":"' + TT.Illustrator + '"
    }'
FROM ODSFeedStaging AS TT
```