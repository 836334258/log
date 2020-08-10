- Query DSL

  - Compound queries

    - Bool query

      - must
      - should
      - Must_not
      - filter

      ```json
      {
        "query": {
          "bool" : {
            "must" : {
              "term" : { "user" : "kimchy" }
            },
            "filter": {
              "term" : { "tag" : "tech" }
            },
            "must_not" : {
              "range" : {
                "age" : { "gte" : 10, "lte" : 20 }
              }
            },
            "should" : [
              { "term" : { "tag" : "wow" } },
              { "term" : { "tag" : "elasticsearch" } }
            ],
            "minimum_should_match" : 1,
            "boost" : 1.0
          }
        }
      }
      ```

    - full text query

      - intervals query
      - match query
      - Match_bool_prefix query
      - Match_phrase query
      - Match_phrase_prefix query
      - Multi_match query
      - Common_terms query
      - query_string query
      - Simple_query_string query