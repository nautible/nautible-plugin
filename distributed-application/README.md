# distributed-application

## 1. 概要

分散アプリケーションランタイム[Dapr](https://dapr.io/)を導入し、分散アプリケーションの構築をサポートする機能を提供する。

### 開発 Tips

Dapr を利用する際の開発 Tips は[こちら](https://github.com/nautible/docs/tree/main/reference/dapr)を参照

## 2. 導入

```
$ kubectl apply -f distributed-application/application.yaml
```

## 3. 確認

```
$ kubectl get deploy -n dapr-system
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
dapr-operator           1/1     1            1           18d
dapr-sentry             1/1     1            1           18d
dapr-sidecar-injector   1/1     1            1           18d
```

## 4. 削除

ArgoCD のコンソール画面より Dapr の削除を行う。

コマンドラインによる削除を行う場合は、Argo CD CLI を使用して Application リソースを削除する。

```
argocd app delete argocd/dapr
```
