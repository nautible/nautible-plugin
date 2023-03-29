# container-workflow-engine

## 1. 概要

コンテナネイティブワークフローエンジン[ArgoWorkflows](https://argoproj.github.io/argo-workflows/)を導入し、バッチアプリケーション等を構築する機能を提供する。

## 2. 導入

```
$ kubectl apply -f container-workflow-engine/application.yaml
```

## 3. 確認

```
$ kubectl get deploy -n argowf
NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
argo-workflows-server                1/1     1            1           3m59s
argo-workflows-workflow-controller   1/1     1            1           3m59s
```


※ WebUIアクセス
```
kubectl -n argowf port-forward deployment/argo-workflows-server 2746:2746
http://localhost:2746/
```

## 4. 削除

```
$ kubectl delete -f container-workflow-engine/application.yaml
```
