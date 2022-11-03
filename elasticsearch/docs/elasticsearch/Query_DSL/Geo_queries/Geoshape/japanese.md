# Geoshape query
geo_shape 型あるいは geo_point 型のいずれかを使用してインデックス化されたドキュメントをフィルタリングします。

geo_shape クエリは、geo_shape または geo_point マッピングと同じインデックスを使用して、指定した空間関係 (交差、内包、内包、非接続のいずれか) を使用してクエリ形状と関連する形状を持つドキュメントを検索します。

このクエリは、形状全体を定義する方法と、他のインデックスにあらかじめ登録されている形状の名前を参照する方法の、2つの方法でクエリ形状を定義することをサポートしています。どちらの形式も、以下に例を挙げて定義します。

## Inline shape definition
geo_point 型と同様に、geo_shape クエリでは GeoJSON を使用して形状を表現します。

場所を geo_shape フィールドとする以下のインデックスがあるとします。

```
PUT /example
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_shape"
      }
    }
  }
}

POST /example/_doc?refresh
{
  "name": "Wind & Wetter, Berlin, Germany",
  "location": {
    "type": "point",
    "coordinates": [ 13.400544, 52.530286 ]
  }
}
```

次のクエリは、Elasticsearchのenvelope GeoJSONエクステンションを使用してポイントを検索します。

```
GET /example/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_shape": {
          "location": {
            "shape": {
              "type": "envelope",
              "coordinates": [ [ 13.0, 53.0 ], [ 14.0, 52.0 ] ]
            },
            "relation": "within"
          }
        }
      }
    }
  }
}
```

上記のクエリは、同様に geo_point フィールドに対してクエリを実行することができる。

```
PUT /example_points
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      }
    }
  }
}

PUT /example_points/_doc/1?refresh
{
  "name": "Wind & Wetter, Berlin, Germany",
  "location": [13.400544, 52.530286]
}
```

同じクエリで、geo_pointフィールドが一致する文書が返されます。

```json
GET /example_points/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_shape": {
          "location": {
            "shape": {
              "type": "envelope",
              "coordinates": [ [ 13.0, 53.0 ], [ 14.0, 52.0 ] ]
            },
            "relation": "intersects"
          }
        }
      }
    }
  }
}
```

```json
{
  "took" : 17,
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
        "_index" : "example_points",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name": "Wind & Wetter, Berlin, Germany",
          "location": [13.400544, 52.530286]
        }
      }
    ]
  }
}
```

## Pre-indexed shape
このクエリは、他のインデックスで既にインデックス化されている図形の使用もサポートしています。これは、あらかじめ定義された図形のリストがあり、毎回座標を指定するのではなく、論理名 (New Zealand など) を使ってリストを参照する場合に特に便利です。このような場合、以下のように指定するだけです。

- id - 事前にインデックスが設定された図形を含むドキュメントのID。
- index - インデックス化された図形が存在するインデックスの名前。デフォルトはshapesです。
- path - インデックス前の形状を含むパスとして指定されたフィールド。デフォルトはshapeです。
- routing - 必要であれば、図形ドキュメントのルーティング。

以下は、事前にインデックスが設定された図形を持つフィルタの使用例です。

```json
PUT /shapes
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_shape"
      }
    }
  }
}

PUT /shapes/_doc/deu
{
  "location": {
    "type": "envelope",
    "coordinates" : [[13.0, 53.0], [14.0, 52.0]]
  }
}

GET /example/_search
{
  "query": {
    "bool": {
      "filter": {
        "geo_shape": {
          "location": {
            "indexed_shape": {
              "index": "shapes",
              "id": "deu",
              "path": "location"
            }
          }
        }
      }
    }
  }
}
```

## Spatial relations
以下は、geo フィールドを検索するときに使用できる空間関係演算子の完全なリストです。

- INTERSECTS - (既定) geo_shape または geo_point フィールドがクエリジオメトリと交差しているすべてのドキュメントを返します。
- DISJOINT - geo_shape または geo_point フィールドがクエリジオメトリと何も共通点がないドキュメントをすべて返します。
- WITHIN - geo_shape または geo_point フィールドがクエリジオメトリの範囲内にあるすべてのドキュメントを返します。線形のジオメトリはサポートされていません。
- CONTAINS - geo_shape または geo_point フィールドにクエリジオメトリが含まれるすべてのドキュメントを返します。

### Ignore unmapped
ignore_unmapped オプションを true に設定すると、 マップされていないフィールドを無視し、このクエリではどのドキュメントにもマッチしなくなります。これは、マッピングが異なる複数のインデックスに対してクエリを実行する際に有用です。false (デフォルト値) に設定すると、フィールドがマップされていない場合に例外をスローします。

## Notes
- geo_shape フィールドにデータが Shape の配列としてインデックスされている場合、その配列は 1 つの Shape として扱われる。このため、以下のリクエストは等価である。

```json
PUT /test/_doc/1
{
  "location": [
    {
      "coordinates": [46.25,20.14],
      "type": "point"
    },
    {
      "coordinates": [47.49,19.04],
      "type": "point"
    }
  ]
}
```

```json
PUT /test/_doc/1
{
  "location":
    {
      "coordinates": [[46.25,20.14],[47.49,19.04]],
      "type": "multipoint"
    }
}
```

geo_shape クエリは、geo_shape フィールドがデフォルトの方向である RIGHT (反時計回り) を使用することを想定しています。ポリゴンの向き」を参照してください。
