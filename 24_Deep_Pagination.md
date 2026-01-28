# Deep Pagination
 Indexing or fetching all documents at onece is inefficient and slow.

Pagination: 
- Retrieves data in small chunks from large indexes.
- Cost effective and fast search experience.

Methods
- from, size : Suitable for smaller datasets
- search_after: Suitable for larger datasets

### from size method

```json

GET /_search
{
  "from": 5,
  "size": 20,
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```
NOTE : 
Avoid using from and size to page too deeply or request too many results at once. 
- Search requests usually span multiple shards. Each shard must load its requested hits and the hits for any previous pages into memory. 
- For deep pages or large sets of results, these operations can significantly increase memory and CPU usage, resulting in degraded performance or node failures.
- By default, you cannot use from and size to page through more than 10,000 hits. 

### Search After

You can use the `search_after` parameter to retrieve the next page of hits using a set of sort values from the previous page.

Using `search_after` requires multiple search requests with the same `query` and `sort` values. 

```json
PUT planets/_doc
  {
    "id": 1,
    "title": "The Solar System",
    "content": "The Solar System consists of the Sun and the objects that orbit it, including eight planets, their moons, dwarf planets, and countless small bodies like asteroids and comets."
  },
  {
    "id": 2,
    "title": "Black Holes",
    "content": "A black hole is a region of space where the gravitational pull is so strong that nothing, not even light, can escape from it. They are formed when massive stars collapse under their own gravity."
  },
  {
    "id": 3,
    "title": "Galaxies",
    "content": "Galaxies are vast systems that consist of stars, stellar remnants, interstellar gas, dust, and dark matter. The Milky Way is the galaxy that contains our Solar System."
  },
  {
    "id": 4,
    "title": "The Big Bang Theory",
    "content": "The Big Bang Theory is the leading explanation about how the universe began. It suggests that the universe was once in an extremely hot and dense state and has been expanding ever since."
  },
  {
    "id": 5,
    "title": "Exoplanets",
    "content": "Exoplanets, or extrasolar planets, are planets that exist outside our solar system. They vary greatly in size and composition and are often found using methods like the transit method and radial velocity."
  },
  {
    "id": 6,
    "title": "The Life Cycle of Stars",
    "content": "Stars are born from clouds of gas and dust in space. They undergo a life cycle that includes stages such as main sequence, red giant, and, ultimately, either a supernova explosion or a gentle fade into a white dwarf."
  },
  {
    "id": 7,
    "title": "Astrobiology",
    "content": "Astrobiology is the study of the origin, evolution, distribution, and future of life in the universe. It combines elements of biology, chemistry, and planetary science."
  },
  {
    "id": 8,
    "title": "Dark Matter",
    "content": "Dark matter is a type of matter that does not emit light or energy. It cannot be observed directly but is believed to make up about 27% of the universe's total mass and energy."
  },
  {
    "id": 9,
    "title": "The Expanding Universe",
    "content": "The universe has been expanding since the Big Bang. Observations of distant galaxies show that they are moving away from us, which supports the idea of an expanding universe."
  },
  {
    "id": 10,
    "title": "Space Exploration",
    "content": "Space exploration involves the use of space technology to explore outer space. It includes missions to planets, moons, and other celestial bodies, aiming to discover more about the universe."
  }
```


The first step is to run an initial request. The following example sorts the results by id:

```json
GET planets/_search
{
    "size": 1,
    "query": {
        "match": {
            "content": "planets"
        }
    },
    "sort": [
        {"id": "asc"}      
    ]
}

// results
       "_source": {
          "id": 1,
          "title": "The Solar System",
          "content": "The Solar System consists of the Sun and the objects that orbit it, including eight planets, their moons, dwarf planets, and countless small bodies like asteroids and comets."
        },
        "sort": [
          1
        ]
```

To retrieve the next page of results, repeat the request, take the sort values from the last hit, and insert those into the search_after array:

```json
GET planets/_search
{
    "size": 1,
    
    "query": {
        "match": {
            "content": "planets"
        }
    },
    "search_after": [ 1 ], 
    "sort": [
        {"id": "asc"}      
    ]
}
```
Repeat this process by updating the search_after array every time you retrieve a new page of results. 
If a refresh occurs between these requests, the order of your results may change, causing inconsistent results across pages. To prevent this, you can create a point in time (PIT) to preserve the current index state over your searches.

