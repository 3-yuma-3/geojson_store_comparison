# Build a map to compare metrics by country or region
マップを初めて使う方は、このチュートリアルから始めるとよいでしょう。このチュートリアルでは、位置情報を使用するための一般的な手順を説明します。

このチュートリアルでは、次のことを学びます。

1. 複数のレイヤーとデータソースを含むマップを作成する
2. シンボル、色、およびラベルを使用して、データ値をスタイル設定する。
3. ダッシュボードにマップを埋め込む
4. ダッシュボードのパネルで検索する

このチュートリアルを完了すると、次のようなマップが作成されます。

## Prerequisites
- Kibana をまだお持ちでない場合は、無料トライアルでセットアップしてください。
- このチュートリアルでは、Web ログのサンプル データセットが必要です。サンプルデータには、このチュートリアルで再作成する [Logs] Total Requests and Bytes マップが含まれています。
- マップを作成するためには、適切な権限が必要です。マップを作成または保存するための十分な権限がない場合、ツールバーに読み取り専用アイコンが表示されま す。詳細については、「Kibana へのアクセス権を付与する」を参照してください。

## Step 1. Create a map
1. メインメニューを開き、[ダッシュボード]をクリックします。
2. ダッシュボードの作成］をクリックします。
3. 時間範囲を「過去7日間」に設定します。
4. 新規マップの作成アイコンアプリのgisアイコンをクリックします。

## Step 2. Add a choropleth layer
最初に追加するレイヤーは、ウェブログのトラフィックによって世界の国々を陰影で表現するコレオプレットレイヤーです。暗い影は、ウェブログのトラフィックが多い国を象徴し、明るい影は、トラフィックが少ない国を象徴します。

1. レイヤーを追加]をクリックし、[Choropleth]をクリックします。
2. EMS 境界のドロップダウンメニューから、「世界の国々」を選択します。
3. Statistics source で、設定します。
  - データ ビューを kibana_sample_data_logs に設定します。
  - フィールドを geo.dest に結合します。
4. レイヤーの追加] をクリックします。
5. レイヤーの設定] で、以下を設定します。
  - 名前] を [宛先別総リクエスト数] に設定
  - 不透明度を50%に設定
6. Tooltipフィールドを追加します。
  - デフォルトでISO 3166-1 alpha-2コードが追加されています。
  - 追加]をクリックすると、フィールドの選択画面が表示されます。
  - 名前を選択し、[追加]をクリックします。
7. レイヤースタイルで
  - レイヤースタイルで：塗りつぶしの色＞数字としてをグレー色のランプに設定します。
  - ボーダーカラーを白に設定します。
  - ラベル]の[バイ]の値を[固定]に変更します。
8. 保存して閉じる]をクリックします。

これで地図は次のようになります。

## Step 3. Add layers for the Elasticsearch data
一度に多くのデータでユーザを圧倒しないように、Elasticsearchのデータ用に2つのレイヤーを追加します。最初のレイヤーは、ユーザーがマップをズームインしたときに個々のドキュメントを表示します。2つ目のレイヤーは、ユーザーがマップをズームアウトしたときに、集約されたデータを表示します。

### Add a layer for individual documents
このレイヤーは、ウェブログ文書を点として表示します。このレイヤーは、ユーザーがズームインしたときにのみ表示されます。

1. レイヤーの追加] をクリックし、[ドキュメント] をクリックします。
2. データ ビューを kibana_sample_data_logs に設定します。
3. レイヤーの追加] をクリックします。
4. レイヤーの設定] で、以下を設定します。
  - 名前] を [実際のリクエスト] に設定します。
  - 可視性を範囲 [9, 24] に設定します。
  - 不透明度を100%に設定
5. ツールチップフィールドを追加し、agent, bytes, clientip, host, machine.os, request, response, and timestamp を選択します。
6. スケーリング]で、[結果を10,000に制限]を有効にします。
7. レイヤースタイル］で、［塗りつぶしの色］を#2200FFに設定します。
8. 保存して閉じる]をクリックします。

マップは、ズームレベル9から24でこのように表示されます。

### Add a layer for aggregated data
集計データのレイヤーを作成し、マップをズームアウトしたときにのみ表示されるようにします。暗い色は、Web ログのトラフィックが多いグリッドを、明るい色は、トラフィックが少ないグリッドを象徴しています。大きな円は、転送された総バイト数が多いグリッドを、小さな円は、転送されたバイト数が少ないグリッドを象徴しています。

1. レイヤーの追加] をクリックし、[クラスター] を選択します。
2. データ］ビューを［kibana_sample_data_logs］に設定します。
3. レイヤーの追加] をクリックします。
4. レイヤーの設定] で、設定します。
  - 名前] を [総リクエスト数] と [バイト数] に設定します。
  - 可視性を範囲 [0, 9] に設定します。
  - 不透明度を100%に設定
5. メトリックス]で
  - Aggregation（集計）］を［Count］に設定します。
  - Add metric]をクリックします。
  - 集計]を[合計]に設定し、[フィールド]を[バイト]に設定します。
6. Layer style]で[Symbol size]を変更します。
  - By value］を［sum bytes］に設定します。
  - 最小サイズを7、最大サイズを25pxに設定します。
7. 保存して閉じる]ボタンをクリックします。

ズームレベル0から9の間、地図はこのように表示されます。

## Step 4. Save the map
マップが完成したら、保存してダッシュボードに戻ります。

1. ツールバーで、[保存して戻る] をクリックします。

## Step 5. Explore your data from the dashboard
地理空間データをヒートマップや円グラフと並べて表示し、データをフィルタリングすることができます。あるパネルでフィルタを適用すると、ダッシュボード上のすべてのパネルに適用されます。

1. ライブラリから追加をクリックすると、ダッシュボードに追加できるパネルのリストが表示されます。
2. Logs] Unique Destination Heatmapと[Logs] Bytes distributionをダッシュボードに追加します。
3. バイト値が異常に高い文書をフィルタリングするには、バイト分布図内でクリック＆ドラッグします。4. フィルタバーの名前の横の×をクリックして、フィルタを削除します。
5. マップからフィルタを設定する。
  a. 米国ベクトル内の任意の場所をクリックすると、ツールチップが表示されます。
  b. geo.srcがUSの文書だけを表示するには、ISO 3066-1 alpha-2の行のフィルターアイコン フィルターアイコンをクリックします。

フィルタリングされたマップは、次のように表示されます。

## What’s next?
- 地図に追加できるレイヤーの種類を確認する。
- マップをカスタマイズする方法について詳しく説明します。
- ベクターツールチップについて
