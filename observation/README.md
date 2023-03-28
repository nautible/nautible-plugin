# 監視系（モニタリング/ロギング/トレーシング）

フォルダ構成

```text
observation
├ docs                      :ドキュメント
├ examples                  :ServiceMonitor,PodMonitor,PrometheusRuleの導入例
├ manifests
│ ├ base
│ │ ├ common                :共通利用マニフェスト（PriorityClass） 
│ │ ├ crd                   :カスタムリソース
│ │ ├ kube-prometheus-stack :kube-prometheus-stackデプロイ用マニフェスト
│ │ ├ promtail              :Promtailデプロイ用マニフェスト
│ │ └ kustomization.yaml    :base配下一括導入用kustomizeファイル
│ └ overlays
│   ├ aws
│   │ ├ loki                :GrafanaLokiデプロイ用マニフェスト 
│   │ ├ application.yaml    :AWS一括導入用applicationマニフェスト
│   │ └ kustomization.yaml  :AWS一括導入用kustomizeファイル
│   └ azure
│     ├ loki                :GrafanaLokiデプロイ用マニフェスト 
│     ├ application.yaml    :Azure一括導入用applicationマニフェスト
│     └ kustomization.yaml  :Azure一括導入用kustomizeファイル
└ README.md                 :ドキュメント（本ファイル）
```

## [セットアップ](./docs/setup.md)

- エコシステム（Prometheus/Grafana/GrafanaLoki/Promtail）をKubernetesに導入する

## [監視対象のカスタマイズ](./docs/custom-metrics.md)

- アプリケーション独自のメトリクスをPrometheusの監視対象に追加する

## [監視ルールのカスタマイズ](./docs/custom-rule.md)

- 監視ルールを追加する
- デフォルトの監視ルールを無効化する

## [ロギング](./docs/logging.md)

- Grafanaでログを確認する
- エラーログなどを検知して、AlertManagerからSlackへ通知する
