# Title ETL: Data Providers

The Title ETL service will maintain its own persistence repository. To enable efficient and extensible storage of
things like BTKey aliasing of products, this should be a relational database.

Maybe we can leverage the TitleInfo mongo db for persistence, since it's covering effectively everything we want.

## Repository Schema


### Title

| Field                | Type     | Description                                                                                  |
|----------------------|----------|----------------------------------------------------------------------------------------------|
| ID                   | int      | a primary key                                                                                |
| ProductKey           | int      | reference to the title this title item refers to                                             |
| AudienceCode         | varchar  |                                                                                              |
| DateAdded            | int      | an integer encoded date (format `yyyymmdd`) defining when the title was purchased (0 = null) |
| UpdatedAt            | int      | a timestamp indicating when this record was last updated                                     |
| Series               | varchar  |                                                                                              |
| Author               | varchar  |                                                                                              |
| Narrator             | varchar  |
| SupplierDesc         | varchar  |                                                                                              |
| ImprintDesc          | varchar  |                                                                                              |
| ESupplierLiteral     | varchar  |                                                                                              |
| EFormatLiteral       | varchar  |                                                                                              |
| PubDate              | int      | a yyyymmdd formatted date as an integer                                                      |
| LanguageDesc         | varchar  |                                                                                              |
| BISAC                | varchar  |                                                                                              |
| Genre                | varchar  |                                                                                              |                                                                                            
| AudienceDesc         | varchar  |                                                                                              |
| UpdateDate           | int      | a yyyymmdd formatted date as an integer                                                      |
| Annotation           | varchar  |                                                                                              |
| AvgRating            | int      |                                                                                              |
| ReadyToVend          | tinyint  |                                                                                              |
| PurchaseOption       | varchar  |                                                                                              |
| Edition              | varchar  |                                                                                              |
| JacketImageInd       | tinyint  |                                                                                              |
| SubTitle             | varchar  |                                                                                              |
| ShortTitle           | varchar  |                                                                                              |
| AudioSampleAvailable | varchar  |                                                                                              |
| AxisAttribute        | varchar  |                                                                                              |
| RunTime              | int      |                                                                                              |
| PageCount            | int      |                                                                                              |
| Lexile               | varchar  |                                                                                              |
| PopularityScore      | int      |                                                                                              |
| DeletedDate          | int      | a yyyymmdd formatted date as an integer. 0 == not deleted.                                   |
### BT Alias

| Field       | Type    | Notes                    |
|-------------|---------|--------------------------|
| InventoryID | INT     | Foreign Key to Inventory |
| BTKey       | VARCHAR |                          |
| Format      | VARCHAR |                          |

## Required Methods

* Get(int ProductID, int LibraryID): [Inventory](domain.md#inventory)
* GetByProductID(int ProductID): [Inventory](domain.md#inventory)
* Save([Inventory](domain.md#inventory) inventory): bool