
# app-ms

## 1. 概要

マイクロサービスサンプルアプリケーション

### ディレクトリ構成

base

|ディレクトリ名|内容|
|:--|:--|
|common|共通設定（namespace,role）|
|customer|ユーザー管理|
|order|受注管理|
|payment|支払い管理|
|product|商品管理|
|stock|在庫管理|

overlays

|ディレクトリ名|内容|
|:--|:--|
|aws|AWSへの導入アプリケーション|
|azure|AZureへの導入アプリケーション|

## 2. 導入

aws

```
$ kubectl apply -f app-ms/overlays/aws/application.yaml
```

azure

```
$ kubectl apply -f app-ms/overlays/azure/application.yaml
```

## 3. 確認

```
$ kubectl get deploy -n nautible-app
NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
nautible-app-customer              2/2     2            2           18d
nautible-app-order                 2/2     2            2           18d
nautible-app-payment-bff           1/1     1            1           18d
nautible-app-payment-cash          1/1     1            1           18d
nautible-app-payment-convenience   1/1     1            1           18d
nautible-app-payment-credit        1/1     1            1           18d
nautible-app-product               1/1     1            1           18d
nautible-app-stock                 2/2     2            2           18d
```

## 4. 削除

aws

```
$ kubectl delete -f app-ms/overlays/aws/application.yaml
```

azure

```
$ kubectl delete -f app-ms/overlays/azure/application.yaml
```
