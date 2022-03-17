
# app-bookinfo

## 1. 概要

Istio動作検証用アプリケーション

### ディレクトリ構成

|ディレクトリ名|内容|
|:--|:--|
|bookinfo|bookinfoアプリケーションの作成|
|bookinfo-gateway|Ingressの作成|
|namespace|namespace作成|
|networking|ルーティングの作成|

## 2. 導入

```
$ kubectl apply -f app-bookinfo/application.yaml
```

## 3. 確認

アプリケーション

```
$ kubectl get deploy -n bookinfo
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
details-v1       1/1     1            1           40s
productpage-v1   1/1     1            1           40s
ratings-v1       1/1     1            1           40s
reviews-v1       1/1     1            1           40s
reviews-v2       1/1     1            1           40s
reviews-v3       1/1     1            1           40s
```

Gateway（Ingress）

```
$ k get Gateway -n bookinfo
NAME               AGE
bookinfo-gateway   4m3s
```

VirtualService

```
$ kubectl get VirtualService -n bookinfo
NAME       GATEWAYS               HOSTS         AGE
bookinfo   ["bookinfo-gateway"]   ["*"]         2m43s
ratings                           ["ratings"]   2m42s
reviews                           ["reviews"]   2m42s
```

DestinationRule

```
$ kubectl get DestinationRule -n bookinfo
NAME          HOST          AGE
details       details       3m18s
productpage   productpage   3m18s
ratings       ratings       3m18s
reviews       reviews       3m18s
```

## 4. 削除

```
$ kubectl delete -f app-bookinfo/application.yaml
```
