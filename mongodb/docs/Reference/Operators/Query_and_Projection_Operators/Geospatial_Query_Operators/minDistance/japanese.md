# $minDistance
## Definition
地理空間 $near または $nearSphere クエリの結果を、中心点から指定した距離以上離れているドキュメントにフィルタリングします。

near または $nearSphere クエリで中心点を GeoJSON ポイントとして指定する場合は、距離を非負の数値 (メートル単位) として指定します。

nearSphere クエリで中心点をレガシー座標のペアとして指定する場合は、距離を非負の数値 (ラジアン) で指定します。near は、クエリで中心点が GeoJSON ポイントとして指定されている場合にのみ、2dsphere インデックスを使 用できます。

## Examples
### Use with $near
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

### Use with $nearSphere
locationフィールドを持つ文書を含み、2dsphereインデックスを持つコレクションplacesを考える。

すると、次の例では、指定された地点から少なくとも1000メートル、最大5000メートルの距離にある人の位置を、近い順に返す。

```bash
db.places.find(
   {
     location: {
        $nearSphere: {
           $geometry: {
              type : "Point",
              coordinates : [ -73.9667, 40.78 ]
           },
           $minDistance: 1000,
           $maxDistance: 5000
        }
     }
   }
)
```

中心点をレガシー座標ペアとして指定する例の場合, see $nearSphere
