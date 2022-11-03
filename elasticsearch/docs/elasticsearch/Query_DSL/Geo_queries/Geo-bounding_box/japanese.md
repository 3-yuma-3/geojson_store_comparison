# Geo-bounding box query
バウンディングボックスと交差する geo_point と geo_shape の値にマッチします。

## Example
Assume the following documents are indexed:

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

geo_bounding_box フィルターを使用して、バウンディングボックスに交差するジオポイント値をマッチングさせます。ボックスを定義するために、2つの対向する角のジオポイント値を提供します。

```
GET my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "pin.location": {
            "top_left": {
              "lat": 40.73,
              "lon": -74.1
            },
            "bottom_right": {
              "lat": 40.01,
              "lon": -71.12
            }
          }
        }
      }
    }
  }
}
```

同じフィルターを使用して、バウンディングボックスと交差する geo_shape 値を照合します。

```
GET my_geoshapes/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "pin.location": {
            "top_left": {
              "lat": 40.73,
              "lon": -74.1
            },
            "bottom_right": {
              "lat": 40.01,
              "lon": -71.12
            }
          }
        }
      }
    }
  }
}
```

To match both geo_point and geo_shape values, search both indices:

```
GET my_locations,my_geoshapes/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "pin.location": {
            "top_left": {
              "lat": 40.73,
              "lon": -74.1
            },
            "bottom_right": {
              "lat": 40.01,
              "lon": -71.12
            }
          }
        }
      }
    }
  }
}
```

## Query options
- `_name`
  - フィルタを識別するためのオプションの名前フィールド
- `validation_method`
  - IGNORE_MALFORMEDに設定すると、無効な緯度・経度のジオポイントを受け入れる。COERCEに設定すると、正しい緯度・経度も推論しようとする。(デフォルトはSTRICT)。

## Accepted formats
geo_point 型がジオポイントのさまざまな表現を受け入れるのと同じように、フィルタもそれを受け入れることができます。

### Lat lon as properties

```
GET my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "pin.location": {
            "top_left": {
              "lat": 40.73,
              "lon": -74.1
            },
            "bottom_right": {
              "lat": 40.01,
              "lon": -71.12
            }
          }
        }
      }
    }
  }
}
```

### Lat lon as array

```
GET my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "pin.location": {
            "top_left": [ -74.1, 40.73 ],
            "bottom_right": [ -71.12, 40.01 ]
          }
        }
      }
    }
  }
}
```

### Lat lon as string
lat,lonでフォーマットする。

```
GET my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "pin.location": {
            "top_left": "POINT (-74.1 40.73)",
            "bottom_right": "POINT (-71.12 40.01)"
          }
        }
      }
    }
  }
}
```

### Bounding box as well-known text (WKT)

```
GET my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "pin.location": {
            "wkt": "BBOX (-74.1, -71.12, 40.73, 40.01)"
          }
        }
      }
    }
  }
}
```

### Geohash

```
GET my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "pin.location": {
            "top_left": "dr5r9ydj2y73",
            "bottom_right": "drj7teegpus6"
          }
        }
      }
    }
  }
}
```

ジオハッシュでバウンディングボックスの境界を指定する場合，ジオハッシュは矩形として扱われる。バウンディングボックスは、その左上が top_left パラメータで指定されたジオハッシュの左上隅に対応し、その右下が bottom_right パラメータで指定されたジオハッシュの右下に対応するように定義されています。

ジオハッシュの全領域に対応するバウンディングボックスを指定するには、top_left と bottom_right の両方のパラメータでジオハッシュを指定することができます。

```
GET my_locations/_search
{
  "query": {
    "geo_bounding_box": {
      "pin.location": {
        "top_left": "dr",
        "bottom_right": "dr"
      }
    }
  }
}
```

この例では、geohash drは左上隅を45.0,-78.75、右下隅を39.375,-67.5とするバウンディングボックスクエリを生成します。

## Vertices
バウンディングボックスの頂点は、top_leftとbottom_rightで設定するか、top_rightとbottom_leftのパラメータで設定することができる。値を対で設定する代わりに、単純な名前 top, left, bottom, right を使って別々に設定することができる。

```
GET my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "pin.location": {
            "top": 40.73,
            "left": -74.1,
            "bottom": 40.01,
            "right": -71.12
          }
        }
      }
    }
  }
}
```

## Multi location per document
このフィルターは、1つの文書に対して複数の場所/点を扱うことができます。1つの場所/点がフィルタに一致すると、その文書がフィルタに含まれます。

## Ignore unmapped
ignore_unmapped オプションを true に設定すると、 マップされていないフィールドを無視し、このクエリではどのドキュメントにもマッチしなくなります。これは、マッピングが異なる複数のインデックスに対してクエリを実行する際に有用です。false (デフォルト値) に設定すると、フィールドがマップされていない場合に例外をスローします。

## Notes on precision
ジオポイントの精度は限られており、インデックス時間中は常に切り捨てられます。クエリー時間では、バウンディングボックスの上部境界は切り捨てられ、下部境界は切り上げられます。その結果、下側の境界（バウンディングボックスの下端と左端）に沿った点は、丸め誤差のためにバウンディングボックスに入らない可能性があります。一方、上辺（上端と右端）に沿った点は、端から少し外れていてもクエリで選択されることがあります。丸め誤差は緯度で4.20e-8度以下、経度で8.39e-8度以下でなければならず、これは赤道上でも1cm以下の誤差に相当します。

また、ジオシェイプは丸め誤差のため、精度に限界があります。バウンディングボックスの底辺と左辺に沿ったジオシェープのエッジは、geo_bounding_boxクエリにマッチしないことがあります。ボックスの上端と右端からわずかに外れたジオシェイプエッジは、クエリにマッチする場合があります。
