This document serves to show where search utility will be replacing fast-util across various codebases.

[[_TOC_]]

### BrowseBySubject (1)

Current Callers:

* VendorAPI:5501 CommonBrowseBySubject()

      browseBySubjectInfo = vendorAPI.BrowseBySubject(parentSubject, sortOrder, extractedLibraryId, collection);

---

### SearchBook (2)

Current Callers:

* FASTUtilService:156 GetRefiner()

      results = SearchBook(param);

* FASTUtilService:209 SearchBookWithLibraryID()

      results = SearchBook(param);

That means this method is only used in the utility itself.

---

### GetRefiner (5)
Current Callers:

* Axis360Admin/Controllers/ListTitlesController:61 SearchRefiner()
* Axis360Admin/Controllers/DiscoveryBaseController:170 SearchRefiner()
* Axis360Admin/Controllers/EngageController:534 SearchRefiner()
* Axis360Admin/Controllers/StaffPickController:192 SearchRefiner()

All call it with:

    var refiner = FASTUtilClient.GetRefiner(param);

All use the same GetParam defaults.

Common/BT.Omne.Axis360.Utility/SearchUtility:330 GetSearchRefiner()

    return fast.GetRefiner(param);

---

### SearchRecommendationsWithLibraryID (2)

Current Callers:
Axis360Services/LIUDWEBAPI/BT.Omne.LIUDWebApi/Controllers/PatronController.cs:222 Recommendations()

    Axis360.Utility.FASTUTIL.SearchRecommendationsWithLibraryIDRequest request = new Axis360.Utility.FASTUTIL.SearchRecommendationsWithLibraryIDRequest(param, 10);
    var response = await _fastClient.Value.SearchRecommendationsWithLibraryIDAsync(request);

---

### SearchBookWithLibraryID (19)
Current Callers:

* Source/BT.Omne/VendorAPI/BT.Omne.VendorAPI/DataContracts/Assignment.cs:203 GetBooks()

  Searches by list of BTKeys param: (term=BTKeyList, scope="btkey")

      var param = GetParam(libraryId, pageSize, currentPage, bookIDs.DefaultTo("xxxxxxxxxx"), "btkey", sort);
      books = FASTUtilClient.SearchBookWithLibraryID(param, ref totalHits);


* Source/BT.Omne/DesktopSite/Axis360Responsive.BLL/Application/ListManagement.cs: 802 GetRecommendedTitles(libID, term, scope)

  If no search term is supplied, term is dummied (`//Set term to explicit dummy value to make sure search will return nothing`)

  sets sorting to descending pubdate.

      var response = await _fastClient.Value.SearchBookWithLibraryIDAsync(request);


* Source/BT.Omne/Common/BT.Omne.Axis360.Utility/SearchUtility.cs: 76 Search(searchTerm, searchBy, libraryId, filter, sortField, sortOrder = 1,
  pageNum = 1, pageSize = 10, includeRecommendable = false, recommendationCollection = null,
  includeAlwaysAvailable = false, alwaysAvailableCollectionId = null, excludeInventoryTitleFromRecommendationCollection = false)

  Called if `includeRecommendable` is true

  Sets param LibraryId to recommendation collection Id

      var listSearchRecommended = fast.SearchBookWithLibraryID(param, ref hitCount);

* Source/BT.Omne/Common/BT.Omne.Axis360.Utility/SearchUtility.cs:111 Search()

  Called if `includeAlwaysAvailable` is true

  Sets param LibraryId to collectionId

      var listSearchAlwaysAvilable = fast.SearchBookWithLibraryID(param, ref hitCount);

* Source/BT.Omne/Common/BT.Omne.Axis360.Utility/SearchUtility.cs:116 Search()

      var listSearch = fast.SearchBookWithLibraryID(param, ref hitCount);

* Source/BT.Omne/Axis360Admin/Models/BookListModel.cs:189 FilterTitleListThroughFAST()
* Source/BT.Omne/Axis360Admin/Models/BookListModel.cs:173 FilterTitleListThroughFAST()

      listSearch = fast.SearchBookWithLibraryID(param, ref TotalHits);

* Source/BT.Omne/Axis360Admin/Controllers/TransferContentController.cs

      books = GetFastSearchBookWithLibraryId(param, out totalHits);

* Source/BT.Omne/Axis360Admin/Controllers/StaffPickController.cs:55 Search()

      books = GetFastSearchBookWithLibraryId(param, out totalHits);
* Source/BT.Omne/Axis360Admin/Controllers/ScopingController.cs:474

      var searchResults = FASTUtilClient.SearchBookWithLibraryID(param, ref totalHits);

* Source/BT.Omne/Axis360Admin/Controllers/ReservationsController.cs:733 Search()

      BookInfo[] searchResults = FASTUtilClient.SearchBookWithLibraryID(param, ref totalHits);
