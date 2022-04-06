# nautible-plugin

## 概要

Kubernetesへエコシステムやアプリケーションを導入するためのリポジトリです。
ディレクトリごとに1つのプラグインとしてまとめており、導入に必要なマニフェストや利用手順を用意していますので、検証/利用したいプラグインを選択して導入できます。

## プラグイン一覧

|プラグイン|概要|依存関係|備考|
|:--|:--|:--|:--|
|app-bookinfo|Istio動作確認アプリケーション|service-mesh||
|app-ms|マイクロサービスアプリケーション|distributed-application<br>secrets<br>service-mesh||
|cluster-autoscaler|クラスタのNode数調整||AWSのみ必要|
|container-registry|Harborの導入|||
|distributed-application|Daprの導入|||
|kong-apigateway|KongApiGatewayの導入|||
|metrics-server|メトリクスサーバーの導入||AWSのみ必要|
|observation|Grafana,Prometheusによる監視|metrics-server（AWSのみ）||
|pod-autoscaler|KEDAの導入|||
|secrets|kubernetes-external-secretsの導入|||
|service-mesh|Istioの導入|||

※依存関係のあるものは、依存先のプラグインを先に導入しておく必要があります。

## 導入手順

すべてArgoCDのアプリケーションとして管理するため、事前にArgoCDの導入までは実施します。

導入はkubectlコマンドで各プラグインのapplication.yamlをデプロイします。
プラグインごとの詳細な導入手順については各プラグインディレクトリ配下のREADME.mdを参照してください。
