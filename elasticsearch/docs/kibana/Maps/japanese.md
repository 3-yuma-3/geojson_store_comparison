# Maps
地理データから美しいマップを作成 マップを使用すると、次のことが可能になります。

複数のレイヤとインデックスを持つマップを作成する。
空間的な時間データのアニメーション化
GeoJSON ファイルとシェープファイルをアップロードする。
ダッシュボードにマップを埋め込む。
データ値を使用したフィーチャのシンボル化
重要なデータのみに集中することができます。
始める準備はできましたか？ビデオをご覧になった後、スタートアップチュートリアルを使ってマップを操作してください。

## Build maps with multiple layers and indices
複数のレイヤーとインデックスを使用して、1 つの地図にすべてのデータを表示します。気象パターンなどの物理的特徴、国境などの人為的特徴、販売地域などのビジネス固有の特徴に関連するデータの位置付けを表示します。個々のドキュメントをプロットしたり、集計を使用して、どんなに大きなデータセットでもプロットすることができます。

## Animate spatial temporal data
データはアニメーションでよみがえる。静的なデータでは発見しにくいパターンが、動きによって浮かび上がってきます。時間スライダーを使用してデータをアニメーション化し、より深い洞察を得ましょう。

このアニメーションマップは、時間スライダーを使用して、15分間のポートランドのバスを表示しています。バスの位置が時間と共に更新され、ルートが生き生きとしてきます。

## Upload GeoJSON files and shapefiles
マップを使って、GeoJSONやシェイプファイルのデータをElasticsearchにドラッグ＆ドロップし、マップのレイヤーとして使用することができます。

## Embed your map in dashboards
データを様々な角度から見ることで、より良い洞察が得られます。ある可視化では不明瞭な点が、別の可視化では明らかになることがあります。ダッシュボードにマップを追加して、地理空間データを棒グラフ、円グラフ、タグクラウドなどと一緒に表示できます。

この地図は、サンディエゴの議会区ごとの非緊急サービス依頼の密度を示しています。このマップはダッシュボードに埋め込まれているため、ユーザーはサービスがいつ要請されたかをよりよく理解し、要請の多いサービスを把握することができます。

## Symbolize features using data values
各レイヤーをカスタマイズして、データの重要な側面を強調することができます。例えば、ウェブログのトラフィックが多いエリアを濃い色で、トラフィックが少ないエリアを薄い色で象徴します。

## Focus on only the data that’s important to you
マップのレイヤーを横断して検索し、必要なデータだけに焦点を当てることができます。Kibana Query Language を使用して、フリーテキスト検索とフィールドベースの検索を組み合わせます。時間フィルタを設定して、時間によってレイヤーを制限します。地図上に多角形を描くか、フィーチャの形状を使用して空間フィルタを作成します。個々のレイヤーをフィルタリングして、ファセットを比較します。