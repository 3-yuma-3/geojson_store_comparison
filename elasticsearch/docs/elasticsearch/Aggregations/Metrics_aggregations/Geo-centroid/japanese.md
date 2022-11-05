# Geo-centroid aggregation
geoフィールドの全座標値から加重セントロイドを計算するメトリック集計。

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
{"location": "POINT (4.912350 52.374081)", "city": "Amsterdam", "name": "NEMO Science Museum"}
{"index":{"_id":2}}
{"location": "POINT (4.901618 52.369219)", "city": "Amsterdam", "name": "Museum Het Rembrandthuis"}
{"index":{"_id":3}}
{"location": "POINT (4.914722 52.371667)", "city": "Amsterdam", "name": "Nederlands Scheepvaartmuseum"}
{"index":{"_id":4}}
{"location": "POINT (4.405200 51.222900)", "city": "Antwerp", "name": "Letterenhuis"}
{"index":{"_id":5}}
{"location": "POINT (2.336389 48.861111)", "city": "Paris", "name": "Musée du Louvre"}
{"index":{"_id":6}}
{"location": "POINT (2.327000 48.860000)", "city": "Paris", "name": "Musée d'Orsay"}

POST /museums/_search?size=0
{
  "aggs": {
    "centroid": {
      "geo_centroid": {
        "field": "location" ---1
      }
    }
  }
}
```

1. geo_centroid 集約は、セントロイドを計算するために使用するフィールドを指定します。(注意: フィールドは Geopoint タイプである必要があります)

上記の集計は、犯罪タイプが強盗であるすべてのドキュメントのロケーションフィールドのセントロイドを計算する方法を示している。

上記の集計に対する応答。

```json
{
  ...
  "aggregations": {
    "centroid": {
      "location": {
        "lat": 51.00982965203002,
        "lon": 3.9662131341174245
      },
      "count": 6
    }
  }
}
```

geo_centroid 集約は、他のバケット集約のサブ集約として組み合わせると、より興味深いものになります。

例

```bash
POST /museums/_search?size=0
{
  "aggs": {
    "cities": {
      "terms": { "field": "city.keyword" },
      "aggs": {
        "centroid": {
          "geo_centroid": { "field": "location" }
        }
      }
    }
  }
}
```

上記の例では、各都市の美術館の中心的な場所を見つけるための条件バケット集計の下位集計として、geo_centroidを使用しています。

上記の集約に対するレスポンスです。

```json
{
  ...
  "aggregations": {
    "cities": {
      "sum_other_doc_count": 0,
      "doc_count_error_upper_bound": 0,
      "buckets": [
        {
          "key": "Amsterdam",
          "doc_count": 3,
          "centroid": {
            "location": {
              "lat": 52.371655656024814,
              "lon": 4.909563297405839
            },
            "count": 3
          }
        },
        {
          "key": "Paris",
          "doc_count": 2,
          "centroid": {
            "location": {
              "lat": 48.86055548675358,
              "lon": 2.3316944623366
            },
            "count": 2
          }
        },
        {
          "key": "Antwerp",
          "doc_count": 1,
          "centroid": {
            "location": {
              "lat": 51.22289997059852,
              "lon": 4.40519998781383
            },
            "count": 1
          }
        }
      ]
    }
  }
}
```

## Geo Centroid Aggregation on geo_shape fields
ジオシェイプのセントロイドのメトリックは、ポイントの場合よりも微妙である。形状を含む特定の集約バケットのセントロイドは、バケット内の最も次元の高い形状タイプのセントロイドとなる。例えば、バケットに多角形と直線からなる形状が含まれている場合、直線はセントロイドの指標に寄与しない。形状の種類によってセントロイドの計算方法は異なります。円形で取り込まれた封筒や円は、多角形として扱われます。

- [Multi]Point
  - 全座標の等加重平均
- [Multi]LineString
  - 各区分の重みはその長さ（度）で、各区分のすべての重心の加重平均。
- [Multi]Polygon
  - 三角形は連続する2つの頂点と始点によって形成される. 穴は負の重みを持つ. 重みは三角形の面積をdeg^2単位で計算したものである.
- GeometryCollection
  - 最も次元の高いジオメトリのセントロイド。Polygons and Lines and/or Points の場合、線または点は無視される。Lines and Points の場合、Point は無視されます。

Example:

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
    "centroid": {
      "geo_centroid": {
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
    "centroid": {
      "location": {
        "lat": 52.39296147599816,
        "lon": 4.967404240742326
      },
      "count": 2
    }
  }
}
```

**WARNING**
Using geo_centroid as a sub-aggregation of geohash_grid
geohash_grid 集約は、個々のジオポイントではなく、ドキュメントをバケットに配置します。ドキュメントの geo_point フィールドに複数の値が含まれている場合、そのドキュメントが複数のバケツに割り当てられる可能性があり、たとえそのジオポイントの 1 つ以上がバケツの境界の外にある場合でも同様です。

geocentroid サブ集計も使用する場合、各セントロイドはバケツ内のすべてのジオポイントを使用して計算され、バケツ境界の外にあるジオポイントも含まれます。このため、セントロイドがバケツ境界の外側になることがあります。
