# Geospatial Queries
MongoDB は、地理空間データに対するクエリ操作をサポートしています。このセクションでは、MongoDB の地理空間機能について紹介します。

## Geospatial Data
MongoDB では、地理空間データを GeoJSON オブジェクトとして、あるいは レガシーな座標ペアとして保存できます。

### GeoJSON Objects
地球のような球体上の形状を計算するには、位置データをGeoJSONオブジェクトとして保存します。

GeoJSON データを指定するには、次のような埋め込みドキュメントを使用します。

- GeoJSON オブジェクトの種類を指定する type というフィールドと
- オブジェクトの座標を指定する coordinates というフィールドがあります。

緯度と経度の座標を指定する場合は、まず経度を記載し、次に緯度を記載します。

- 有効な経度の値は-180から180の間です。
- 緯度の有効範囲は-90〜90です。

```bash
<field>: { type: <GeoJSON type> , coordinates: <coordinates> }
```

例えば、GeoJSONのPointを指定する場合。

```json
location: {
      type: "Point",
      coordinates: [-73.856077, 40.848447]
}
```

MongoDB でサポートされている GeoJSON オブジェクトの一覧とその例については、 GeoJSON オブジェクト を参照してください。

MongoDB の GeoJSON オブジェクトへの地理空間クエリーは球体で計算します。MongoDB は GeoJSON オブジェクトへの地理空間クエリーに WGS84 参照系を使用します。

### Legacy Coorinate Pairs
ユークリッド平面上の距離を計算するには、位置データをレガシー座標のペアとして保存し、そのペアに対して 2d インデックスを使います。MongoDB は、レガシー座標ペアの球面計算を 2dsphere インデックスで球面計算をサポートしています。

データをレガシー座標ペアで指定するには、配列 (推奨) か埋め込みドキュメントを使用します。

#### Specify via an array (Preferred):

```bash
<field>: [ <x>, <y> ]
```

緯度と経度の座標を指定する場合は、まず経度を、次に緯度を記載します。

```bash
<field>: [<longitude>, <latitude> ]
```

- 有効な経度値は、-180から180の間です。
- 有効な緯度の値は、-90から90の間です（両方を含む）。

#### Specify via an embedded document:

```bash
<field>: { <field1>: <x>, <field2>: <y> }
```

緯度経度の座標を指定する場合、最初のフィールドはフィールド名に関係なく、経度の値、2番目のフィールドは緯度の値でなければならない；すなわち

```bash
<field>: { <field1>: <longitude>, <field2>: <latitude> }
```

- 有効な経度値は、-180から180の間です。
- 有効な緯度の値は、-90 から 90 の間です（両方を含む）。

レガシーな座標ペアを指定する場合、言語によっては連想マップの順序が保証されないため、埋め込み文書よりも配列の方が望ましい。

## Geospatial Indexes
MongoDBは、地理空間クエリをサポートするために、以下の地理空間インデックスタイプを提供します。

### 2dsphere
2dsphere インデックスは、地球のような球体上のジオメトリを計算するクエリをサポートします。を計算するクエリをサポートします。

2dsphere インデックスを作成するには、 db.collection.createIndex() メソッドを使用して、インデックスタイプとして文字列リテラル "2dsphere" を指定する。

```bash
db.collection.createIndex( { <location field> : "2dsphere" } )
```

ここで、<location field>はフィールドで、その値は GeoJSON オブジェクト または レガシー座標のペアです。

2dsphere インデックスの詳細については、2dsphere インデックスを参照してください。

### 2d
2d インデックスは、2 次元平面上のジオメトリを計算するクエリをサポートします。このインデックスは、球面上で計算する $nearSphere クエリーをサポートできますが、可能であれば、球面上のクエリーには 2dsphere インデックスを使用してください。

2Dインデックスを作成するには、db.collection.createIndex()メソッドを使用し、キーとしてlocationフィールド、インデックスタイプとして文字列リテラル「2D」を指定します。

```bash
db.collection.createIndex( { <location field> : "2d" } )
```

ここで、<location field>はレガシー座標のペアを値とするフィールドである。レガシー座標のペアを値とするフィールドです。

2次元インデックスの詳細については、2次元インデックスを参照してください。

### Geospatial Indexes and Sharded Collections
コレクションをシャーディングする際、地理空間インデックスをシャードキーとして使用することはできません。ただし、別のフィールドをシャードキーとして使用することで、シャードされたコレクションに地理空間インデックスを作成することは可能です。

シャード化されたコレクションでは、次の地理空間操作がサポートされています。

- geoNear 集約ステージ
- near および $nearSphere クエリ演算子 (MongoDB 4.0 以降で使用可能)

MongoDB 4.0 からは、$near と $nearSphere クエリがシャード化されたコレクションでサポートされるようになりました。

それ以前のバージョンの MongoDB では、$near および $nearSphere クエリはシャード・コレクションではサポートされていません。その代わり、シャード・クラスタでは $geoNear 集約ステージか geoNear コマンド (MongoDB 4.0 およびそれ以前で利用可能) を使わなければなりません。

geoWithin と $geoIntersects を使って、シャード化されたクラスタの地理空間データを問い合わせることもできます。

### Covered Queries
Geospatial indexes cannot cover a query.

## Geospatial Queries
球形クエリには、2dsphere インデックスの結果を使用します。

