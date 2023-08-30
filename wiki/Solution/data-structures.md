# Data Structures

This document outlines the various data structures which will be used to store and communicate title availability
within the context of Search.

## Search Index

### Document Structure

### Regions

Talk about the concept of regions here

## Ingestion

Talk about the various intermediate records and tables here, and how they relate to the final index

### inventory

| name      | type         | 
|-----------|--------------|
| ID        | int          |
| LibKey    | int          |
| Regions   | int[]        |
| Available | enum{-1,0,1} |
| Audience  | string       |
| DateAdded | int          |

_A note on storage:_
It makes more sense for this to be represented in a relational database from the PoV of managing regions.
No need to parse arrays to slice and so on, just join select on relevant regions.

### title


## Search Queries

Talk about request transforms here

## Search Results

Talk about response transforms here