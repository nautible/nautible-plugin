# nautible-plugin

Kubernetesへエコシステムやアプリケーションを導入するためのリポジトリ

すべてArgoCDのアプリケーションとして管理するため、事前にArgoCDの導入までは実施する。

## プラグイン一覧

|プラグイン|概要|依存関係|備考|
|:--|:--|:--|:--|
|app-bookinfo|Istio動作確認アプリケーション|service-mesh||
|app-ms|マイクロサービスアプリケーション|distributed-application,secrets||
|cluster-autoscaler|クラスタのNode数調整||AWSのみ必要|
|container-registry|Harbor（プライベートレジストリ）の導入|||
|distributed-application|Daprの導入|||
|kong-apigateway|KongApiGatewayの導入|||
|metrics-server|Istio動作確認アプリケーション|service-mesh|AWSのみ必要|
|observation|Grafana,Prometheusによる監視|metrics-server||
|pod-autoscaler|KEDAの導入|||
|secrets|kubernetes-external-secretsの導入|||
|service-mesh|Istioの導入|||

※依存関係のあるものは、依存先のプラグインを先に導入しておく必要がある