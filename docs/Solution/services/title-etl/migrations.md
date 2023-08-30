# Title ETL: Migrations

How initial data will be populated.
Using Axis Poller:
```sql
INSERT INTO EventQueue (Topic, Type, Message)
SELECT 'omni.updated.success',
       'document',
       '{
        "BTKey":"' + BTKey + '",
        "ISBN":"' + ISBN + '",
        "Title":"' + Title + '",
        "SubTitle":"' + SubTitle + '",
        "ShortTitle ":"' + ShortTitle + '",
        "Series":"' + Series + '",
        "Author":"' + Author + '",
        "SupplierDesc":"' + SupplierDesc + '",
        "ESupplierLiteral":"' + ESupplierLiteral + '",
        "EFormatLiteral":"' + EFormatLiteral + '",
        "PubDate":"' + PubDate + '",
        "LanguageDesc":"' + LanguageDesc + '",
        "BisacDesc":"' + BisacDesc + '",
        "AudienceDesc":"' + AudienceDesc + '",
        "Annotation":"' + Annotation + '",
        "Created":"' + Created + '",
        "Updated":"' + Updated + '",
        "ImprintDesc":"' + ImprintDesc + '",
        "ReadyToVend":"' + ReadyToVendFlag + '",
        "Narrator":"' + Narrator + '",
        "Edition":"' + Edition + '",
        "RunTime":"' + RunTime + '",
        "FileSize":"' + FileSize + '",
        "Illustrator":"' + Illustrator + '"
    }'
FROM OMNI;
```