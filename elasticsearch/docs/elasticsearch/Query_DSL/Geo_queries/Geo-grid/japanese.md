# Geo-grid query
GeoGrid 集約からグリッドセルと交差する geo_point および geo_shape 値にマッチする。

このクエリは、バケットのキーを指定することで、ジオグリッド集計のバケット内にあるドキュメントにマッチするように設計されています。geohash と geotile グリッドの場合、このクエリは geo_point と geo_shape フィールドに対して使用することができる。geo_hex グリッドでは、geo_point フィールドにのみ使用できる。

## Example
次のような文書がインデックス化されていると仮定する。

```
PUT /my_locations
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      }
    }
  }
}

PUT /my_locations/_doc/1?refresh
{
  "location" : "POINT(4.912350 52.374081)",
  "city": "Amsterdam",
  "name": "NEMO Science Museum"
}

PUT /my_locations/_doc/2?refresh
{
  "location" : "POINT(4.405200 51.222900)",
  "city": "Antwerp",
  "name": "Letterenhuis"
}

PUT /my_locations/_doc/3?refresh
{
  "location" : "POINT(2.336389 48.861111)",
  "city": "Paris",
  "name": "Musée du Louvre"
}
```

# geohash grid
geohash_grid 集約を使用すると、ジオハッシュの値に応じて文書をグループ化することができます。

```
GET /my_locations/_search
{
  "size" : 0,
  "aggs" : {
     "grouped" : {
        "geohash_grid" : {
           "field" : "location",
           "precision" : 2
        }
     }
  }
}
```

```
{
  "took" : 10,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "grouped" : {
      "buckets" : [
        {
          "key" : "u1",
          "doc_count" : 2
        },
        {
          "key" : "u0",
          "doc_count" : 1
        }
      ]
    }
  }
}
```

そのバケットのキーを使って、以下の構文でgeo_gridクエリを実行すれば、そのバケット上の文書を抽出することができます。

```
GET /my_locations/_search
{
  "query": {
    "geo_grid" :{
      "location" : {
        "geohash" : "u0"
      }
    }
  }
}
```

```
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "my_locations",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "location" : "POINT(2.336389 48.861111)",
          "city" : "Paris",
          "name" : "Musée du Louvre"
        }
      }
    ]
  }
}
```

# geotile grid
geotile_grid 集約を使用すると、ジオタイルの値によって文書をグループ化することができる。

```
GET /my_locations/_search
{
  "size" : 0,
  "aggs" : {
     "grouped" : {
        "geotile_grid" : {
           "field" : "location",
           "precision" : 6
        }
     }
  }
}
```

```
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "grouped" : {
      "buckets" : [
        {
          "key" : "6/32/21",
          "doc_count" : 2
        },
        {
          "key" : "6/32/22",
          "doc_count" : 1
        }
      ]
    }
  }
}
```

そのバケットのキーを使って、以下の構文でgeo_gridクエリを実行すれば、そのバケット上の文書を抽出することができます。

```
GET /my_locations/_search
{
  "query": {
    "geo_grid" :{
      "location" : {
        "geotile" : "6/32/22"
      }
    }
  }
}
```

```
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "my_locations",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "location" : "POINT(2.336389 48.861111)",
          "city" : "Paris",
          "name" : "Musée du Louvre"
        }
      }
    ]
  }
}
```

# geohex grid
geohex_grid 集約を使用すると、ジオヘックスの値によって文書をグループ化することができる。

```
GET /my_locations/_search
{
  "size" : 0,
  "aggs" : {
     "grouped" : {
        "geohex_grid" : {
           "field" : "location",
           "precision" : 1
        }
     }
  }
}
```

```
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "grouped" : {
      "buckets" : [
        {
          "key" : "81197ffffffffff",
          "doc_count" : 2
        },
        {
          "key" : "811fbffffffffff",
          "doc_count" : 1
        }
      ]
    }
  }
}
```

そのバケットのキーを使って、以下の構文でgeo_gridクエリを実行すれば、そのバケット上の文書を抽出することができます。

```
GET /my_locations/_search
{
  "query": {
    "geo_grid" :{
      "location" : {
        "geohex" : "811fbffffffffff"
      }
    }
  }
}
```

```
{
  "took" : 26,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "my_locations",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "location" : "POINT(2.336389 48.861111)",
          "city" : "Paris",
          "name" : "Musée du Louvre"
        }
      }
    ]
  }
}
```
