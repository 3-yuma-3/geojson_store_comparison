# geojson_store_comparison

## 無償提供geojson
- [国土数値情報ダウンロードサービス](https://nlftp.mlit.go.jp/ksj/)
  - [農業地域](https://nlftp.mlit.go.jp/ksj/gml/datalist/KsjTmplt-A12.html)
    - `非商用`: 出典・加工者等表示のうえ、原著作者等の許諾上、非商用目的のみでの利用（ただし複製物の再配布を除く）が可能なもの
  - [行政区域データ](https://nlftp.mlit.go.jp/ksj/gml/datalist/KsjTmplt-N03-v3_1.html)
    - `商用可`: 出典・加工者等表示のうえ、原著作者等の許諾上、商用目的での利用（複製物の再配布を含む）が可能なもの
- [smartnews-smri / japan-topography](https://github.com/smartnews-smri/japan-topography)
  - `クレジット`
    - データを使用する際に、加工者としてスマートニュースおよびスマートニュース メディア研究所の名前をクレジットする必要はありません。
    - 商用・非商用にかかわらず、無償でお使いいただけます。
    - ただし市区町村データは、国土交通省の指示するクレジット記載が必要です。
その他、詳細は各データソースの利用規約を参照してください。
  - `利用事例ご報告のお願い`
  -   利用実態を把握するため、市区町村・選挙区地形データ 利用事例ご報告フォームまでご報告をお願いしております。
    - データの利用にあたって回答は必須ではありませんが、ご協力いただければ幸いです。
    - なお、ご報告に対して弊社から返信や許可・不許可を行うことは原則としてありません。

## gdal
- [GDAL documentation](https://gdal.org/index.html)
  - [GDAL documentation » Vector drivers » PostgreSQL / PostGIS](https://gdal.org/drivers/vector/pg.html)
- `$ docker pull osgeo/gdal:ubuntu-small-3.5.3`
- [Use GDAL With Docker](https://tannergeo.com/2017/10/05/Use-GDAL-With-Docker.html)
- [Using ogr2ogr to convert data between GeoJSON, PostGIS and Esri Shapefile](https://morphocode.com/using-ogr2ogr-convert-data-formats-geojson-postgis-esri-geodatabase-shapefiles/)

## その他
- [GIS実習オープン教材](https://gis-oer.github.io/gitbook/book/)
