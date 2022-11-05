# Geohash grid aggregation
geo_point と geo_shape の値を、グリッドを表すバケットにグループ化するマルチバケット集計です。結果として得られるグリッドは疎にすることができ、一致するデータを持つセルのみを含む。各セルは、ユーザーが定義できる精度のジオハッシュを使用してラベル付けされます。

- 高精度のジオハッシュは文字列の長さが長く、狭い範囲をカバーするセルを表します。
- 低精度のジオハッシュは文字列の長さが短く、それぞれが広い面積を持つセルを表します。

この集計に使用されるジオハッシュは、1～12の間で精度を選択することができます。

**WARNING**
長さ12の最高精度のジオハッシュは、1平方メートル以下の面積のセルを生成するため、高精度のリクエストはRAMと結果サイズの点で非常にコストがかかる可能性があります。以下の例では、高精度の要求をする前に、まず小さい地理的範囲に集計をフィルタリングする方法について説明しています。


geohash_grid を使用して集計できるのは、明示的にマップされた geo_point または geo_shape フィールドに限られます。geo_point フィールドに配列が含まれている場合、geohash_grid はすべての配列値を集計します。

## Simple low-precision request

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
  "aggregations": {
    "large-grid": {
      "geohash_grid": {
        "field": "location",
        "precision": 3
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
  "large-grid": {
    "buckets": [
      {
        "key": "u17",
        "doc_count": 3
      },
      {
        "key": "u09",
        "doc_count": 2
      },
      {
        "key": "u15",
        "doc_count": 1
      }
    ]
  }
}
}
```

## High-precision requests
詳細なバケットを要求する場合（通常、「拡大された」地図を表示するため）、対象領域を絞り込むために geo_bounding_box のようなフィルタを適用する必要があります。

```bash
POST /museums/_search?size=0
{
  "aggregations": {
    "zoomed-in": {
      "filter": {
        "geo_bounding_box": {
          "location": {
            "top_left": "POINT (4.9 52.4)",
            "bottom_right": "POINT (5.0 52.3)"
          }
        }
      },
      "aggregations": {
        "zoom1": {
          "geohash_grid": {
            "field": "location",
            "precision": 8
          }
        }
      }
    }
  }
}
```

geohash_grid の集約で返されるジオハッシュは、ズームインにも利用することができます。前の例で返された最初のジオハッシュ u17 にズームインするには、 top_left と bottom_right corner の両方を指定する必要があります。

```bash
POST /museums/_search?size=0
{
  "aggregations": {
    "zoomed-in": {
      "filter": {
        "geo_bounding_box": {
          "location": {
            "top_left": "u17",
            "bottom_right": "u17"
          }
        }
      },
      "aggregations": {
        "zoom1": {
          "geohash_grid": {
            "field": "location",
            "precision": 8
          }
        }
      }
    }
  }
}
```

```json
{
  ...
  "aggregations": {
    "zoomed-in": {
      "doc_count": 3,
      "zoom1": {
        "buckets": [
          {
            "key": "u173zy3j",
            "doc_count": 1
          },
          {
            "key": "u173zvfz",
            "doc_count": 1
          },
          {
            "key": "u173zt90",
            "doc_count": 1
          }
        ]
      }
    }
  }
}
```

ジオハッシュをサポートしていないシステムで「ズームイン」する場合、バケットキーをジオハッシュライブラリのいずれかを使ってバウンディングボックスに変換する必要があります。例えば、javascript では、node-geohash ライブラリを使用することができます。

```javascript
var geohash = require('ngeohash');

// bbox will contain [ 52.03125, 4.21875, 53.4375, 5.625 ]
//                   [   minlat,  minlon,  maxlat, maxlon]
var bbox = geohash.decode_bbox('u17');
```

## Requests with additional bounding box filtering
geohash_grid 集約は、オプションの bounds パラメータをサポートしており、考慮するセルを指定した境界と交差するものに制限します。bounds パラメータは、Geo Bounding Box クエリで指定された境界と同じフォーマットで境界ボックスを受け取ります。このバウンディングボックスは、集計前にポイントをフィルタリングする追加の geo_bounding_box クエリとともに、またはなしで使用することができます。これは独立したバウンディングボックスで、集約のコンテキストで定義された追加の geo_bounding_box クエリと交差したり、等しかったり、不連続になったりします。

```bash
POST /museums/_search?size=0
{
  "aggregations": {
    "tiles-in-bounds": {
      "geohash_grid": {
        "field": "location",
        "precision": 8,
        "bounds": {
          "top_left": "POINT (4.21875 53.4375)",
          "bottom_right": "POINT (5.625 52.03125)"
        }
      }
    }
  }
}
```

```json
{
  ...
  "aggregations": {
    "tiles-in-bounds": {
      "buckets": [
        {
          "key": "u173zy3j",
          "doc_count": 1
        },
        {
          "key": "u173zvfz",
          "doc_count": 1
        },
        {
          "key": "u173zt90",
          "doc_count": 1
        }
      ]
    }
  }
}
```

## Cell dimensions at the equator
下の表は、様々な文字列の長さのジオハッシュでカバーされるセルの寸法をメートル法で示したものです。セルの寸法は緯度によって異なるため、この表は赤道での最悪のケースを想定しています。

### Aggregating geo_shape fields
Geoshapeフィールドの集計は、1つの形状が複数のタイルでカウントされることを除いて、ポイントの場合と同じように動作します。形状は、その形状のいずれかの部分がそのタイルと交差している場合、マッチング値のカウントに貢献します。以下は、これを示す画像である。

## Options
- field
  - 必須。GeoPoints でインデックス化されたフィールドの名前。
- precision
  - 任意。結果のセル/バケットを定義するために使用されるジオハッシュの文字列の長さ。デフォルトは 5。精度は、上記の整数の精度レベルで定義することもできます。1,12]以外の値は却下される。あるいは、精度レベルは、"1km "や "10m "のような距離の尺度から近似させることも可能である。精度レベルは、セルが要求される精度の指定サイズ（対角線）を超えないように計算される。これがサポートされる12レベルより高い精度レベルにつながる場合、（例えば、距離<5.6cmの場合）その値は拒否される。
- bounds
  - 任意。バケツ内の点をフィルタリングするためのバウンディングボックス。
- size
  - オプション。返すジオハッシュバケットの最大数(デフォルトは10,000)。結果が切り捨てられる際、バケットは含まれるドキュメントの量に基づいて優先順位がつけられる。
- shard_size
  - オプション。最終結果で返される上位セルをより正確にカウントするために、集約のデフォルトは各シャードから max(10,(size x number-of-shards)) のバケットを返すようになっている。このヒューリスティックな方法が好ましくない場合は、このパラメータを使用して、各シャードから考慮される数を上書きすることができます。
