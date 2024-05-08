# オブザーバビリティ

## 1. 導入エコシステム

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

## 2. フォルダ構成

```text
observation
  ├─ base                ・・・環境に依存しない設定を配置
  │    ├─ alloy          ・・・Grafana Alloy
  │    ├─ common         ・・・Daemonsetを優先配置するためのPriorityClass
  │    ├─ grafana        ・・・Grafana
  │    ├─ loki           ・・・Grafana Loki
  │    ├─ otel           ・・・OpenTelemetry Operator
  │    └─ tempo          ・・・Grafana Tempo導入
  ├─ overlays            ・・・環境別のパッチを配置
  │    ├─ aws            ・・・AWS用の設定値
  │    │   ├─ alloy      ・・・Grafana Alloy用設定ファイル
  │    │   ├─ grafana    ・・・Grafana パラメータ
  │    │   ├─ loki       ・・・Grafana Lokiパラメータ
  │    │   ├─ mimir      ・・・Grafana Mimirパラメータ
  │    │   ├─ prometheus ・・・kube-statemetrics,prometheus-node-exporter導入
  │    │   └─ tempo      ・・・Grafana Tempoパラメータ
  │    └─ local          ・・・ローカル（Minikube）用の設定値
  │        ├─ alloy      ・・・Grafana Alloy用設定ファイル
  │        ├─ grafana    ・・・Grafana パラメータ
  │        ├─ loki       ・・・Grafana Lokiパラメータ
  │        └─ prometheus ・・・Prometheus導入（kube-statemetrics,prometheus-node-exporterも同梱）
  ├─ docs                ・・・ドキュメント一式
  ├─ examples
  │  └─ instrumentation  ・・・アプリケーションの自動計装を有効化するためのサンプルマニフェスト
  └─ README.md           ・・・README(本ファイル)
```

## 3. エコシステムの導入手順

[ローカル環境](./docs/LOCAL.md)

[AWS](./docs/AWS.md)

Azure（TODO）

## 4. アプリケーションへの OpenTelemetry 導入

アプリケーションへの OpenTelemetry 機能導入には Auto Instrumentation 機能を利用して実現する。

### 4.1 Auto Instrumentation

マニフェストファイルにアノテーションを付与しておくことで、デプロイ時に OpenTelemetry Operator が自動的に自動計測ライブラリを挿入する機能。

![auto instrumentation](./docs/autoinstrumentation.png)

参考：[Auto Instrumentation](https://opentelemetry.io/docs/kubernetes/operator/automatic/)

### 4.2 Auto Instrumentation の有効化

計測を行う namespace に対して Instrumentation リソースをデプロイする。  
サンプルは nautible-app-ms に対して有効化している。

```bash
kubectl apply -f observation/examples/instrumentation/nautible-app-ms.yaml
```

### 4.3 アプリケーションへのアノテーション付与

自動計装ライブラリを挿入するアプリケーションの Deployment に instrumentation.opentelemetry.io/inject-<言語名> アノテーションを付与する。

[nautible-app-examples-java](https://github.com/nautible/nautible-app-examples-manifest/blob/main/nautible-app-examples-manifest-java/base/examples-deployment.yaml) の例

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nautible-app-examples-java
  namespace: nautible-app-examples
...
spec:
  replicas: 2
  selector:
    matchLabels:
      app.kubernetes.io/instance: nautible-app-examples-java
      app.kubernetes.io/component: app
  template:
    metadata:
      annotations:
        instrumentation.opentelemetry.io/inject-java: 'true'
...
```

Java 以外のアノテーションについては[公式ドキュメント](https://opentelemetry.io/docs/kubernetes/operator/automatic/#add-annotations-to-existing-deployments)を参照

### 4.4 アプリケーションのデプロイ

上記修正したマニフェストをデプロイする。

### 参考） OpenTelemetry

モニタリング情報、ロギング情報、トレース情報の収集構成には OpenTelemetry を利用している。

OpenTelemetry は開発言語ごとにサポート状況が異なるため、最新の状態は[公式サイト](https://opentelemetry.io/docs/languages/)を参照。

## Grafana からの接続設定

| データソース        | ローカル                                    | AWS                      |
| :------------------ | :------------------------------------------ | :----------------------- |
| Mimir（Prometheus） | http://mimir-query-frontend:8080/prometheus | http://prometheus-server |
| Loki                | http://loki-gateway                         | http://loki-gateway      |
| Tempo               | http://tempo:3100                           | http://tempo:3100        |
