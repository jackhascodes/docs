# Requirements Overview
The following requirements have been identified.
* Titles may change ISBN, which must filter down to inventory, if inventory relates to ISBN as a foreign key.
* Titles must be searchable by a variety of fields
* Results must be relevant to a target library's access to a Title regardless of availability
* Results must be filterable based on availability and a variety of other fields
* Search results must not contain duplications of a Title
* Search results must not contain Titles which have been scoped out
* Search results should contain matches where a Title is accessible for either the target library or their parent
* We should be able to index the data in a way that enables all of this with minimal overhead in returning matches
* We should be able to maintain the index in a way that reduces the number of records requiring an update for any of the
  operations described above. (i.e. no update to a Title's details or Library's availability should require updates to
  more than a single record).
* Libraries may have access to titles from other inventories.
* Library access to other inventories may be paused after a certain quota has been met.

## More to be added when gathered

# Simplified Requirements for external providers (i.e. MSFT)
We have 4.5 million titles and 3000+ (and growing) libraries.

Each title consists of up to 20 searchable/filterable data points.

Each library may have access to a number of the 4.5 million titles (their inventory). Due to licensing considerations, 
some titles may become temporarily unavailable for checkout, but should still be reflected in a library's inventory.

We would like to be able to search for titles that a given library owns. This includes details about whether a library
is currently able to supply that title.

Further, we include the capability for some libraries to have access to other libraries' inventory of titles, which must
be reflected in the search results.

The index of searchable titles must be maintained and expandable in a way that does not require replication of title 
data thousands of times over, as new titles become available, and updates to existing titles occur. This means a brute-
force representation is undesirable, not least for overall performance and storage considerations.

Beyond the scope of the search engine in and of itself, we must consider the development impact of ensuring backward 
compatibility with third party systems currently using our existing search feature (which is exposed via our own API).