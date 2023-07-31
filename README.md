# Steps to recreate the issue:

1. `docker-compose up`
1. `docker-compose exec -it solr post -c techproducts example/exampledocs/ipod_video.xml`
1. Send the [boolean query from the JSON Query DSL documentation](https://solr.apache.org/guide/8_0/json-query-dsl.html#nested-boolean-query-example)

```
curl -X POST http://localhost:8983/solr/techproducts/query -d '
{
    "query": {
        "bool": {
            "must": [
                {"lucene": {"df": "name", query: "iPod"}}
            ],
            "must_not": [
                {"frange": {"l": "0", "u": "5", "query": "popularity"}}
            ]
        }
    }, "params": {
        "debug": "true"
    }
}'
```

# Expected behavior

* We get one result
* The parsedquery is:

```
"parsedquery":"+name:ipod -FunctionRangeQuery(ConstantScore(frange(int(popularity)):[0 TO 5]))"
```

# Actual behavior

* We get no results
* The parsedquery is:

```
"parsedquery":"+(DisjunctionMaxQuery((text:bool)) DisjunctionMaxQuery((text:must)) DisjunctionMaxQuery((text:_tt2)) DisjunctionMaxQuery((text:must_not)) DisjunctionMaxQuery((text:_tt6)))"
```

# Workarounds

To get the expected behavior, you can either:

* Set luceneMatchVersion to 7.1.0 on line 38 of conf/solrconfig.xml, or
* Set defType to lucene on line 821 of conf/solrconfig.xml

Then:

1. `docker-compose down -v && docker-compose up`
1. Retry the curl command
