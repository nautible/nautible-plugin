# distributed-application

## 1. 概要

分散アプリケーションランタイム[Dapr](https://dapr.io/)を導入し、分散アプリケーションの構築をサポートする機能を提供する。

### 開発Tips

Daprを利用する際の開発Tipsは[こちら](https://github.com/nautible/docs/tree/main/reference/dapr)を参照

## 2. 導入

```
$ kubectl apply -f distributed-application/application.yaml
```

## 3. 確認

```
$ kubectl get deploy -n dapr-system
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
dapr-dashboard          1/1     1            1           18d
dapr-operator           1/1     1            1           18d
dapr-sentry             1/1     1            1           18d
dapr-sidecar-injector   1/1     1            1           18d
```

## 4. 削除

```
$ kubectl delete -f distributed-application/application.yaml
```
