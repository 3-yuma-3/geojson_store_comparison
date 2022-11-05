# Geohex grid aggregation
geo_point 値をグリッドを表すバケットにグループ化するマルチバケット集計。結果として得られるグリッドは疎になる可能性があり、一致するデータを持つセルのみを含む。各セルはH3セルインデックスに対応し、H3Index表現を使用してラベル付けされます。

精度（ズーム）が地上でのサイズとどのように相関するかについては、H3解像度のセルエリアのテーブルを参照してください。このアグリゲーションの精度は、0から15の間とすることができる。

**WARNING**
高精度のリクエストは、RAMと結果サイズの点で非常に高価になる可能性があります。たとえば、精度が 15 の最高精度のジオヘックスは、10cm×10cm 以下のセルを生成します。フィルタを使用して、高精度の要求をより小さい地理的領域に制限することをお勧めします。例としては、高精度のリクエストを参照してください。

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
      "geohex_grid": {
        "field": "location",
        "precision": 4
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
          "key": "841969dffffffff",
          "doc_count": 3
        },
        {
          "key": "841fb47ffffffff",
          "doc_count": 2
        },
        {
          "key": "841fa4dffffffff",
          "doc_count": 1
        }
      ]
    }
  }
}
```

## High-precision requests
詳細なバケットを要求する場合（通常、「拡大された」地図を表示するため）、対象領域を絞り込むために geo_bounding_box のようなフィルタを適用する必要があります。そうしないと、何百万ものバケツが作成され、返される可能性があります。

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
          "geohex_grid": {
            "field": "location",
            "precision": 12
          }
        }
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
    "zoomed-in": {
      "doc_count": 3,
      "zoom1": {
        "buckets": [
          {
            "key": "8c1969c9b2617ff",
            "doc_count": 1
          },
          {
            "key": "8c1969526d753ff",
            "doc_count": 1
          },
          {
            "key": "8c1969526d26dff",
            "doc_count": 1
          }
        ]
      }
    }
  }
}
```

## Requests with additional bounding box filtering
geohex_grid 集約は、オプションの bounds パラメータをサポートしており、考慮するセルを指定した境界と交差するものに限定することができます。bounds パラメータは、geo-bounding box クエリと同じバウンディングボックス形式を受け付けます。このバウンディングボックスは、集計前にポイントをフィルタリングするための追加の geo_bounding_box クエリとともに、またはなしで使用することができる。これは独立したバウンディングボックスで、集約のコンテキストで定義された追加の geo_bounding_box クエリと交差したり、等しくなったり、あるいは不連続になったりすることができます。

```bash
POST /museums/_search?size=0
{
  "aggregations": {
    "tiles-in-bounds": {
      "geohex_grid": {
        "field": "location",
        "precision": 12,
        "bounds": {
          "top_left": "POINT (4.9 52.4)",
          "bottom_right": "POINT (5.0 52.3)"
        }
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
    "tiles-in-bounds": {
      "buckets": [
        {
          "key": "8c1969c9b2617ff",
          "doc_count": 1
        },
        {
          "key": "8c1969526d753ff",
          "doc_count": 1
        },
        {
          "key": "8c1969526d26dff",
          "doc_count": 1
        }
      ]
    }
  }
}
```

## Options
- field
  - (必須、 文字列） イ ンデ ッ ク ス さ れた ジオポー ト 値を内容 と し て持つ フ ィ ール ド 。geo_point フ ィ ール ド と し て明示的にマ ッ プ し てお く 必要があ り ます。フィールドが配列を含む場合、geohex_grid はすべての配列値を集約する。
- precision
  - (オプ シ ョ ナル、 整数） 結果内のセル / バ ッ ケ ッ ト を定義する ために用い ら れる キーの整数倍。デフォルトは 6。0,15] 以外の値は却下されます。
- bounds
  - (Optional, object) 各バケツ内のジオポイントをフィルターするために使用される境界ボックス。ジオバウンディングボックスクエリと同じバウンディングボックス形式を受け付けます。
- size
  - (オプション、整数) 返すバケツ数の最大値。デフォルトは10,000。結果が切り詰められるとき、バケツは、それが含むドキュメントの量に基づいて優先されます。
- shard_size
  - (オプション, integer) 各シャードから返されるバケットの数。デフォルトは max(10,(size x number-of-shards)) で、最終結果の上位セルをより正確にカウントできるようにします。
