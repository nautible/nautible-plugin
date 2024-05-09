# cert-manager

## 1. 概要

Kubernetes クラスタ内で証明書を自動的に管理するためのツール

## 2. 導入

```bash
kubectl apply -f cert-manager/application.yaml
```

## 3. 削除

ArgoCD のコンソール画面より cert-manager の削除を行う。

コマンドラインによる削除を行う場合は、Argo CD CLI を使用して Application リソースを削除する。

```
argocd app delete argocd/cert-manager
```
