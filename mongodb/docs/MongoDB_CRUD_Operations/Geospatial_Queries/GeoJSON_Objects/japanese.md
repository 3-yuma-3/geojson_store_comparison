# GeoJSON Objects
## Overview
MongoDB は、このページに記載されている GeoJSON オブジェクトの種類をサポートしています。

GeoJSON データを指定するには、次のような埋め込みドキュメントを使用します。

- GeoJSON オブジェクトの種類を指定する type というフィールドと
- 座標というフィールドで、オブジェクトの座標を指定します。

緯度と経度の座標を指定する場合は、まず経度を記載し、次に緯度を記載します。

- 有効な経度の値は-180から180の間です。
- 緯度の有効範囲は-90〜90です。

```bash
<field>: { type: <GeoJSON type> , coordinates: <coordinates> }
```

MongoDB の GeoJSON オブジェクトの地理空間クエリーは球体上で計算します。MongoDB は GeoJSON オブジェクトの地理空間クエリーに WGS84 参照システムを使用します。

## Point
次の例では、GeoJSONを指定しています。Point

```bash
{ type: "Point", coordinates: [ 40, 5 ] }
```

## LineString
次の例では、GeoJSONを指定しています。LineString

```bash
{ type: "LineString", coordinates: [ [ 40, 5 ], [ 41, 6 ] ] }
```

## Polygon
ポリゴンは、GeoJSON LinearRing座標配列の配列から構成されます。これらのLinearRingは閉じたLineStringsです。閉じたLineStringは、少なくとも4つの座標ペアを持ち、最初と最後の座標として同じ位置を指定する。

曲面上の2点を結ぶ線は、平面上のその2点を結ぶ座標と同じ組を含むこともあれば、含まないこともある。曲面上の2点を結ぶ線は測地線となる。エッジの共有やオーバーラップなどのエラーが発生しないように、慎重にポイントをチェックします。

### Polygons with a Single Ring
次の例では、外側のリングと内側のリング（または穴）を持たないGeoJSONポリゴンを指定しています。ポリゴンを閉じるには、最初と最後の座標が一致する必要があります。

```json
{
  type: "Polygon",
  coordinates: [ [ [ 0 , 0 ] , [ 3 , 6 ] , [ 6 , 1 ] , [ 0 , 0  ] ] ]
}
```

リングが1つのPolygonの場合、リングは自己交差することができません。

### Polygons with Multiple Rings
複数のリングを持つPolygonsの場合。

- 最初に記述されたリングは外側のリングでなければならない。
- 外側のリングは自己交差することはできない。
- 内側のリングは、外側のリングに完全に含まれていなければならない。
- 内側のリングは互いに交差したり、重なり合うことはできない。内部のリングはエッジを共有することはできない。

次の例は、内部リングを持つ GeoJSON ポリゴンを表しています。

```json
{
  type : "Polygon",
  coordinates : [
     [ [ 0 , 0 ] , [ 3 , 6 ] , [ 6 , 1 ] , [ 0 , 0 ] ],
     [ [ 2 , 2 ] , [ 3 , 3 ] , [ 4 , 2 ] , [ 2 , 2 ] ]
  ]
}
```

## MultiPoint
Requires Versions

GeoJSON MultiPoint 埋め込み文書は、ポイントのリストをエンコードしています。

```json
{
  type: "MultiPoint",
  coordinates: [
     [ -73.9580, 40.8003 ],
     [ -73.9498, 40.7968 ],
     [ -73.9737, 40.7648 ],
     [ -73.9814, 40.7681 ]
  ]
}
```

## MultiLineString
Requires Versions

次の例では、GeoJSON MultiLineString を指定しています。

```json
{
  type: "MultiLineString",
  coordinates: [
     [ [ -73.96943, 40.78519 ], [ -73.96082, 40.78095 ] ],
     [ [ -73.96415, 40.79229 ], [ -73.95544, 40.78854 ] ],
     [ [ -73.97162, 40.78205 ], [ -73.96374, 40.77715 ] ],
     [ [ -73.97880, 40.77247 ], [ -73.97036, 40.76811 ] ]
  ]
}
```

## MultiPolygon
Requires Versions

次の例では、GeoJSON MultiPolygon を指定しています。

```json
{
  type: "MultiPolygon",
  coordinates: [
     [ [ [ -73.958, 40.8003 ], [ -73.9498, 40.7968 ], [ -73.9737, 40.7648 ], [ -73.9814, 40.7681 ], [ -73.958, 40.8003 ] ] ],
     [ [ [ -73.958, 40.8003 ], [ -73.9498, 40.7968 ], [ -73.9737, 40.7648 ], [ -73.958, 40.8003 ] ] ]
  ]
}
```

## GeometryCollection
Requires Versions

次の例は、GeoJSON GeometryCollection の座標を格納するものです。

```json
{
  type: "GeometryCollection",
  geometries: [
     {
       type: "MultiPoint",
       coordinates: [
          [ -73.9580, 40.8003 ],
          [ -73.9498, 40.7968 ],
          [ -73.9737, 40.7648 ],
          [ -73.9814, 40.7681 ]
       ]
     },
     {
       type: "MultiLineString",
       coordinates: [
          [ [ -73.96943, 40.78519 ], [ -73.96082, 40.78095 ] ],
          [ [ -73.96415, 40.79229 ], [ -73.95544, 40.78854 ] ],
          [ [ -73.97162, 40.78205 ], [ -73.96374, 40.77715 ] ],
          [ [ -73.97880, 40.77247 ], [ -73.97036, 40.76811 ] ]
       ]
     }
  ]
}
```
