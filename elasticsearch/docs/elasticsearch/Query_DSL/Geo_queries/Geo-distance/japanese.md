# Geo-distance query
ジオポイントから指定された距離内にあるジオポイントとジオシェイプの値にマッチします。

## Example
次のような文書がインデックスされていると仮定する。

```
PUT /my_locations
{
  "mappings": {
    "properties": {
      "pin": {
        "properties": {
          "location": {
            "type": "geo_point"
          }
        }
      }
    }
  }
}

PUT /my_locations/_doc/1
{
  "pin": {
    "location": {
      "lat": 40.12,
      "lon": -71.34
    }
  }
}

PUT /my_geoshapes
{
  "mappings": {
    "properties": {
      "pin": {
        "properties": {
          "location": {
            "type": "geo_shape"
          }
        }
      }
    }
  }
}

PUT /my_geoshapes/_doc/1
{
  "pin": {
    "location": {
      "type" : "polygon",
      "coordinates" : [[[13.0 ,51.5], [15.0, 51.5], [15.0, 54.0], [13.0, 54.0], [13.0 ,51.5]]]
    }
  }
}
```

geo_distance フィルターを使用して、別のジオポイントから指定された距離内のジオポイント値に一致させます。

```
GET /my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_distance": {
          "distance": "200km",
          "pin.location": {
            "lat": 40,
            "lon": -70
          }
        }
      }
    }
  }
}
```

同じフィルターを使って、指定された距離内の geo_shape 値をマッチングさせる。

```
GET my_geoshapes/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_distance": {
          "distance": "200km",
          "pin.location": {
            "lat": 40,
            "lon": -70
          }
        }
      }
    }
  }
}
```

geo_point と geo_shape の両方の値にマッチするように、両方のインデックスを検索します。

```
GET my_locations,my_geoshapes/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_distance": {
          "distance": "200km",
          "pin.location": {
            "lat": 40,
            "lon": -70
          }
        }
      }
    }
  }
}
```

## Accepted formats
geo_point 型がジオポイントのさまざまな表現を受け入れるのと同じように、フィルタもそれを受け入れることができます。

### Lat lon as properties

```
GET /my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_distance": {
          "distance": "12km",
          "pin.location": {
            "lat": 40,
            "lon": -70
          }
        }
      }
    }
  }
}
```

### Lat lon as array
lon, lat]でフォーマットする。GeoJSONに準拠するため、ここではlon/latの順序に注意。

```
GET /my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_distance": {
          "distance": "12km",
          "pin.location": [ -70, 40 ]
        }
      }
    }
  }
}
```

### Lat lon as WKT string
Well-Known Textのフォーマット。

```
GET /my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_distance": {
          "distance": "12km",
          "pin.location": "POINT (-70 40)"
        }
      }
    }
  }
}
```

### Geohash

```
GET /my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_distance": {
          "distance": "12km",
          "pin.location": "drm3btev3e86"
        }
      }
    }
  }
}
```

## Options
フィルターで許可されるオプションは次のとおりです。

- `distance`
  - 指定した場所を中心とした円の半径。この円内に入る点がマッチとみなされる。距離は様々な単位で指定することができる。距離の単位」を参照。
- `distance_type`
  - 距離の計算方法。円弧（デフォルト）、平面（高速だが、長距離や極に近いところでは不正確）のいずれかを選択できる。
- `_name`
  - クエリを識別するためのオプションの名前フィールド
- `validation_method`
  - IGNORE_MALFORMED に設定すると、無効な緯度・経度のジオポイントを受け入れ、COERCE に設定すると、さらに正しい座標を推測しようとします（デフォルトは STRICT）。

## Multi location per document
geo_distance フィルタは、文書ごとに複数の位置 / 地点を扱うことができます。一つの位置/点がフィルタにマッチすると、その文書がフィルタの対象になります。

## Ignore unmapped
ignore_unmapped オプションを true に設定すると、 マップされていないフィールドを無視し、このクエリではどのドキュメントにもマッチしなくなります。これは、マッピングが異なる複数のインデックスに対してクエリを実行する際に有用です。false (デフォルト値) に設定すると、フィールドがマップされていない場合に例外をスローします。
