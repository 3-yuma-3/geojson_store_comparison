# $center
## Definition
center 演算子は、$geoWithin クエリで円を指定します。このクエリは、円の境界内にあるレガシー座標のペアを返します。この演算子は、GeoJSON オブジェクトを返しません。

center 演算子を使用するには、以下を含む配列を指定します。

円の中心点のグリッド座標。

座標系で使用される単位で測定された、円の半径。

経度と緯度を使用する場合は、先に経度を指定します。

## Behavior
このクエリは、平面 (プレーナー) ジオメトリを使用して距離を計算します。

アプリケーションは、地理空間インデックスを持たずに $center を使用できます。ただし、地理空間インデックスを使用すると、インデックスがない同等品よりもはるかに高速なクエリをサポートします。

2d 地理空間インデックスのみが $center をサポートします。

## Example
以下のクエリ例では、[ -74, 40.74 ]を中心とした半径10の円内に存在する座標を持つ文書をすべて返します。

```bash
db.places.find(
   { loc: { $geoWithin: { $center: [ [-74, 40.74], 10 ] } } }
)
```
