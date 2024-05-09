# ローカル環境向け 削除手順

## 1. Instrumentation リソースの削除

```bash
kubectl delete -f observation/examples/instrumentation/nautible-app-examples.yaml
```

## 2. その他リソースの削除

ArgoCD のコンソール画面より 各エコシステム の削除を行う。

コマンドラインによる削除を行う場合は、Argo CD CLI を使用して Application リソースを削除する。

```bash
argocd app delete argocd/observation
```