```json
POST /planets/_pit?keep_alive=1m

{
  "id": "9s2wBAEHcGxhbmV0cxZDSHFadWF6YlJpeUJxQ3RMekZjUjZRAAEWcmpxQ3o3MmNRLU84NmJyQUg1Q3p1QQABAAAAAAAAXhgWNVNTMVRFOXNSUWU3ZjZ4cVRYNnZ4dwABFkNIcVp1YXpiUml5QnFDdEx6RmNSNlEAAA==",
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  }
}

```
The search response includes an array of `sort` values for each hit. If you used a PIT, a tiebreaker is included as the last sort values for each hit. This tiebreaker called `_shard_doc` is added automatically on every search requests that use a PIT. The `_shard_doc` value is the combination of the shard index within the PIT and the Lucene’s internal doc ID, it is unique per document and constant within a PIT. 

`_shard_doc` can also be added explicitly

```json
// THe index name is not require for GET 
GET /_search
{
  "size": 1,
  "query": {
    "match" : {
      "content": "planets"
    }
  },
  "pit": {
    "id":  "9s2wBAEHcGxhbmV0cxZDSHFadWF6YlJpeUJxQ3RMekZjUjZRAAEWcmpxQ3o3MmNRLU84NmJyQUg1Q3p1QQABAAAAAAAAXnAWNVNTMVRFOXNSUWU3ZjZ4cVRYNnZ4dwABFkNIcVp1YXpiUml5QnFDdEx6RmNSNlEAAA==",
    "keep_alive": "1m"
  },
  "sort": [ 
    {"id": "asc"}
    // , {"_shard_doc" : "desc" }  // Can be added explicilty
  ]
}

// Results
...

{
  "pit_id": "9s2wBAEHcGxhbmV0cxZDSHFadWF6YlJpeUJxQ3RMekZjUjZRAAEWcmpxQ3o3MmNRLU84NmJyQUg1Q3p1QQABAAAAAAAAXnAWNVNTMVRFOXNSUWU3ZjZ4cVRYNnZ4dwABFkNIcVp1YXpiUml5QnFDdEx6RmNSNlEAAA==",
  .
  .
  .
"sort": [
          1,  
          0  // shard doc
        ]

...
}
```
Fo next page results you can use this `sort` array in the `search_after` parameter  and the new `pit_id` from last last search results
```json
GET /_search
{
  "size": 1,
  "query": {
    "match" : {
      "content": "planets"
    }
  },
  "pit": {
    "id":  "9s2wBAEHcGxhbmV0cxZDSHFadWF6YlJpeUJxQ3RMekZjUjZRAAEWcmpxQ3o3MmNRLU84NmJyQUg1Q3p1QQABAAAAAAAAXnAWNVNTMVRFOXNSUWU3ZjZ4cVRYNnZ4dwABFkNIcVp1YXpiUml5QnFDdEx6RmNSNlEAAA==",
    "keep_alive": "1m"
  },
  "search_after": [1, 0],
  "sort": [ 
    {"id": "asc"} 
  ]
  ,
  "track_total_hits": false   // THis is useful to just focus on results and ignore total value for efficiency purpose
}
```

You can repeat this process to get additional pages of results. If using a PIT, you can extend the PIT’s retention period using the keep_alive parameter of each search request.

When you’re finished, you should delete your PIT.
```json

DELETE /_pit
{
    "id" : "9s2wBAEHcGxhbmV0cxZDSHFadWF6YlJpeUJxQ3RMekZjUjZRAAEWcmpxQ3o3MmNRLU84NmJyQUg1Q3p1QQABAAAAAAAAXnAWNVNTMVRFOXNSUWU3ZjZ4cVRYNnZ4dwABFkNIcVp1YXpiUml5QnFDdEx6RmNSNlEAAA=="
}

```

#### Important
- All PIT search requests add an implicit sort tiebreaker field called `_shard_doc`, which can also be provided explicitly. If you cannot use a PIT, we recommend that you include a tiebreaker field in your sort. This tiebreaker field should contain a unique value for each document. If you don’t include a tiebreaker field, your paged results could miss or duplicate hits.

- Search after requests have optimizations that make them faster when the sort order is _shard_doc and total hits are not tracked. If you want to iterate over all documents regardless of the order, this is the most efficient option
