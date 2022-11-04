# $maxDistance
## Definition
maxDistance 演算子は、地理空間 $near または $nearSphere のクエリの結果を、指定された距離に制限します。最大距離の測定単位は、使用中の座標系によって決まります。GeoJSON 点オブジェクトの場合、ラジアンではなくメートル単位で距離を指定します。maxDistanceには、負でない数値を指定する必要があります。

2dsphere と 2d 地理空間インデックスの両方が $maxDistance: をサポートしています。

## Example
以下のクエリ例では、点[ -74 , 40 ]から10単位以下の位置値を持つ文書を返します。

```bash
db.places.find( {
   loc: { $near: [ -74 , 40 ],  $maxDistance: 10 }
} )
```

MongoDB は、結果を [ -74 , 40 ] からの距離で並べ替えます。この操作では、cursor.limit() メソッドでクエリを変更しない限り、最初の 100 件の結果が返されます。
