# $geoWithin
## Definition
指定された形状内に完全に存在する地理空間データを持つドキュメントを選択します。

指定された図形は、GeoJSON Polygon (シングルリングまたはマルチリング)、GeoJSON MultiPolygon、またはレガシー座標ペアで定義された図形のいずれかとなります。geoWithin 演算子は、GeoJSON オブジェクトを指定するために $geometry 演算子を使用します。

デフォルトの座標参照系 (CRS) を使用して GeoJSON ポリゴンまたはマルチポリゴンを指定するには、次の構文を使用します。

```bash
{
   <location field>: {
      $geoWithin: {
         $geometry: {
            type: <"Polygon" or "MultiPolygon"> ,
            coordinates: [ <coordinates> ]
         }
      }
   }
}
```

単一の半球よりも大きな面積を持つ GeoJSON ジオメトリを指定する $geoWithin クエリでは、デフォルトの CRS を使用すると、補完的なジオメトリのクエリになりま す。

カスタム MongoDB CRS を使って一重の GeoJSON ポリゴンを指定するには、 $geometry 式にカスタム MongoDB CRS を指定した次のプロトタイプを使用します。

```bash
{
   <location field>: {
      $geoWithin: {
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

カスタムの MongoDB CRS は反時計回りの巻き順で、$geoWithin は面積が半球以上ある一重の GeoJSON ポリゴンを使ったクエリをサポートするようにします。指定した多角形が半球より小さい場合は、MongoDB CRS での $geoWithin の挙動はデフォルトの CRS と同じになります。"Big" Polygonsも参照ください。

平面上のレガシー座標のペアで定義された図形に含まれるものを問い合わせる場合は、 次の構文を使用します。

```bash
{
   <location field>: {
      $geoWithin: { <shape operator>: <coordinates> }
   }
}
```

使用可能な図形演算子は以下の通りです。
- $box
- $polygon
- $center (defines a circle), and
- $centerSphere (defines a circle on a sphere).

緯度経度を使用する場合は、経度、緯度の順に座標を指定します。

## Behavior
### Geospatial Indexes
geoWithin は、地理空間インデックスを必要としません。ただし、地理空間インデックスを使用すると、クエリーのパフォーマンスが向上します。2dsphere と 2d の両方の地理空間インデックスが $geoWithin をサポートします。

### Unsorted Results
geoWithin 演算子は、ソートされた結果を返しません。そのため、MongoDB は $geoWithin クエリを返すほうが、結果をソートする地理空間 $near や $nearSphere クエリより高速になります。

### Degenerate Geometry
geoWithin は、あるジオメトリがその構成ジオメトリを含むかどうか、 あるいはその構成ジオメトリを共有する別のポリゴンを含むかどうかを保証するものではありません。

### "Big" Polygons
geoWithin では、単一の半球よりも大きな面積を持つ単一の輪のポリゴンを指定する場合、$geometry 式にカスタム MongoDB 座標参照系を含めます; そうでない場合は $geoWithin が補完するジオメトリを照会します。それ以外のすべての GeoJSON ポリゴンで、半球よりも大きな面積を持つ場合は、$geoWithin は補完的なジオメトリを問い合わせます。

## Examples
### Within a Polygon
次の例は、GeoJSON ポリゴン内に完全に存在するすべてのロクデータを選択するものです。ポリゴンの面積は、半球の面積よりも小さい。

```bash
db.places.find(
   {
     loc: {
       $geoWithin: {
          $geometry: {
             type : "Polygon" ,
             coordinates: [ [ [ 0, 0 ], [ 3, 6 ], [ 6, 1 ], [ 0, 0 ] ] ]
          }
       }
     }
   }
)
```

単環のポリゴンで、面積が半球より大きいものについては、以下 Within a "Big" Polygon を参照してください。

### Within a "Big" Polygon
面積が半球よりも大きい一重の GeoJSON ポリゴンでクエリを実行するには、 $geometry 式で MongoDB 独自の座標参照系を指定する必要があります。たとえば

```bash
db.places.find(
   {
     loc: {
       $geoWithin: {
          $geometry: {
             type : "Polygon" ,
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

## $within
Deprecated since version 2.4: $geoWithin replaces $within in MongoDB 2.4.
