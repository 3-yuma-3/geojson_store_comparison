# Logs settings in Kibana
KibanaでLogsアプリを使用するために、何か設定をする必要はありません。デフォルトで有効になっています。

## General Logs Settings
- `xpack.infra.sources.default.fields.message`
  - Logsアプリでメッセージを表示するために使用するフィールドです。デフォルトは ['message', '@message'] です。
- `xpack.infra.alerting.inventory_threshold.group_by_page_size`
  - インベントリ閾値が全ホストを取得するために使用する複合集計のサイズを制御します。デフォルトは10_000です。
- `xpack.infra.alerting.metric_threshold.group_by_page_size`
  - Metric Thresholdの機能別グループ化で使用する複合集計のサイズを制御します。デフォルトは10_000です。
