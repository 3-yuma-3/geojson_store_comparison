# Geospatial Query Operators
地理空間演算子は、地理空間表現条件に基づいてデータを返します。

特定の演算子の構文や例など、詳細については、その演算子のリファレンスページへのリンクをクリックしてください。

## Operators
### Query Selectors
- `$geoIntersects`
  - GeoJSON ジオメトリと交差するジオメトリを選択します。2dsphere インデックスは $geoIntersects をサポートします。
- `$geoWithin`
  - バウンディング GeoJSON ジオメトリ内にあるジオメトリを選択します。2dsphere および 2d インデックスは、$geoWithin をサポートします。
- `$near`
  - ある点に近接している地理空間オブジェクトを返します。地理空間インデックスが必要です。2dsphere と 2d インデックスは $near をサポートしています。
- `$nearSphere`
  - 球体上のある点に近接する地理空間オブジェクトを返します。地理空間インデックスが必要です。2dsphere と 2d インデックスは $nearSphere をサポートしています。

### Geometry Specifiers
- `$box`
  - geoWithin クエリで、レガシーな座標ペアを使用した長方形のボックスを指定します。2d インデックスでは、$box をサポートします。
- `$center`
  - 平面ジオメトリを使用する際に、$geoWithin クエリでレガシー座標ペアを使用して円を指定します。2d インデックスは $center をサポートしています。
- `$centerSphere`
  - 球形ジオメトリを使用する場合に、$geoWithin クエリでレガシー座標ペアまたは GeoJSON 形式を使用した円を指定します。2dsphere と 2d インデックスは、$centerSphere をサポートします。
- `$geometry`
  - 地理空間クエリ演算子で、GeoJSON 形式のジオメトリを指定します。
- `$maxDistance`
  - near および $nearSphere クエリの結果を制限するための最大距離を指定します。2dsphere および 2d インデックスは、$maxDistance をサポートします。
- `$minDistance`
  - near および $nearSphere クエリの結果を制限する最小距離を指定します。2dsphere インデックスでのみ使用できます。
- `$polygon`
  - geoWithin クエリでレガシー座標ペアを使用するための多角形を指定します。2d インデックスでは、$center がサポートされています。
