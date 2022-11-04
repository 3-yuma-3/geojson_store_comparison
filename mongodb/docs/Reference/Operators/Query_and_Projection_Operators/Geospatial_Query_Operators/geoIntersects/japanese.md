# $geoIntersects
## Definition
地理空間データが指定された GeoJSON オブジェクトと交差するドキュメント、つまり、データと指定されたオブジェクトの交差点が空でないドキュメントを選択します。

geoIntersects 演算子は、$geometry 演算子を使用して GeoJSON オブジェクトを指定します。デフォルトの座標参照系 (CRS) を使用して GeoJSON ポリゴンまたはマルチポリゴンを指定するには、次の構文を使用します。

```bash
{
  <location field>: {
     $geoIntersects: {
        $geometry: {
           type: "<GeoJSON object type>" ,
           coordinates: [ <coordinates> ]
        }
     }
  }
}
```

単一の半球よりも大きな面積を持つ GeoJSON ジオメトリを指定する $geoIntersects クエリでは、デフォルトの CRS を使用すると、補完するジオメトリのクエリになります。

カスタム MongoDB CRS を使って一重の GeoJSON ポリゴンを指定するには、 $geometry 式にカスタム MongoDB CRS を指定した次のプロトタイプを使用します。

```bash
{
  <location field>: {
     $geoIntersects: {
        $geometry: {
           type: "Polygon" ,
           coordinates: [ <coordinates> ],
           crs: {
              type: "name",
              properties: { name: "urn:x-mongodb:crs:strictwinding:EPSG:4326" }
           }
        }
     }
  }
}
```

カスタムの MongoDB CRS は反時計回りの巻き順で、 $geoIntersects で一辺が半球以上の GeoJSON ポリゴンを指定したクエリをサポートします。指定した多角形が半球よりも小さい場合は、 MongoDB CRS での $geoIntersects の挙動はデフォルトの CRS と同じになります。 "Big Polygonsも参照ください。

緯度と経度の座標を指定する場合は、まず経度を記載し、次に緯度を記載します。

- 有効な経度の値は-180から180の間です。
- 緯度の有効範囲は-90〜90です。

## Behavior
### Geospatial Indexes
geoIntersects は球状ジオメトリを使用します。geoIntersects は、地理空間インデックスを必要としません。ただし、地理空間インデックスを使用すると、クエリーのパフォーマンスが向上します。2dsphere 地理空間インデックスのみが $geoIntersects をサポートします。

### Degenerate Geometry
geoIntersects は、ポリゴンが自身のエッジ、自身の頂点、または頂点またはエッジを共有し て内部空間を持たない別のポリゴンと交差することを保証するものではありません。

### "Big" Polygons
geoIntersects では、単一の半球よりも大きな面積を持つ単一の輪のポリゴンを指定する場合は、 カスタム MongoDB 座標参照系を $geometry 式に含めてください。それ以外のすべての GeoJSON ポリゴンで半球より大きな面積を持つ場合は、 $geoIntersect で相補的なジオメトリを検索します。

## Examples
### Intersects a Polygon
次の例は、$geoIntersects を使って、座標配列で定義された Polygon と交差するすべての loc データを選択するものです。ポリゴンの面積は、半球の面積よりも小さくなります。

```bash
db.places.find(
   {
     loc: {
       $geoIntersects: {
          $geometry: {
             type: "Polygon" ,
             coordinates: [
               [ [ 0, 0 ], [ 3, 6 ], [ 6, 1 ], [ 0, 0 ] ]
             ]
          }
       }
     }
   }
)
```

半球以上の面積を持つ単環のポリゴンについては、「Intersects a "Big" Polygon」を参照してください。

### Intersects a "Big" Polygo
面積が半球よりも大きい一重の GeoJSON ポリゴンでクエリを実行するには、 $geometry 式で MongoDB 独自の座標参照系を指定する必要があります。たとえば

```bash
db.places.find(
   {
     loc: {
       $geoIntersects: {
          $geometry: {
             type : "Polygon",
             coordinates: [
               [
                 [ -100, 60 ], [ -100, 0 ], [ -100, -60 ], [ 100, -60 ], [ 100, 60 ], [ -100, 60 ]
               ]
             ],
             crs: {
                type: "name",
                properties: { name: "urn:x-mongodb:crs:strictwinding:EPSG:4326" }
             }
          }
       }
     }
   }
)
```
