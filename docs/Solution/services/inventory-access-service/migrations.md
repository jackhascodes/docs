Using Axis Poller

Export vendors:
```sql

INSERT INTO EventQueue (Topic, Type, Message)
SELECT 'availability.access.updated.success',
        'delta',
        '{"ClientID":"' + LibraryID + '","InventoryRef":"' + VendorID + ',"Blocked":0}'
FROM LibraryAccessProperty;
```
Export Libraries:
```sql
INSERT INTO EventQueue (Topic, Type, Message)
SELECT 'availability.access.updated.success',
       'delta',
       '{"ClientID":"' + LibraryID + '","InventoryRef":"' + LibraryID + ',"Blocked":0}'
FROM Library;
```