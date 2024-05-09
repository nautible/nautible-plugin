# nautible-plugin

## 概要

Kubernetes へエコシステムやアプリケーションを導入するためのリポジトリです。
ディレクトリごとに 1 つのプラグインとして導入に必要なマニフェストや利用手順を用意しており、検証/利用したいプラグインを選択して導入できます。

## プラグイン一覧

| プラグイン                | 概要                             | 依存関係                                           | 備考         |
| :------------------------ | :------------------------------- | :------------------------------------------------- | :----------- |
| albc                      | AWS LoadBalancer Controller      |                                                    | AWS のみ必要 |
| app-bookinfo              | Istio 動作確認アプリケーション   | service-mesh                                       |              |
| app-examples              | サンプルアプリケーション         |                                                    |              |
| app-ms                    | マイクロサービスアプリケーション | distributed-application<br>secrets<br>service-mesh |              |
| auth                      | 認証(keycloak)の導入             | service-mesh                                       |              |
| cert-manager              | SSL 証明書管理の導入             |                                                    |              |
| cluster-autoscaler        | クラスタの Node 数調整           |                                                    | AWS のみ必要 |
| container-registry        | Harbor の導入                    |                                                    |              |
| container-workflow-engine | Argo Workflows の導入            |                                                    |              |
| distributed-application   | Dapr の導入                      |                                                    |              |
| kong-apigateway           | KongApiGateway の導入            | distributed-application<br>pod-autoscaler          |              |
| metrics-server            | メトリクスサーバーの導入         |                                                    | AWS のみ必要 |
| observation               | Grafana,Prometheus による監視    | metrics-server（AWS のみ） <br>cert-manager        |              |
| pod-autoscaler            | KEDA の導入                      |                                                    |              |
| secrets                   | external-secrets-operator の導入 |                                                    |              |
| service-mesh              | Istio の導入                     | albc（AWS のみ）                                   |              |

※依存関係のあるものは、依存先のプラグインを先に導入しておく必要があります。

## 導入

nautible-plugin では ArgoCD の Application リソース（カスタムリソース）を利用してエコシステム/アプリケーションを Kubernetes にデプロイします。

![nautible-plugin全体像](./outline.svg)

エコシステムのマニフェストファイルは nautible-plugin のリポジトリで管理しているものを参照するケースと、各エコシステムのリポジトリにあるものを直接参照するケースがあります。（パラメータ調整等を行わず導入しているものは各エコシステムのリポジトリを直接参照しています）

アプリケーションのマニフェストファイルは nautible の Github でアプリケーションごとに用意しているマニフェストリポジトリ（<アプリケーションリポジトリ名>-manifest）を参照しています。

プラグインごとの詳細な導入手順については各プラグインディレクトリ配下の README.md を参照してください。
