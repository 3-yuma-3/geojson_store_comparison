# Query a 2dsphere Index
以下のセクションでは、2dsphere インデックスでサポートされているクエリについて説明します。

## GeoJSON Objects Bounded by a Polygon
geoWithin 演算子は、GeoJSON ポリゴン内で見つかった位置データを照会します。位置データは、GeoJSON 形式で保存されている必要があります。次のような構文を使用します。

```bash
db.<collection>.find( { <location field> :
                         { $geoWithin :
                           { $geometry :
                             { type : "Polygon" ,
                               coordinates : [ <coordinates> ]
                      } } } } )
```

次の例では、GeoJSONポリゴン内に完全に存在するすべての点と図形を選択します。

```bash
db.places.find( { loc :
                  { $geoWithin :
                    { $geometry :
                      { type : "Polygon" ,
                        coordinates : [ [
                                          [ 0 , 0 ] ,
                                          [ 3 , 6 ] ,
                                          [ 6 , 1 ] ,
                                          [ 0 , 0 ]
                                        ] ]
                } } } } )
```

## Intersections of GeoJSON Objects
geoIntersects演算子は、指定されたGeoJSONオブジェクトと交差する位置の検索を行います。交差点が空でない場合、その場所はオブジェクトと交差しています。これには、共有エッジを持つドキュメントが含まれます。

geoIntersects演算子は、次の構文を使用します。

```bash
db.<collection>.find( { <location field> :
                         { $geoIntersects :
                           { $geometry :
                             { type : "<GeoJSON object type>" ,
                               coordinates : [ <coordinates> ]
                      } } } } )
```

次の例では、$geoIntersects を使って、座標配列で定義された多角形と交差するインデックス付きの点および図形をすべて選択しています。

```bash
db.places.find( { loc :
                  { $geoIntersects :
                    { $geometry :
                      { type : "Polygon" ,
                        coordinates: [ [
                                         [ 0 , 0 ] ,
                                         [ 3 , 6 ] ,
                                         [ 6 , 1 ] ,
                                         [ 0 , 0 ]
                                       ] ]
                } } } } )
```

## Proximity to a GeoJSON Point
近接クエリは、定義されたポイントに最も近いポイントを返し、その結果を距離でソートします。GeoJSON データに対する近接クエリには、2dsphere インデックスが必要です。

GeoJSON ポイントへの近接をクエリするには、$near 演算子を使用します。距離はメートル単位です。

near は次のような構文を使用します。

```bash
db.<collection>.find( { <location field> :
                         { $near :
                           { $geometry :
                              { type : "Point" ,
                                coordinates : [ <longitude> , <latitude> ] } ,
                             $maxDistance : <distance in meters>
                      } } } )
```

例については、$near

nearSphere 演算子および $geoNear 集約パイプラインステージも参照してください。

## Points within a Circle Defined on a Sphere
球面上の「球面キャップ」内のすべてのグリッド座標を選択するには、$geoWithin と $centerSphere 演算子を使用します。含む配列を指定します。

- 円の中心点のグリッド座標。
- 円の半径をラジアン単位で指定します。ラジアンを計算するには、「球面形状を使用した距離の計算」を参照してください。

次の構文を使用します。

```bash
db.<collection>.find( { <location field> :
                         { $geoWithin :
                           { $centerSphere :
                              [ [ <x>, <y> ] , <radius> ] }
                      } } )
```

次の例は、グリッド座標を照会し、西経88度、北緯30度の半径10マイル以内の文書をすべて返します。この例では、距離10マイルを地球の近似赤道半径3963.2マイルで割ってラジアンに変換しています。

```bash
db.places.find( { loc :
                  { $geoWithin :
                    { $centerSphere :
                       [ [ -88 , 30 ] , 10 / 3963.2 ]
                } } } )
```
