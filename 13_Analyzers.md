# Back to Analyzers and Analysis
# Stemming / Stop words
### Lets look at a example
```json
POST /test_stemming/_doc
{
    "name" : "Purandeswar",
    "description" : "A 19 year old strong man who loves playing kabbadi"
}

GET /test_stemming/_search
{
  "query": {
    "match": {
      "description": "loved"
    }
  }
}

// Now the total hits = 0, becuase anlayzer stores tokens as it is ( loves != loved)
POST /_analyze
{
  "text" : "A 19 year old strong man who loves playing kabbadi",
  "analyzer": "standard" 
}
```

### For this we can use Stemming 
```json
POST /_analyze
{
  "text" : "A 19 year old strong man who loves playing kabbadi",
  "analyzer": "englsh" 
}
// COMPARE the OUTPUT with standard analyzer
```

### Lets deep dive into ANalyzers

# Built In Analyzer
### Lets look at a example on how to use built in analyzer 
```json
PUT /test_stemming
{
  "mappings": {
    "properties": {
      "description": {
        "type": "text",
        "analyzer": "english"
      }
    }
  }    
}
POST /test_stemming/_doc
{
    "name" : "Purandeswar",
    "description" : "A 19 year old strong man who loves playing kabbadi"
}

GET /test_stemming/_search
{
  "query": {
    "match": {
      "description": "loved"
    }
  }
}
```

# Custom Analyzer
### Example 1
```json
PUT /analyzer_test
{
  "settings": {
    "analysis": {
      "filter": {
        "filter_stemmer" : {
          "type": "stemmer",
          "language": "english"          
        }
      },
      "analyzer": {
        "stemming" : {
          "tokenizer": "standard",
          "filter": ["lowercase", "stemmer"]          
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "description" : {
        "type" : "text",
        "analyzer": "stemming"
      }
    }
  }
}


GET /analyzer_test/_analyze
{
  "analyzer": "stemming",
  "text": "loves loved loving"
}
```


# Creating custom analyzers from Scratch

### Remove HTML tags and convert HTML entities. Follow the steps they are cool and intuitive
```json
POST /_analyze
{
  "char_filter": ["html_strip"],
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}


// Add the `standard` tokenizer
POST /_analyze
{
  "char_filter": ["html_strip"],
  "tokenizer": "standard",
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}

// Add the `lowercase` token filter
POST /_analyze
{
  "char_filter": ["html_strip"],
  "tokenizer": "standard",
  "filter": [
    "lowercase"
  ],
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}

// Add the `stop` token filter
// This removes English stop words by default.
POST /_analyze
{
  "char_filter": ["html_strip"],
  "tokenizer": "standard",
  "filter": [
    "lowercase",
    "stop"
  ],
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}


// Add the `asciifolding` token filter
POST /_analyze
{
  "char_filter": ["html_strip"],
  "tokenizer": "standard",
  "filter": [
    "lowercase",
    "stop",
    "asciifolding"
  ],
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}

// Create a custom analyzer named `my_custom_analyzer`

PUT /analyzer_test
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": ["html_strip"],
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "stop",
            "asciifolding"
          ]
        }
      }
    }
  }
}

// Test the custom analyzer
POST /analyzer_test/_analyze
{
  "analyzer": "my_custom_analyzer", 
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> ascii!"
}
```


# Adding analyzers to existing indices
### Close ->Change -> Open ?( Check Dynamic and Static Setting Conept)
#### Close `analyzer_test` index
```json
POST /analyzer_test/_close 
// index stops writes and reads
```

#### Add new analyzer
```json
PUT /analyzer_test/_settings
{
  "analysis": {
    "analyzer": {
      "my_second_analyzer": {
        "type": "custom",
        "tokenizer": "simple",
        "char_filter": ["html_strip"],
        "filter": [
          "lowercase",
          "stop",
          "asciifolding"
        ]
      }
    }
  }
}
```

#### Open `analyzer_test` index
```json
POST /analyzer_test/_open
```

### Retrieve index settings
```
GET /analyzer_test/_settings
```


# Updating analyzers

#### Add `description` mapping using `my_custom_analyzer`
```json
PUT /analyzer_test/_mapping
{
  "properties": {
    "description": {
      "type": "text",
      "analyzer": "my_custom_analyzer"
    }
  }
}

// Index a test document

POST /analyzer_test/_doc
{
  "description": "Is that Peter's cute-looking dog?"
}
// Use Search for searching as stop word
GET /analyzer_test/_search
{
  "query": {
    "match": {
      "description": {
        "query": "that"
      }
    }
  }
}
```
#### Now we want to Modify the `my_custom_analyzer` by removing the stopwords from filter.
Same procedure as Adding the Analyzer
```json
// Close `analyzer_test` index
POST /analyzer_test/_close

// Update `my_custom_analyzer` (remove `stop` token filter)
PUT /analyzer_test/_settings
{
  "analysis": {
    "analyzer": {
      "my_custom_analyzer": {
        "type": "custom",
        "tokenizer": "standard",
        "char_filter": ["html_strip"],
        "filter": [
          "lowercase",
          "asciifolding"
        ]
      }
    }
  }
}
// Open `analyzer_test` index
POST /analyzer_test/_open

// Retrieve index settings
GET /analyzer_test/_settings
```

#### Reindex documents whenever there is any Change in Analyzer..Some times may cause pain..But you will be fine
```
POST /analyzer_test/_update_by_query?conflicts=proceed
```

