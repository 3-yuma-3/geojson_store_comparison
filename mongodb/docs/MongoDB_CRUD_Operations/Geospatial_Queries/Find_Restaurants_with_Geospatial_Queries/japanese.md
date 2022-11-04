# Find Restaurants with Geospatial Queries
## Overview
MongoDB の地理空間インデックスを使うと、地理空間上の図形や点を含むコレクションに対して空間クエリを効率的に実行できます。地理空間機能の機能を紹介し、さまざまなアプローチを比較するために、このチュートリアルでは簡単な地理空間アプリケーション用のクエリを書く手順を説明します。

このチュートリアルでは、地理空間インデックスの概念を簡単に紹介し、$geoWithin、$geoIntersects、および $nearSphere を使用してその使用法を実演します。

ユーザーがニューヨーク市のレストランを検索するのに役立つモバイルアプリケーションを設計しているとします。このアプリケーションでは、次のことが必要です。

geoIntersects を使ってユーザーの現在の居住地を決定する

geoWithin を使って、その近所にあるレストランの数を表示する。

nearSphere を使って、ユーザーから指定された距離内にあるレストランを検索する。

このチュートリアルでは、2dsphere インデックスを使用して、このデータを球面形状でクエリすることにします。

球形と平面のジオメトリの詳細については、「地理空間モデル」を参照してください。

## Distortion
球面幾何学は、地球のような3次元の球体を平面に投影するという性質上、地図上に可視化すると歪んで見える。

例えば、経度緯度点（0,0）、（80,0）、（80,80）、（0,80）で定義される球状の正方形の仕様を考えてみよう。この領域が占める面積を下図に示す。

## Searching for Restaurants
### Prerequisites
サンプルデータセットは以下からダウンロードしてください。
https://raw.githubusercontent.com/mongodb/docs-assets/geospatial/neighborhoods.json
 と
https://raw.githubusercontent.com/mongodb/docs-assets/geospatial/restaurants.json
. これらはそれぞれ、レストランと近隣のコレクションを含んでいる。

データセットをダウンロードした後、データベースにインポートする。

```bash
mongoimport <path to restaurants.json> -c=restaurants
mongoimport <path to neighborhoods.json> -c=neighborhoods
```

地理空間インデックスで、ほとんどの場合、$geoWithin と $geoIntersects クエリのパフォーマンスが向上します。

このデータは地理的なものなので、各コレクションに 2dsphere インデックスを作成します。
を使用して各コレクションに 2dsphere インデックスを作成します。

```bash
db.restaurants.createIndex({ location: "2dsphere" })
db.neighborhoods.createIndex({ geometry: "2dsphere" })
```

### Exploring the Data
mongoshで新しく作成されたレストランコレクションのエントリーを検査します。

```bash
db.restaurants.findOne()
```

このクエリは、以下のような文書を返す。

```json
{
   location: {
      type: "Point",
      coordinates: [-73.856077, 40.848447]
   },
   name: "Morris Park Bake Shop"
}
```

このレストランの資料は、下図の場所に対応しています。

このチュートリアルでは2dsphereインデックスを使用しているため、locationフィールドのジオメトリデータはGeoJSON形式に従っている必要があります。

ここで、neighborhoods コレクションのエントリーを検査します。

```bash
db.neighborhoods.findOne()
```

このクエリは、以下のような文書を返します。

```json
{
   geometry: {
      type: "Polygon",
      coordinates: [[
         [ -73.99, 40.75 ],
         ...
         [ -73.98, 40.76 ],
         [ -73.99, 40.75 ]
      ]]
    },
    name: "Hell's Kitchen"
}
```

この形状は、下図に示す領域に相当する。

### Find the Current Neighborhood
ユーザーのモバイルデバイスがある程度正確な位置情報を提供できると仮定すると、$geoIntersects を使ってユーザーの現在の近隣地域を簡単に見つけることができます。

ユーザーが経度 -73.93414657 ，緯度 40.82302903 に位置しているとします。現在の近隣地域を見つけるには、GeoJSON 形式の特別な $geometry フィールドを使用してポイントを指定します。

```bash
db.neighborhoods.findOne({ geometry: { $geoIntersects: { $geometry: { type: "Point", coordinates: [ -73.93414657, 40.82302903 ] } } } })
```

このクエリーは次のような結果を返します。

```json
{
    "_id" : ObjectId("55cb9c666c522cafdb053a68"),
    "geometry" : {
        "type" : "Polygon",
        "coordinates" : [
            [
                [
                    -73.93383000695911,
                    40.81949109558767
                ],
                ...
            ]
        ]
    },
    "name" : "Central Harlem North-Polo Grounds"
}
```

### Find all Restaurants in the Neighborhood
また、指定された地域に含まれるすべてのレストランを検索するクエリも可能です。
mongoshで次のように実行して、ユーザーを含む近隣地域を見つけ、その近隣地域にあるレストランを数えます。

```bash
var neighborhood = db.neighborhoods.findOne( { geometry: { $geoIntersects: { $geometry: { type: "Point", coordinates: [ -73.93414657, 40.82302903 ] } } } } )
db.restaurants.find( { location: { $geoWithin: { $geometry: neighborhood.geometry } } } ).count()
```

このクエリは、次の図で可視化されているように、リクエストされた近隣に127のレストランがあることを教えてくれます。

### Find Restaurants within a Distance
指定した地点から指定した距離内にあるレストランを検索するには、$geoWithin と $centerSphere を使用して結果をソートせずに返すか、$nearSphere と $maxDistance を使用して結果を距離でソートして返すことができます。

### Unsorted with $geoWithin
円形領域内のレストランを探すには、$geoWithin を $centerSphere と一緒に使います。centerSphere は MongoDB 固有の構文で、中心と半径をラジアン単位で指定することで円形領域を表します。

geoWithin は特定の順番でドキュメントを返すわけではないので、ユーザーに一番遠いドキュメントを最初に表示することもあります。

以下は、ユーザーから 5 マイル以内にあるすべてのレストランを検索します。

```bash
db.restaurants.find({ location:
   { $geoWithin:
      { $centerSphere: [ [ -73.93414657, 40.82302903 ], 5 / 3963.2 ] } } })
```

centerSphereの第2引数は半径をラジアン単位で受け取るので、それを地球の半径（マイル）で割る必要があります。距離単位の変換について詳しくは、「球面幾何学を用いた距離の計算」を参照してください。

### Sorted with $nearSphere
また、$nearSphereを使用して、$maxDistanceの項をメートル単位で指定することもできます。これは、ユーザーから 5 マイル以内にあるすべてのレストランを、最も近いところから遠いところへ順に並べ替えて返します。

```bash
var METERS_PER_MILE = 1609.34
db.restaurants.find({ location: { $nearSphere: { $geometry: { type: "Point", coordinates: [ -73.93414657, 40.82302903 ] }, $maxDistance: 5 * METERS_PER_MILE } } })
```
