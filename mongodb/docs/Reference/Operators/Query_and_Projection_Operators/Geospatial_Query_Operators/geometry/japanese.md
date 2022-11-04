# $geometry
geometry 演算子は、次の地理空間クエリ演算子で使用する GeoJSON ジオメトリを指定します。geoWithin、$geoIntersects、$near、および $nearSphere の各ジオメトリを指定します。geometry は、デフォルトの座標参照系 (CRS) として EPSG:4326 を使用します。

デフォルトの CRS を持つ GeoJSON オブジェクトを指定するには、$geometry に次のプロトタイプを使用します。

```bash
$geometry: {
   type: "<GeoJSON object type>",
   coordinates: [ <coordinates> ]
}
```

MongoDB のカスタム CRS でシングルリングの GeoJSON ポリゴンを指定するには、次のプロトタイプを使います ($geoWithin および $geoIntersects のみで使用可能です)。

```bash
$geometry: {
   type: "Polygon",
   coordinates: [ <coordinates> ],
   crs: {
      type: "name",
      properties: { name: "urn:x-mongodb:crs:strictwinding:EPSG:4326" }
   }
}
```

MongoDBのカスタム座標参照系は、厳密に反時計回りの巻き順になっています。

緯度と経度の座標を指定する場合は、まず経度を記載し、次に緯度を記載します。

- 有効な経度の値は-180から180の間です。
- 緯度の有効範囲は-90〜90です。
