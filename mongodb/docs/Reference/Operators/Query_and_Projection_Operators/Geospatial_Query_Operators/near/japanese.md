# $near
## Definition
地理空間クエリで、最も近いものから最も遠いものまでドキュメントを返す点を指定します。near 演算子は、GeoJSON 点またはレガシー座標点のいずれかを指定できます。

near には、地理空間インデックスが必要です。

- GeoJSON ポイントを指定する場合は、2dsphere インデックス。
- レガシー座標を使用してポイントを指定する場合は、2d インデックス。

GeoJSON ポイントを指定するには、$near 演算子に 2dsphere インデックスが必要で、次のような構文になります。

```bash
{
   <location field>: {
     $near: {
       $geometry: {
          type: "Point" ,
          coordinates: [ <longitude> , <latitude> ]
       },
       $maxDistance: <distance in meters>,
       $minDistance: <distance in meters>
     }
   }
}
```

緯度と経度の座標を指定する場合は、まず経度を記載し、次に緯度を記載します。

- 有効な経度の値は-180から180の間です。
- 緯度の有効範囲は-90〜90です。

GeoJSON ポイントを指定する場合、オプションの $minDistance および $maxDistance 指定を使用すると、$near の結果をメートル単位の距離で制限することができます。

- minDistanceは、中心点から指定した距離以上離れているドキュメントに結果を制限します。
- minDistanceでは、中心点から指定した距離以上離れたドキュメントに結果が限定されます。

レガシー座標を使用して点を指定するには、$near には 2 次元インデックスが必要で、次の構文になります。

```bash
{
  $near: [ <x>, <y> ],
  $maxDistance: <distance in radians>
}
```


レガシー座標を指定する場合、オプションの $maxDistance 指定を使用すると、$near の結果をラジアン単位の距離で制限することができます。maxDistanceを指定すると、中心点から最大で指定した距離だけ離れたドキュメントに結果が制限されます。

## Behavior
### Special Indexes Restriction
特殊な地理空間インデックスを必要とする $near 演算子を、別の特殊なインデックスを必要とするクエリ演算子やコマンドと組み合わせることはできません。たとえば、$near と $text クエリを組み合わせることはできません。

### Sharded Collections
MongoDB 4.0 からは、$near クエリがシャーディングコレクションでサポートされるようになりました。

そのかわり、シャードされたクラスタでは $geoNear 集約ステージか geoNear コマンド (MongoDB 4.0 以前で利用可能) を使わなければなりません。

### Sort Operation
near は、ドキュメントを距離でソートします。クエリに sort() も指定すると、 sort() はマッチしたドキュメントを並べ替え、 $near がすでに実行しているソート操作を事実上上書きします。地理空間クエリで sort() を使用する場合は、 $near のかわりにドキュメントのソートを行わない $geoWithin 演算子を使用することを検討しましょう。

See also:
2d Indexes and Geospatial Near Queries

## Examples
### Query on GeoJSON Data
緯度と経度の座標を指定する場合は、まず経度を記載し、次に緯度を記載します。

- 有効な経度の値は-180から180の間です。
- 緯度の有効範囲は-90〜90です。

2dsphereインデックスを持つコレクションplaceを考えてみよう。

次の例では、指定された GeoJSON ポイントから少なくとも 1000 メートル、最大 5000 メートルの距離にあるドキュメントを、近いものから遠いものへとソートして返します。

```bash
db.places.find(
   {
     location:
       { $near :
          {
            $geometry: { type: "Point",  coordinates: [ -73.9667, 40.78 ] },
            $minDistance: 1000,
            $maxDistance: 5000
          }
       }
   }
)
```

### Query on Legacy Coordinates
緯度と経度の座標を指定する場合は、まず経度を記載し、次に緯度を記載します。

- 有効な経度の値は-180から180の間です。
- 緯度の有効範囲は-90〜90です。

2dのインデックスを持つコレクションlegacy2dを考えてみましょう。

次の例は、指定されたレガシー座標のペアから最大0.10ラジアン離れたドキュメントを、最も近いものから遠いものへとソートして返すものです。

```bash
db.legacy2d.find(
   { location : { $near : [ -73.9667, 40.78 ], $maxDistance: 0.10 } }
)
```
