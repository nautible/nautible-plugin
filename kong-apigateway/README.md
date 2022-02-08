# kong-apigateway

## 1. 概要

Kong社が提供するOSSのApiGateway

nginxベースのApiGatewayをKubernetes内にデプロイして稼働するため、オンプレ/クラウドどちらでも利用することが可能


## 2. 導入

```
$ kubectl apply -f kong-apigateway/application.yaml
```
### （参考）導入元の定義ファイル

DB不要版の導入ファイルは下記のものを使用している

```
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

```
$ kubectl delete -f kong-apigateway/application.yaml
```
