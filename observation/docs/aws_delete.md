# AWS 環境向け 削除手順

## 1. Instrumentation リソースの削除

```bash
kubectl delete -f observation/examples/instrumentation/nautible-app-examples.yaml
```

## 2. その他リソースの削除

ArgoCD のコンソール画面より observation の削除を行う。

コマンドラインによる削除を行う場合は、Argo CD CLI を使用して Application リソースを削除する。

```bash
argocd app delete argocd/observation
```

※ Grafana Loki のポッドは削除に時間がかかるケースがある。

## 3. ServiceAccount の削除

```bash
kubectl delete -f observation/overlays/aws/serviceaccount.yaml
```

## 4. Namespace の削除

```bash
kubectl delete -f observation/overlays/aws/namespace.yaml
```

## 5. nautible-infra で作成した S3 バケット、IAM ロールの削除

永続データを保存している S3 およびアクセス用のロールを削除する。  
削除の際は必要に応じて S3 の内容をバックアップする。

nautible-infra

```bash
cd aws/plugin/env/dev
terraform destroy
```

※ 他のプラグイン用の環境を残す場合は destroy ではなく variables.tf を編集して terraform apply すること。