* Source/BT.Omne/Axis360Admin/Controllers/ManageHoldQueueController.cs Search()

      var books = GetFastSearchBookWithLibraryId(param, out totalHits);
* Source/BT.Omne/Axis360Admin/Controllers/ListTitlesController.cs:432 AddAllPages()

      var searchResults = FASTUtilClient.SearchBookWithLibraryID(param, ref totalHits);
* Source/BT.Omne/Axis360Admin/Controllers/ListTitlesController.cs:151 Search()

      var searchResults = FASTUtilClient.SearchBookWithLibraryID(param, ref totalHits);
* Source/BT.Omne/Axis360Admin/Controllers/ListTitlesController.cs:117 Search()

      var searchResults = FASTUtilClient.SearchBookWithLibraryID(param, ref totalHits);
* Source/BT.Omne/Axis360Admin/Controllers/ListTitlesController.cs:96: Search()

      var searchResults = FASTUtilClient.SearchBookWithLibraryID(param, ref totalHits);
* Source/BT.Omne/Axis360Admin/Controllers/EngageController.cs:186 TitleSearch()

      var searchResults = FASTUtilClient.SearchBookWithLibraryID(param, ref totalHits)
* Source/BT.Omne/Axis360Admin/Controllers/DiscoveryBaseController.cs:214 GetFastSearchBookWithLibraryId()

      var searchResults = FASTUtilClient.SearchBookWithLibraryID(param, ref totalHits);

* Source/BT.Omne/Axis360Admin/Controllers/ManageTitlesController.cs:74 Search()

      var searchResults = FASTUtilClient.SearchBookWithLibraryID(param, ref totalHits);

---

### GetBookDetail (0)

This doesn't appear to have any callers. There is a comparable method in LIUDUtil which may have replaced it?

---

### GetBookDetailbyISBN (0)

This doesn't appear to have any callers.

---

### GetBookListByBookIDs (1)

Callers:

Common/BT.Omne.Axis360.Utility/SearchUtility:80 Search()

Used to crosscheck recommendation results against library.

There is an opportunity to reduce waste here, since we can use the new search service to do this automatically.

    var listSearch = fast.GetBookListByBookIDs(listSearchRecommended.Select(t => t.BookID).ToArray(), libraryId);


---

### GetBookListByISBNs (2)

Callers:

Axis360Admin/Controllers/ListTitlesController:455 AddISBNList()

    var IsbnResults = FASTUtilClient.GetBookListByISBNs(inputISBN, CurrentLibrary.LibraryId, false);

Axis360Admin/Controllers/ScopingController:571 AddISBNView()

    var IsbnResults = FASTUtilClient.GetBookListByISBNs(inputISBN, CurrentLibrary.LibraryId, true);


---

### GetOfficialReviews (0)

This doesn't appear to have any callers.

### GetTitlesByCategory (2)
Callers:

Axis360Admin/Models/BookListModel:319 GetMagicWallBookModel()

    BookInfo[] lstItemDetail = fastUtil.GetTitlesByCategory(lib.LibraryId, categoryObj.Name, subcat, titleFormat,
        page, pageSize, sort, ref hitCount, additionalFilters, subject, audience, author);


ReservationsController:466 GetTitleCarousel()

    List<BookInfo> books = FASTUtilClient.GetTitlesByCategory(CurrentLibrary.LibraryId, "Featured", "Most-Popular", null, 0, 24, null, ref hitCount, null, null, null, null).ToList();



---

### GetMagicWallTitles (2)
Callers:

ListTitlesController:221 Search()

    BookInfo[] lstItemDetail = FASTUtilClient.GetMagicWallTitles(model.LibraryID, category, subcat, param.CurrentPage, param.Pagefile, null, ref hitCount, refinerList.ToArray(), joinedbisacString, audienceString, null);

VendorApiUtil:4321 CreateCatagoryObject()


    lstItemDetails = fastUTIL.GetMagicWallTitles(libraryID, catagory.ToLower(), "",
        pageNumber, pageSize, searchSortWithOrder, ref hitCnt, refinerList.ToArray(), 
        subject, audience, author).ToList();

---

### GetLibraryRefiner (2)
Current Callers:

Axis360Admin/Controllers/CategoryController:86 Filterresults()

    List<Refiner> alllanguages = new List<Refiner>(FASTUtilClient.GetLibraryRefiner(CurrentLibrary.LibraryId, null,
    subjectstr, audiencestr, null, null, "languagenavigator"));

DesktopSite/Axis360Responsive/Application/ListManagement.cs:756 GetAllLanguages()
    
    return SearchUtility.fast.GetLibraryRefiner(libraryId, null, null, null, null, null, "languagenavigator").ToList();


---

### GetLibraryFacet (0)

No direct callers

---

### GetTitleInfo (1)

Callers:
A number of places in VendorAPI call this, but untimately fall back to...

vendorAPIUtil:8911

    var bookInfoArray = fastUtil.GetTitleInfo(param, ref TotalHits);