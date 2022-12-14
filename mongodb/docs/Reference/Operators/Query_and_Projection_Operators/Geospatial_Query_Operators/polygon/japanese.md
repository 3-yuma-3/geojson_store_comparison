# $polygon
## Definition
レガシー座標ペアの地理空間 $geoWithin クエリで使用する多角形を指定します。このクエリは、ポリゴンの境界内にあるペアを返します。この演算子は、GeoJSON オブジェクトに対してクエリーを実行しません。

多角形を定義するには、座標点の配列を指定します。

```bash
{
   <location field>: {
      $geoWithin: {
         $polygon: [ [ <x1> , <y1> ], [ <x2> , <y2> ], [ <x3> , <y3> ], ... ]
      }
   }
}
```

最後の点は常に最初の点と暗黙のうちに接続される。点、すなわち辺はいくつでも指定できる。

経度と緯度を使用する場合は、先に経度を指定します。

## Behavior
polygon 演算子は、平面幾何学を使って距離を計算します。

アプリケーションは、地理空間インデックスを持たずに $polygon を使用することができます。ただし、地理空間インデックスを使用すると、インデックスがない同等品よりもはるかに高速なクエリーをサポートします。

2d 地理空間インデックスのみが $polygon 演算子をサポートします。

## Example
次の問い合わせは、[ 0 , 0 ]、[ 3 , 6 ]、[ 6 , 0 ]で定義される多角形の中に存在する座標を持つすべての文書を返す。

```bash
db.places.find(
  {
     loc: {
       $geoWithin: { $polygon: [ [ 0 , 0 ], [ 3 , 6 ], [ 6 , 0 ] ] }
     }
  }
)
```
