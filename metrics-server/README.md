
# Metrics Server

## 1. 概要

クラスタ内のリソース使用状況を取得するメトリクスサーバー

AWS（EKS）にはデフォルトで導入されないため、メトリクスを取得する際は導入する必要がある。

## 2. 導入

```
$ kubectl apply -f metrics-server/application.yaml
```

## 3. 確認

```
$ kubectl get deployment metrics-server -n kube-system

NAME             READY   UP-TO-DATE   AVAILABLE   AGE
metrics-server   1/1     1            1           6m
```

## 4. 削除

```
$ kubectl delete -f metrics-server/application.yaml
```
