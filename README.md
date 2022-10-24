# nautible-plugin

## 概要

Kubernetesへエコシステムやアプリケーションを導入するためのリポジトリです。
ディレクトリごとに1つのプラグインとして導入に必要なマニフェストや利用手順を用意しており、検証/利用したいプラグインを選択して導入できます。

## プラグイン一覧

|プラグイン|概要|依存関係|備考|
|:--|:--|:--|:--|
|albc|AWS LoadBalancer Controller||AWSのみ必要|
|app-bookinfo|Istio動作確認アプリケーション|service-mesh||
|app-examples|サンプルアプリケーション|||
|app-ms|マイクロサービスアプリケーション|distributed-application<br>secrets<br>service-mesh||
|auth|認証(keycloak)の導入|service-mesh||
|cluster-autoscaler|クラスタのNode数調整||AWSのみ必要|
|container-registry|Harborの導入|||
|distributed-application|Daprの導入|||
|kong-apigateway|KongApiGatewayの導入|distributed-application<br>pod-autoscaler||
|metrics-server|メトリクスサーバーの導入||AWSのみ必要|
|observation|Grafana,Prometheusによる監視|metrics-server（AWSのみ）||
|pod-autoscaler|KEDAの導入|||
|secrets|external-secrets-operatorの導入|||
|service-mesh|Istioの導入|albc（AWSのみ）||

※依存関係のあるものは、依存先のプラグインを先に導入しておく必要があります。

## 導入

nautible-pluginではArgoCDのApplicationリソース（カスタムリソース）を利用してエコシステム/アプリケーションをKubernetesにデプロイします。

![nautible-plugin全体像](./outline.svg)

エコシステムのマニフェストファイルはnautible-pluginのリポジトリで管理しているものを参照するケースと、各エコシステムのリポジトリにあるものを直接参照するケースがあります。（パラメータ調整等を行わず導入しているものは各エコシステムのリポジトリを直接参照しています）

アプリケーションのマニフェストファイルはnautibleのGithubでアプリケーションごとに用意しているマニフェストリポジトリ（<アプリケーションリポジトリ名>-manifest）を参照しています。

プラグインごとの詳細な導入手順については各プラグインディレクトリ配下のREADME.mdを参照してください。
