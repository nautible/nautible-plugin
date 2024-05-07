# オブザーバビリティ

## 導入エコシステム

オブザーバビリティでは Grafana プロジェクトをベースとしたモニタリング/ロギング/トレーシングの設定を行う。

- Monitoring
  - Grafana Mimir（※ローカル実行時は Prometheus）
- Logging
  - Grafana Loki
- Tracing
  - Grafana Tempo
- Exporter
  - prometheus-node-exporter
  - kube-state-metrics
- OpenTelemetry Collector
  - Grafana Alloy
- View
  - Grafana

## フォルダ構成

```text
observation
  ├─ cloud                 ・・・クラウド環境にデプロイするためのマニフェスト
  │  ├─ base               ・・・クラウド環境に依存しない設定を配置
  │  │    ├─ alloy         ・・・Grafana Alloy
  │  │    ├─ grafana       ・・・Grafana
  │  │    ├─ loki          ・・・Grafana Loki
  │  │    ├─ otel          ・・・OpenTelemetry Operator
  │  │    ├─ prometheus    ・・・node-exporter,kube-state-metrics
  │  │    ├─ tempo
  │  │    │  variables.tf　・・・開発用の設定値
  │  └─ overlays           ・・・環境別のパッチを配置
  │       └─  aws          ・・・AWS用の設定値
  ├─ common                ・・・Daemonsetを優先配置するためのPriorityClass
  ├─ docs                  ・・・ドキュメント一式
  ├─ examples
  │  └─ instrumentation    ・・・アプリケーションの自動計装を有効化するためのサンプルマニフェスト
  ├─ local
  │   └─ manifests         ・・・ローカル環境にデプロイするためのマニフェスト
  └─ README.md             ・・・README(本ファイル)
```

## 導入手順

[ローカル環境](./docs/LOCAL.md)

[AWS](./docs/AWS.md)

Azure（TODO）

## OpenTelemetry

モニタリング情報、ロギング情報、トレース情報の収集構成には OpenTelemetry を利用している。

OpenTelemetry は開発言語ごとにサポート状況が異なるため、最新の状態は[公式サイト](https://opentelemetry.io/docs/languages/)を参照。

## アプリケーションへの OpenTelemetry 導入

アプリケーションへの OpenTelemetry 機能導入には Auto Instrumentation 機能を利用して実現している。

### Auto Instrumentation

マニフェストファイルにアノテーションを付与しておくことで、デプロイ時に OpenTelemetry Operator が自動的に自動計測ライブラリを挿入する機能。

![auto instrumentation](./docs/autoinstrumentation.png)

参考：[Auto Instrumentation](https://opentelemetry.io/docs/kubernetes/operator/automatic/)

## 永続化

ローカル環境モードでは外部ストレージへの永続化を行わないため、Pod の削除でデータは削除される。

クラウド環境では各クラウドのオブジェクトストレージに永続化を行う。（Grafana Mimir / Grafana Loki / Grafana Tempo）

※Grafana はボリュームストレージをノードにアタッチして永続化する
