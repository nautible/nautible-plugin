# 監視系（モニタリング/ロギング/トレーシング）

フォルダ構成

```text
observation
├ common              :共通利用マニフェスト（PriorityClass） 
├ docs                :ドキュメント 
├ loki                :GrafanaLokiデプロイ用マニフェスト 
├ monitors            :ServiceMonitorデプロイ用マニフェスト
├ prometheus-operator :kube-prometheus-stackデプロイ用マニフェスト
├ promtail            :Promtailデプロイ用マニフェスト
├ rules               :Prometheusの設定を行うカスタムリソースデプロイ用マニフェスト
├ application.yaml    :observation配下一括導入用applicationマニフェスト
├ kustomization.yaml  :observation配下一括導入用kustomizeファイル
└ README.md           :本ファイル
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