球形クエリに2dインデックスを使用すると、極を回り込むような不正な結果になることがあります。

### Geospatial Query Operators
MongoDB には以下の地理空間クエリ演算子があります。

- `$geoIntersects`
  - GeoJSON ジオメトリと交差するジオメトリを選択します。2dsphere インデックスは $geoIntersects をサポートします。
- `$geoWithin`
  - バウンディング GeoJSON ジオメトリ内のジオメトリを選択します。2dsphere と 2d インデックスは $geoWithin をサポートします。
- `$near`
  - ある地点に近接する地理空間オブジェクトを返す。地理空間インデックスが必要です。2dsphere と 2d インデックスは $near をサポートします。
- `$nearSphere`
  - 球面上のある点に近接する地理空間オブジェクトを返します。地理空間インデックスが必要です。2dsphere と 2d インデックスは $nearSphere をサポートします。

例を含む詳細は、個別のリファレンスページをご覧ください。

### Geospatial Aggregation Stage
MongoDBは以下の地理空間集約パイプラインのステージを提供します。

- `$geoNear`
  - 地理空間上の点への近接性に基づいて、ドキュメントの順序付きストリームを返します。地理空間データに対する $match、$sort、および $limit の機能を組み込んでいます。出力されるドキュメントには、追加の距離フィールドが含まれ、位置識別子フィールドを含めることができます。
  - geoNear には、地理空間インデックスが必要です。

例を含む詳細は、$geoNear のリファレンスページを参照してください。
## Geospatial Models
MongoDB の地理空間クエリは、平面か球面上のジオメトリを解釈することができます。

2dsphere インデックスは、球体クエリ (球面上のジオメトリを解釈するクエリ) のみをサポートしています。

2d インデックスは、平面クエリ（つまり、平面上のジオメトリを解釈するクエリ）と一部の球面クエリをサポートしています。2d インデックスは一部の球形クエリをサポートしていますが、これらの球形クエリに 2d インデックスを使用すると、エラーが発生する可能性があります。可能であれば、球形クエリには2dsphere インデックスを使用してください。

次の表は、それぞれの地理空間演算で使用される地理空間クエリ演算子、サポートされているクエリ、およびその一覧を示したものです。

- `$near` ( ジオJSON この線と次の線における重心点。2dsphere インデックス)
  - Spherical
  - と一緒に使用すると同じ機能を提供する $nearSphere 演算子も参照してください。ジオJSON と 2dsphere のインデックスを参照してください。
- `$near` ( レガシー座標 , 2d インデックス)
  - Flat
- `$nearSphere` ( ジオJSON ポイント2dsphere インデックス)
  - Spherical
  - GeoJSON ポイントと 2dsphere インデックスを使用する $near オペレーションと同じ機能を提供します。
  - 球形クエリの場合は、$near 演算子ではなく、球形クエリを名前に明示的に指定する $nearSphere を使用する方が望ましい場合があります。
- `$nearSphere` (レガシー座標, 2d インデックス)
  - Spherical
  - 代わりにGeoJSONポイントを使用します。
- `$geoWithin` : { $geometry: ... }
- `$geoWithin` : { $box: ... }
  - Flat
- `$geoWithin` : { $polygon: ... }
  - Flat
- `$geoWithin` : { $center: ... }
  - Flat
- `$geoWithin` : { $centerSphere: ... }
  - Spherical
- `$geoIntersects`
  - Spherical
- `$geoNear` aggregation stage (2dsphere index)
  - Spherical
- `$geoNear` aggregation stage (2d index)
  - Flat

## Examples
以下のドキュメントを含むコレクションプレイスを作成します。

```bash
db.places.insertMany( [
   {
      name: "Central Park",
      location: { type: "Point", coordinates: [ -73.97, 40.77 ] },
      category: "Parks"
   },
   {
      name: "Sara D. Roosevelt Park",
      location: { type: "Point", coordinates: [ -73.9928, 40.7193 ] },
      category: "Parks"
   },
   {
      name: "Polo Grounds",
      location: { type: "Point", coordinates: [ -73.9375, 40.8303 ] },
      category: "Stadiums"
   }
] )
```

次の操作は、locationフィールドに2dsphereインデックスを作成するものである。

```bash
db.places.createIndex( { location: "2dsphere" } )
```

上記の場所コレクションは、2dsphere インデックスを持っています。次のクエリは、$near 演算子を使用して、指定した GeoJSON ポイントから最低 1000 メートル、最高 5000 メートルのドキュメントを、近いものから遠いものの順に並べ替えて返します。

```bash
db.places.find(
   {
     location:
       { $near:
          {
            $geometry: { type: "Point",  coordinates: [ -73.9667, 40.78 ] },
            $minDistance: 1000,
            $maxDistance: 5000
          }
       }
   }
)
```

次の操作は、$geoNear 集約操作を使用して、クエリ・フィルタ { category: "Parks" } に一致するドキュメントを、指定された GeoJSON ポイントに最も近い順から最も遠い順に並べ替えて返します。

```bash
db.places.aggregate( [
   {
      $geoNear: {
         near: { type: "Point", coordinates: [ -73.9667, 40.78 ] },
         spherical: true,
         query: { category: "Parks" },
         distanceField: "calcDistance"
      }
   }
] )
```
