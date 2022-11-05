# Geo-distance aggregation
geo_point フィールドで動作するマルチバケット集計で、概念的には範囲集計と非常によく似た動作をします。ユーザは、原点と距離範囲のバケット群を定義することができます。この集約では、各文書値の原点からの距離を評価し、その範囲に基づいて属するバケットを決定します（文書と原点の間の距離がバケットの距離範囲に含まれる場合、その文書はバケットに属します）。

```bash
PUT /museums
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      }
    }
  }
}

POST /museums/_bulk?refresh
{"index":{"_id":1}}
{"location": "POINT (4.912350 52.374081)", "name": "NEMO Science Museum"}
{"index":{"_id":2}}
{"location": "POINT (4.901618 52.369219)", "name": "Museum Het Rembrandthuis"}
{"index":{"_id":3}}
{"location": "POINT (4.914722 52.371667)", "name": "Nederlands Scheepvaartmuseum"}
{"index":{"_id":4}}
{"location": "POINT (4.405200 51.222900)", "name": "Letterenhuis"}
{"index":{"_id":5}}
{"location": "POINT (2.336389 48.861111)", "name": "Musée du Louvre"}
{"index":{"_id":6}}
{"location": "POINT (2.327000 48.860000)", "name": "Musée d'Orsay"}

POST /museums/_search?size=0
{
  "aggs": {
    "rings_around_amsterdam": {
      "geo_distance": {
        "field": "location",
        "origin": "POINT (4.894 52.3760)",
        "ranges": [
          { "to": 100000 },
          { "from": 100000, "to": 300000 },
          { "from": 300000 }
        ]
      }
    }
  }
}
```

Response:

```json
{
  ...
  "aggregations": {
    "rings_around_amsterdam": {
      "buckets": [
        {
          "key": "*-100000.0",
          "from": 0.0,
          "to": 100000.0,
          "doc_count": 3
        },
        {
          "key": "100000.0-300000.0",
          "from": 100000.0,
          "to": 300000.0,
          "doc_count": 1
        },
        {
          "key": "300000.0-*",
          "from": 300000.0,
          "doc_count": 2
        }
      ]
    }
  }
}
```

指定するフィールドは geo_point 型でなければなりません (これはマッピングの中で明示的にのみ設定可能です)。また、geo_point フィールドの配列を保持することもでき、その場合、集約時にすべてが考慮されます。原点は、geo_point 型がサポートするすべての形式を受け入れることができます。

- オブジェクトの形式。オブジェクト形式: { "lat" : 52.3760, "lon" : 4.894 }. - これは、緯度と経度の値が最も明確であるため、最も安全な形式です。
- 文字列のフォーマット。「52.3760, 4.894" - 最初の数字が緯度、次の数字が経度である。
- 配列形式。[4.894, 52.3760] - GeoJSON 標準に基づくもので、最初の数字が lon、2 番目の数字が lat となります。

デフォルトでは、距離の単位はm（メートル）であるが、mi（マイル）、in（インチ）、yd（ヤード）、km（キロメートル）、cm（センチメートル）、mm（ミリメートル）も受け付けることができる。

```bash
POST /museums/_search?size=0
{
  "aggs": {
    "rings": {
      "geo_distance": {
        "field": "location",
        "origin": "POINT (4.894 52.3760)",
        "unit": "km", ---1
        "ranges": [
          { "to": 100 },
          { "from": 100, "to": 300 },
          { "from": 300 }
        ]
      }
    }
  }
}
```

1. 距離はキロメートル単位で計算されます。

距離の計算モードには、円弧（デフォルト）と平面の2種類があります。円弧計算が最も正確です。平面計算は最も速いですが、最も正確ではありません。plane は、検索対象が「狭い」場合や、地理的に狭い範囲 (~5km) である場合に使用します。距離計算のタイプは、distance_type パラメータで設定することができます。

```bash
POST /museums/_search?size=0
{
  "aggs": {
    "rings": {
      "geo_distance": {
        "field": "location",
        "origin": "POINT (4.894 52.3760)",
        "unit": "km",
        "distance_type": "plane",
        "ranges": [
          { "to": 100 },
          { "from": 100, "to": 300 },
          { "from": 300 }
        ]
      }
    }
  }
}
```

## Keyed Response
keyed フラグを true に設定すると、各バケツに一意な文字列キーが関連付けられ、その範囲が配列ではなくハッシュとして返されます。

```bash
POST /museums/_search?size=0
{
  "aggs": {
    "rings_around_amsterdam": {
      "geo_distance": {
        "field": "location",
        "origin": "POINT (4.894 52.3760)",
        "ranges": [
          { "to": 100000 },
          { "from": 100000, "to": 300000 },
          { "from": 300000 }
        ],
        "keyed": true
      }
    }
  }
}
```

Response:

```json
{
  ...
  "aggregations": {
    "rings_around_amsterdam": {
      "buckets": {
        "*-100000.0": {
          "from": 0.0,
          "to": 100000.0,
          "doc_count": 3
        },
        "100000.0-300000.0": {
          "from": 100000.0,
          "to": 300000.0,
          "doc_count": 1
        },
        "300000.0-*": {
          "from": 300000.0,
          "doc_count": 2
        }
      }
    }
  }
}
```

また、各レンジのキーをカスタマイズすることも可能です。

```bash
POST /museums/_search?size=0
{
  "aggs": {
    "rings_around_amsterdam": {
      "geo_distance": {
        "field": "location",
        "origin": "POINT (4.894 52.3760)",
        "ranges": [
          { "to": 100000, "key": "first_ring" },
          { "from": 100000, "to": 300000, "key": "second_ring" },
          { "from": 300000, "key": "third_ring" }
        ],
        "keyed": true
      }
    }
  }
}
```

Response:

```json
{
  ...
  "aggregations": {
    "rings_around_amsterdam": {
      "buckets": {
        "first_ring": {
          "from": 0.0,
          "to": 100000.0,
          "doc_count": 3
        },
        "second_ring": {
          "from": 100000.0,
          "to": 300000.0,
          "doc_count": 1
        },
        "third_ring": {
          "from": 300000.0,
          "doc_count": 2
        }
      }
    }
  }
}
```
