# kong-apigateway

## 1. 概要

Kong社が提供するOSSのAPIGatewayにカスタムプラグインを追加したAPIGateway

### カスタムプラグイン（serverless）

バックエンドのアプリケーションコンテナをサーバレス化（未使用時はコンテナ数を０にする）するための機能。

APIGatewayでリクエストを受信した際に、バックエンド

## 2. 導入

### 事前準備

[KEDA](https://github.com/nautible/nautible-plugin/tree/main/pod-autoscaler)を事前に導入しておく。

### base/application.yaml

リポジトリおよびタグを標準のkongから変更する。また、プラグインの起動に必要な環境変数を設定する。

```yaml
    helm:
      parameters:
        - name: 'image.repository'
          value: 'public.ecr.aws/nautible/nautible-kong-serverless'
        - name: 'image.tag'
          value: 'v0.1.4'
        - name: 'env.plugins'
          value: 'bundled, serverless'
        - name: 'env.pluginserver_names'
          value: 'serverless'
        - name: 'env.pluginserver_serverless_query_cmd'
          value: '/usr/local/bin/serverless -dump'
```

- AWS環境にデプロイ

```bash
kubectl apply -f kong-apigateway/overlays/aws/application.yaml
```

- Azure環境にデプロイ

```bash
kubectl apply -f kong-apigateway/overlays/azure/application.yaml
```

### （参考）導入元の定義ファイル

Kong-APIGatewayはDB不要版の書きマニフェストをベースに導入している。

```bash
https://raw.githubusercontent.com/Kong/kubernetes-ingress-controller/master/deploy/single/all-in-one-dbless.yaml
```

### カスタムプラグインの導入

TODO: nautible-kong-serverless作成後に追記する

## 3. 確認

```
$ kubectl get deploy -n kong
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
ingress-kong   1/1     1            1           8d
```

## 4. 削除

```bash
kubectl delete -f kong-apigateway/application.yaml
```

## 5. 参考

カスタムプラグインの開発リポジトリは[こちら](https://github.com/nautible/nautible-kong-serverless)
