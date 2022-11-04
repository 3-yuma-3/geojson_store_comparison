# $nearSphere
## Definition
地理空間クエリで、最も近いところから最も遠いところまでのドキュメントを返す点を指定します。MongoDB は、球形ジオメトリを使用して $nearSphere の距離を計算します。

nearSphere は、地理空間インデックスが必要です。

- 2dsphere インデックス (GeoJSON ポイントとして定義された位置データ用)
- 2d インデックスは、レガシー座標ペアとして定義された位置データ用です。GeoJSON ポイントで 2d インデックスを使用するには、GeoJSON オブジェクトの座標フィールドにインデックスを作成します。

nearSphere 演算子は、GeoJSON ポイントまたはレガシー座標ポイントのいずれかを指定できます。

GeoJSON ポイントを指定するには、次の構文を使用します。

```bash
{
  $nearSphere: {
     $geometry: {
        type : "Point",
        coordinates : [ <longitude>, <latitude> ]
     },
     $minDistance: <distance in meters>,
     $maxDistance: <distance in meters>
  }
}
```

- オプションの $minDistance は、中心点から指定した距離以上離れているドキュメントに結果を限定します。
- オプションの $maxDistance は、どちらのインデックスでも使用できます。

レガシー座標を使用して点を指定するには、次の構文を使用します。

```bash
{
  $nearSphere: [ <x>, <y> ],
  $minDistance: <distance in radians>,
  $maxDistance: <distance in radians>
}
```

- オプションの $minDistance は、クエリが 2dsphere インデックスを使用する場合のみ使用可能です。minDistance は、中心点から指定した距離以上離れているドキュメントに結果を限定します。
- オプションの $maxDistance は、どちらのインデックスでも使用できます。

レガシー座標に経度と緯度を使用する場合は、まず経度を指定し、次に緯度を指定します。

See also:
2d Indexes and Geospatial Near Queries

## Behavior
### Special Indexes Restriction
特殊な地理空間インデックスを必要とする $nearSphere 演算子を、別の特殊なインデックスを必要とするクエリ演算子またはコマンドと組み合わせることはできません。たとえば、$nearSphere と $text クエリを組み合わせることはできません。

### Sharded Collections
MongoDB 4.0 からは、$nearSphere クエリがシャーデッドコレクションでサポートされるようになりました。

そのかわり、シャードされたクラスタでは $geoNear 集約ステージか geoNear コマンド (MongoDB 4.0 以前で使用可能) を使用する必要があります。

### Sort Operation
$nearSphere は、ドキュメントを距離でソートします。クエリに sort() も指定すると、sort() はマッチしたドキュメントを並べ替え、$nearSphere がすでに実行しているソート操作を効果的に上書きします。地理空間クエリで sort() を使用する場合は、$nearSphere の代わりに、ドキュメントをソートしない $geoWithin 演算子を使用することを検討してください。

## Examples
### Specify Center Point Using GeoJSON
locationフィールドを持つ文書を含み、2dsphereインデックスを持つコレクションplacesを考える。

すると、次の例では、指定された地点から少なくとも1000メートル、最大5000メートルの距離にある人の位置を、近い順に返す。

### Specify Center Point Using Legacy Coordinates

#### 2d Index
locationフィールドにレガシー座標のペアを持つ文書を含み、2次元のインデックスを持つコレクションlegacyPlacesを考える。

そして、次の例は、指定された点から最大0.10ラジアン離れた位置にある文書を、最も近いものから遠いものの順に返します。

```bash
db.legacyPlaces.find(
   { location : { $nearSphere : [ -73.9667, 40.78 ], $maxDistance: 0.10 } }
)
```

#### 2dsphere Index
コレクションに 2dsphere インデックスがある場合は、オプションで $minDistance を指定することもできます。たとえば、次の例では、指定した点から少なくとも 0.0004 ラジアン離れた位置にあるドキュメントを、 最も近いものから順に返します。

```bash
db.legacyPlaces.find(
   { location : { $nearSphere : [ -73.9667, 40.78 ], $minDistance: 0.0004 } }
)
```
