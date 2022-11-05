# Geotile grid aggregation
geo_point と geo_shape の値を、グリッドを表すバケットにグループ化するマルチバケット集計です。結果として得られるグリッドは疎にすることができ、一致するデータを持つセルのみを含む。各セルは、多くのオンライン地図サイトで使用されている地図タイルに対応します。各セルは「{zoom}/{x}/{y}」の形式でラベル付けされ、zoomはユーザーが指定した精度と同じである。

- 高精度のキーはxとyの範囲が大きく、小さな面積をカバーするタイルを表す。
- 低精度のキーは、xとyの範囲が小さく、それぞれが大きな面積をカバーするタイルを表します。

精度（ズーム）が地上でのサイズとどのように関連するかについては、ズームレベルのドキュメントを参照してください。この集計の精度は、0から29の間とすることができる。

**WARNING**
長さ29の最高精度のジオタイルは、10cm×10cm以下の土地をカバーするセルを生成するため、高精度のリクエストはRAMと結果サイズの点で非常にコストがかかる可能性があります。以下の例では、高レベルの詳細情報を要求する前に、まず小さな地理的領域に集約をフィルタリングする方法を紹介しています。

geotile_grid を使用して集計できるのは、明示的にマップされた geo_point または geo_shape フィールドのみです。geo_point フィールドに配列が含まれている場合、geotile_grid はすべての配列の値を集計します。

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
      "geotile_grid": {
        "field": "location",
        "precision": 8
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
          "key": "8/131/84",
          "doc_count": 3
        },
        {
          "key": "8/129/88",
          "doc_count": 2
        },
        {
          "key": "8/131/85",
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
          "geotile_grid": {
            "field": "location",
            "precision": 22
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
            "key": "22/2154412/1378379",
            "doc_count": 1
          },
          {
            "key": "22/2154385/1378332",
            "doc_count": 1
          },
          {
            "key": "22/2154259/1378425",
            "doc_count": 1
          }
        ]
      }
    }
  }
}
```

## Requests with additional bounding box filtering
geotile_grid 集約は、オプションの bounds パラメータをサポートし、考慮されるセルを指定された境界と交差するものに制限します。bounds パラメータは、Geo Bounding Box クエリで指定された境界と同じフォーマットで境界ボックスを受け取ります。このバウンディングボックスは、集計前にポイントをフィルタリングするための追加のgeo_bounding_boxクエリとともに、またはなしで使用することができます。これは独立したバウンディングボックスで、集約のコンテキストで定義された追加の geo_bounding_box クエリと交差したり、等しかったり、不連続になったりします。

```bash
POST /museums/_search?size=0
{
  "aggregations": {
    "tiles-in-bounds": {
      "geotile_grid": {
        "field": "location",
        "precision": 22,
        "bounds": {
          "top_left": "POINT (4.9 52.4)",
          "bottom_right": "POINT (5.0 52.3)"
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
          "key": "22/2154412/1378379",
          "doc_count": 1
        },
        {
          "key": "22/2154385/1378332",
          "doc_count": 1
        },
        {
          "key": "22/2154259/1378425",
          "doc_count": 1
        }
      ]
    }
  }
}
```

### Aggregating geo_shape fields
Geoshapeフィールドの集計は、1つの形状が複数のタイルでカウントされることを除いて、ポイントの場合と同じように動作します。形状は、その形状のいずれかの部分がそのタイルと交差している場合、マッチング値のカウントに貢献します。以下は、これを示す画像である。

## Options
- field
  - 必須。GeoPoints でインデックス化されたフィールドの名前。
- precision
  - 任意。結果のセル/バケットを定義するために使用されるキーの整数倍。デフォルトは 7。0,29] 以外の値は拒否される。
- bounds
  - オプション。バケツ内の点をフィルタリングするためのバウンディングボックス。
- size
  - オプション。返すジオハッシュバケットの最大数(デフォルトは10,000)。結果が切り捨てられる際、バケットは含まれるドキュメントの量に基づいて優先順位がつけられる。
- shard_size
  - オプション。最終結果で返される上位セルをより正確にカウントするために、集約のデフォルトは各シャードから max(10,(size x number-of-shards)) のバケットを返すようになっている。このヒューリスティックな方法が好ましくない場合は、このパラメータを使用して、各シャードから考慮される数を上書きすることができます。
