# Geo-bounds aggregation
あるフィールドのすべてのジオ値を含むバウンディングボックスを計算するメトリックアグリゲーション。

例

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
  "query": {
    "match": { "name": "musée" }
  },
  "aggs": {
    "viewport": {
      "geo_bounds": {
        "field": "location", ---1
        "wrap_longitude": true ---2
      }
    }
  }
}
```

1. geo_bounds 集約は、境界を取得するために使用するフィールドを指定します。
2. wrap_longitude はオプションのパラメータで、バウンディングボックスが国際日付変更線をオーバーラップすることを許可すべきかどうかを指定する。デフォルト値はtrueである。

上記の集計は、ビジネスタイプがshopであるすべてのドキュメントのロケーションフィールドのバウンディングボックスを計算する方法を示しています。

上記の集計に対する応答です。

```json
{
  ...
  "aggregations": {
    "viewport": {
      "bounds": {
        "top_left": {
          "lat": 48.86111099738628,
          "lon": 2.3269999679178
        },
        "bottom_right": {
          "lat": 48.85999997612089,
          "lon": 2.3363889567553997
        }
      }
    }
  }
}
```

## Geo Bounds Aggregation on geo_shape fields
Geo Bounds Aggregation は、geo_shape フィールドでもサポートされています。

wrap_longitude を true（デフォルト）に設定すると、バウンディングボックスが国際日付変更線をオーバーラップし、top_left の経度が top_right の経度より大きい境界を返すことができるようになります。

たとえば、地理的なバウンディング・ボックスの右上の経度は、通常、左下の経度よりも大きくなります。ただし、地域が 180° 子午線を通過する場合、左下経度の値は右上経度の値より大きくなります。詳細については、Open Geospatial Consortium の Web サイトの Geographic bounding box を参照してください。

例

```bash
PUT /places
{
  "mappings": {
    "properties": {
      "geometry": {
        "type": "geo_shape"
      }
    }
  }
}

POST /places/_bulk?refresh
{"index":{"_id":1}}
{"name": "NEMO Science Museum", "geometry": "POINT(4.912350 52.374081)" }
{"index":{"_id":2}}
{"name": "Sportpark De Weeren", "geometry": { "type": "Polygon", "coordinates": [ [ [ 4.965305328369141, 52.39347642069457 ], [ 4.966979026794433, 52.391721758934835 ], [ 4.969425201416015, 52.39238958618537 ], [ 4.967944622039794, 52.39420969150824 ], [ 4.965305328369141, 52.39347642069457 ] ] ] } }

POST /places/_search?size=0
{
  "aggs": {
    "viewport": {
      "geo_bounds": {
        "field": "geometry"
      }
    }
  }
}
```

```json
{
  ...
  "aggregations": {
    "viewport": {
      "bounds": {
        "top_left": {
          "lat": 52.39420966710895,
          "lon": 4.912349972873926
        },
        "bottom_right": {
          "lat": 52.374080987647176,
          "lon": 4.969425117596984
        }
      }
    }
  }
}
```
