# SearchUtil

[[_TOC_]]

A new utility class will be required to replace FASTUtil in order to more easily handle backwards compatibility.

The new utility will take existing method signatures and perform a simple call out to the search service, which will
return a response more in line with the newer approach.

To maintain compatibility, the SearchUtil will return the same objects as FASTUtil, thereby making response encoding the
responsibility of the caller, as it is currently.

Moving forward, the search service will be called directly.

For more information on where these methods are currently being called from, please
see [Touch Points](search-utility/touch-points.md).

## Methods

### Search(libraryID, term, filters, facets, page, limit, orderby)

**New Method.** Variant signature to take granular search requests.

**Params:**
* libraryID int
* filters [Filter](#filter)[]
* facets [Facets](#facet)[]
* limit int
* orderby [OrderBy](#orderby)

Post to search service:

```json
{
  "clientID": "{libraryID}",
  "search": {term},
  "filter": {filters},
  "facets": {facets},
  "page": {page},
  "limit": {limit},
  "orderBy": {orderBy}
}
```
---

### Search(param, limit)

**New Method.** Variant signature to take param values

**Params:**

* [SearchParam](#searchparam) param
* Limit int

Given that all current methods are variations on searching, and set up rules around how param is dealt with, this will
be the primary call out to search service

map param.[Refiners](#refiner)[] to [Facets](#facet)[]

call new Filter(param.Scope, param.Term, "match")

map param.Sort to orderBy

Post to search service:

```json
{
  "clientID": "{param.LibraryID}",
  "search": "",
  "filter": [
    {
      "field": "{param.Scope}",
      "value": "{param.Term}",
      "operator": "match"
    }
  ],
  "facets": [
    {
      mapped
      facets...
    }
  ],
  "page": "{param.CurrentPage}",
  "limit": "{limit}",
  "orderBy": "{orderBy}"
}
```

---

### MapFacets

**New Method**

### MapSort

**New Method**

**Params:**

* sort string

determine "ASC"|"DESC" based on leading character of sort string. `+` = `ASC`, `-` = `DESC`

remove leading character

create new OrderBy(strippedSort, direction)

Returns [OrderBy](#orderby)

### BrowseBySubject

Params:

* string libraryID

Calls search with a group by category operator

Returns: ICollection<[Category](#category)>

Call `Search(param, param.Pagefile)`

---

### SearchBook

**Params:**

* [SearchParam](#searchparam) param

With ACS, scope is equivalent to filter. With no scope provided, it will perform a general search.

Call `Search(param, param.Pagefile)`

Return result.Resultset mapped to ICollection<[BookInfo](#bookinfo)>

**Returns:** ICollection<[BookInfo](#bookinfo)>

---

### GetRefiner

**Params:**

* [SearchParam](#searchparam) param

This method essentially gets the available facets for a search query resultset. Redundant for future use, as ACS returns
facets in response anyway. For backward compatibility though, this should be done.

A call to search service with limit = 0 should return the relevant metadata about the search, including facets.

Call `Search(param, 0)`

Return result.Facets mapped to ICollection<[Refiner](#refiner)>

**Returns:** ICollection<[Refiner](#refiner)>


---

### SearchRecommendationsWithLibraryID

See also FastUtil:[SearchRecommendations](#searchrecommentdations), for which this is a bool wrapper. Little bit
redundant, but this is the public method called, so need to stick with this signature.

Sets up a direct call to FAST, so we can probably just use this as a compatibility wrapper
for [SearchBookWithLibraryID()](#searchbookwithlibraryid)

**Params**

* [SearchParam](#searchparam) param
* ref int totalHits

Call `Search(param, param.Pagefile)`

set totalHits = result.ResultCount

Return result.Resultset mapped to ICollection<[BookInfo](#bookinfo)>

**Returns:** ICollection<[BookInfo](#bookinfo)>

---

### SearchBookWithLibraryID

This is the primary search mechanism. We will take the search params and convert them to search-service calls.
This will require handling conversion of sorting params (e.g. `+field` vs `-field` will need to be converted to
`sortby=field, sortorder=asc|desc`)

**Params:**

* [SearchParam](#searchparam) param
* ref int totalHits

Call `Search(param, param.Pagefile)`

set totalHits = result.ResultCount

Return result.Resultset mapped to ICollection<[BookInfo](#bookinfo)>

**Returns:** ICollection<[BookInfo](#bookinfo)>

---

### GetBookDetail

**Params:**

* string btkey
* string libraryID

Fairly sure this can be dummied to maintain contracts. See [usage](search-utility/touch-points.md#getbookdetail-0)

**Returns:** [BookDetail](#bookdetail)

---

### GetBookDetailbyISBN

Params:

* string isbn
* string libraryID

Fairly sure this can be dummied to maintain contracts. See [usage](search-utility/touch-points.md#getbookdetailbyisbn-0)

Returns: [BookDetail](#bookdetail)

---

### GetBookListByBookIDs

Params:

* string[] btkeyList
* string libraryID

create new SearchParam with value = btkeylist, scope = btkey, library = libraryID

call Search(param, btkeylist.length)

Returns: ICollection<[BookInfo](#bookinfo)>

---

### GetBookListByISBNs

Params:

* string[] isbnList
* string libraryID
* bool inlcludeExcluded

create new SearchParam with value = isbnkeylist, scope = isbn, library = libraryID

call Search(param, isbnlist.length)

Returns: ICollection<[BookInfo](#bookinfo)>


---

### GetOfficialReviews

Params:

* string ISBNID

This doesn't appear to have any callers, so safe to dummy.

Returns: ICollection<[ReviewDetail](#reviewdetail)>



### GetTitlesByCategory

Params:

* string libId
* string category
* string subcat
* string titleFormat
* int page
* int pagesize
* string sort
* ref int totalHits
* string[] additionalFilters
* string subject
* string audience
* string author

this is a wrapper for GetMagicWallTitles, return result of that directly with same params.

Returns: ICollection<[BookInfo](#bookinfo)>

---

### GetMagicWallTitles

Params:

* string libId
* string category
* string subcat
* int page
* int pagesize
* string sort
* ref int totalHits
* Refiner[] filters
* string subject
* string audience
* string author

create [Filter](#filter) array `filters`.

append "eq" [Filter](#filter)s for params.category and params.subcat with operator.

append "match" [Filter](#filter)s for params.subject, params.audience and params.author.

map params.filters to [Facet](#facet) array `facets`

map params.sort to [OrderBy](#orderby) `orderby`

call Search(`params.libID`, "", `filters`, `facets`, `params.page`, `params.pagesize`, `orderby`)

map result.Resultset to BookInfo collection

Returns: ICollection<BookInfo>

---

### GetLibraryRefiner

Params:

* string libId
* string category
* string subject
* string audience
* string author
* Refiner[] filters
* string refinergroupname


create [Filter](#filter) array `filters`.

append "eq" [Filter](#filter)s for params.category and params.subcat with operator.

append "match" [Filter](#filter)s for params.subject, params.audience and params.author.

map params.filters to [Facet](#facet) array `facets`

call Search(`params.libID`, "", `filters`, `facets`, `params.page`, `params.pagesize`, `orderby`)

map result.Facets to Refiner collection
Returns: ICollection<Refiner>

---

### GetLibraryFacet

Params:

* SearchParam sparam
* string facetgroupname

No callers, so may be dummied.

Returns: ICollection<Refiner>


---

### GetTitleInfo

Params:

* SearchParam sparam
* ref int totalHits

call Search(sparam, sparam.Pagefile)

map result.Resultset to BookeInfo collection

Returns: ICollection<BookInfo>


---

## Data Types

### SearchRequest

| Field    | Type    | Notes             |
|----------|---------|-------------------|
| clientID | int     | {param.LibraryID} |
| search   | string  |                   |
| filter   | Filter  |                   |
| facets   | Facet[] |                   |
| page     | int     |                   |
| limit    | int     |                   |
| orderBy  | OrderBy |                   |

### BookInfo

| Field               | Type                | Notes |
|---------------------|---------------------|-------|
| ExtensionData       | ExtensionDataObject |       |
| Audience            | string              |       |
| AudienceCode        | string              |       |
| Author              | string              |       |
| AvailLibrary        | string              |       |
| AxisAttribute       | string              |       |
| BookID              | string              |       |
| CheckoutPatrons     | string              |       |
| CreatedDate         | string              |       |
| Edition             | string              |       |
| Format              | string              |       |
| FormatType          | string              |       |
| Genre               | string              |       |
| GradeLevel          | string              |       |
| HasCirculationLimit | bool                |       |
| HoldPatrons         | string              |       |
| HoldQueueSize       | int                 |       |
| ISBN                | string              |       |
| ImgUrl              | string              |       |
| IsExcluded          | bool                |       |
| IsOwnedByLibrary    | bool                |       |
| JacketImageInd      | bool                |       |
| Language            | string              |       |
| LicenseUpdatedDate  | string              |       |
| Narrator            | string              |       |
| OmniReadyToVendFlag | bool                |       |
| OnOrderQuantity     | int                 |       |
| PublicationDate     | string              |       |
| Publisher           | string              |       |
| PurchaseOption      | string              |       |
| Rating              | float               |       |
| ReadyToVend         | string              |       |
| ReservePatrons      | string              |       |
| RunTime             | string              |       |
| SampleAvailable     | bool                |       |
| Series              | string              |       |
| ShortTitle          | string              |       |
| Status              | string              |       |
| SubTitle            | string              |       |
| Subject             | string              |       |
| Synopsis            | string              |       |
| Title               | string              |       |
| TotalCopies         | int                 |       |
| UpdatedDate         | string              |       |

### BookDetail

| Field          | Type           | Notes |
|----------------|----------------|-------|
| Excerpt        | string         |       |
| OfficialReview | ReviewDetail[] |       |
| TOC            | string         |       |

### Category

| Field          | Type                | Notes |
|----------------|---------------------|-------|
| ExtensionData  | ExtensionDataObject |       |
| BookCount      | int                 |       |
| CategoryName   | string              |       |
| ParentCategory | string              |       |
| Value          | string              |       |

### Refiner

| Field             | Type                | Notes                                                 |
|-------------------|---------------------|-------------------------------------------------------|
| ExtensionData     | ExtensionDataObject |                                                       |
| BookCount         | int                 |                                                       |
| DisplayName       | string              | (see below: [Available Refiners](#available-refiners) |
| GroupDisplayName  | string              |                                                       |
| GroupRefinerValue | string              |                                                       |
| Value             | string              |                                                       |

#### Available Refiners

* bisacsdescnavigator
* languagenavigator
* audiencenavigator
* includedformatnavigator
* devicesnavigator
* formattypenavigator

### ReviewDetail

| Field         | Type                | Notes |
|---------------|---------------------|-------|
| ExtensionData | ExtensionDataObject |       |
| Id            | int                 |       |
| Issue         | string              |       |
| Publication   | string              |       |
| Review        | string              |       |
| Supplier      | string              |       |

### SearchParam

| Field         | Type                  | Notes                        |
|---------------|-----------------------|------------------------------|
| ExtensionData | ExtensionDataObject   |                              |
| CurrentPage   | int                   |                              |
| LibraryID     | string                |                              |
| ListLibraryID | string                |                              |
| Pagefile      | int                   |                              |
| Refiner       | [Refiner](#refiner)[] |                              |
| Scope         | string                | (see below: [Scope](#scope)) |
| Sort          | string                | (see below: [Sorts](#sorts)) |
| Term          | string                |                              |


### Filter

| Field     | Type   | Notes |
|-----------|--------|-------|
| FieldName | string |       |
| Value     | string |       |
| Operator  | string |       |

### Facet

| Field     | Type     | Notes |
|-----------|----------|-------|
| FacetName | string   |       |
| Values    | string[] |       |

### OrderBy

| Field          | Type   | Notes |
|----------------|--------|-------|
| orderField     | string |       |
| orderDirection | string |       |

#### Scope:

One of:

* author
* authorAndSeries
* bisac2desc
* bisacscode
* book
* btkey (associated term may be # sep list)
* else
* format
* genre
* isbn (associated term may be # sep list)
* isbn13 (associated term may be # sep list)
* isownedbylibrary
* keywords
* recommendations
* responsibleparties
* series
* series
* subject
* title

#### Sorts:

* relevancy
* popularity
* score
* pubdate
* createddate
* returneddate

