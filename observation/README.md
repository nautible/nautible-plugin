# 監視系（モニタリング/ロギング/トレーシング）

フォルダ構成

```
observation
├docs                :ドキュメント 
├loki                :GrafanaLokiデプロイ用マニフェスト 
├prometheus-operator :kube-prometheus-stackデプロイ用マニフェスト
├promtail            :Promtailデプロイ用マニフェスト
├rules               :Prometheusの設定を行うカスタムリソースデプロイ用マニフェスト
├servicemonitors     :ServiceMonitorデプロイ用マニフェスト
├namespace.yaml      :ネームスペース（名前：observation）デプロイ用マニフェスト
└README.md           :本ファイル
```

## [セットアップ](./docs/setup.md)

- 監視系ツール（Prometheus/Alertmanager/Grafana/GrafanaLoki etc）の導入

## [監視対象のカスタマイズ](./docs/custom-metrics.md)

- アプリケーション独自のメトリクスをPrometheusの監視対象に追加する

## [監視ルールのカスタマイズ](./docs/custom-rule.md)

- 監視ルールを追加する
- デフォルトの監視ルールを無効化する

## [ロギング](./docs/logging.md)

- Grafanaでログを確認する
- エラーログなどを検知して、AlertManagerからSlackへ通知する
