# Geo-Line Aggregation
geo_line 集約は、バケツ内のすべての geo_point 値を、選択したソートフィールドで並べた LineString に集約するものです。このソートは、たとえば日付フィールドにすることができます。返されるバケツは、ラインのジオメトリを表す有効な GeoJSON Feature です。

```bash
PUT test
{
    "mappings": {
        "dynamic": "strict",
        "_source": {
            "enabled": false
        },
        "properties": {
            "my_location": {
                "type": "geo_point"
            },
            "group": {
                "type": "keyword"
            },
            "@timestamp": {
                "type": "date"
            }
        }
    }
}

POST /test/_bulk?refresh
{"index": {}}
{"my_location": {"lat":37.3450570, "lon": -122.0499820}, "@timestamp": "2013-09-06T16:00:36"}
{"index": {}}
{"my_location": {"lat": 37.3451320, "lon": -122.0499820}, "@timestamp": "2013-09-06T16:00:37Z"}
{"index": {}}
{"my_location": {"lat": 37.349283, "lon": -122.0505010}, "@timestamp": "2013-09-06T16:00:37Z"}

POST /test/_search?filter_path=aggregations
{
  "aggs": {
    "line": {
      "geo_line": {
        "point": {"field": "my_location"},
        "sort": {"field": "@timestamp"}
      }
    }
  }
}
```

Which returns:

```json
{
  "aggregations": {
    "line": {
      "type" : "Feature",
      "geometry" : {
        "type" : "LineString",
        "coordinates" : [
          [
            -122.049982,
            37.345057
          ],
          [
            -122.050501,
            37.349283
          ],
          [
            -122.049982,
            37.345132
          ]
        ]
      },
      "properties" : {
        "complete" : true
      }
    }
  }
}
```

## Options
### point: (Required)
このオプションは、geo_point フィールドの名前を指定する。

my_location をポイントフィールドとして設定する使用例。

```json
"point": {
  "field": "my_location"
}
```

### sort: (Required)
このオプションは、ポイントを並べるためのソートキーとして使用する数値フィールドの名前を指定します。

使用例 @timestampをソートキーとして設定する。

```json
"point": {
  "field": "@timestamp"
}
```

### include_sort: (Optional, boolean, default: false)
このオプションは、true の場合、その機能のプロパティにソート値の配列を追加します。

### sort_order: (Optional, string, default: "ASC")
このオプションは、2つの値のうちの1つを受け付ける。"ASC"、"DESC"。

ASC "を指定するとソートキーで昇順に、"DESC "を指定すると降順にソートされます。

### size: (Optional, integer, default: 10000)
集計で表現される行の最大長を指定する。有効なサイズは1～10000です。
