# AWS 環境向け 導入手順

## 1. Terraform で必要なリソースを作成

AWS では Grafana Mimir / Loki / Tempo のデータを S3 上に永続化するように設定を行う。必要な S3 バケット作成及び権限設定を Terraform で実施する。

// TODO 参照先

## 2. ServiceAccount の登録

Grafana Loki / Grafana Tempo / Grafana Mimir に S3 へのアクセス権限を付与するため、ServiceAccount の登録を行う。

```bash
kubectl apply -f observation/overlays/aws/namespace.yaml
ACCOUNT_ID=<AWSアカウントID> && eval "echo \"$(cat observation/overlays/aws/serviceaccount.yaml)\"" | kubectl apply -f -
```

なお、本手順では GitHub 上のコミットファイルに AWS ACCOUNT_ID を含めない前提の手順としているため、serviceaccount.yaml の登録を手動で行っている。コミットファイルに AWS ACCOUNT_ID を含めてもいい場合は、observation/setup/overlays/aws/{loki-patch.yaml|tempo-patch.yaml|mimir-patch.yaml} の HELM パラメータを以下のように変更することで、本手順はスキップできる。

```
serviceAccount:
  create: true
  name: {loki-sa|tempo-sa|mimir-sa}
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<ACCOUNT_ID>:role/<ロール名>
```

## 3. エコシステムの導入

### Grafana Alloy の設定変更

Grafana Alloy の設定は下記 ConfigMap で実施している。各導入環境に合わせて設定内容を変更する。

```bash
observation/overlays/aws/alloy-config.yaml
```

### パッチの設定変更

各エコシステムのパッチについても必要に応じて変更を行う。デフォルトでは以下の設定としている。

- observation/overlays/aws/alloy-patch.yaml
  - 受信ポート設定（gRPC:4317 HTTP:4318）
  - ログボリュームのマウント
  - ConfigMap の読み込み
- observation/overlays/aws/grafana-patch.yaml
  - 設定の永続化設定（2G のボリュームをマウント）
- observation/overlays/aws/loki-patch.yaml
  - 認証不要
  - S3 への永続設定
  - レプリカ設定（すべて 1）
- observation/overlays/aws/mimir-patch.yaml
  - S3 への永続設定
  - レプリカ設定（すべて 1）
- observation/overlays/aws/tempo-patch.yaml
  - S3 への永続設定
  - レプリカ設定（ingester 2、その他 1 ※ ingester は最低 2 となっている）

### エコシステムのデプロイ

```bash
kubectl apply -f observation/overlays/aws/application.yaml
```

### 4. Auto Instrumentation の有効化

計測を行う namespace に対して Instrumentation リソースをデプロイする。

```bash
kubectl apply -f observation/examples/otel/instrumentation.yaml
```

### アプリケーションへのアノテーション付与

nautible-app-examples-java の例

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nautible-app-examples-java
  namespace: nautible-app-examples
...
spec:
  replicas: 2
  selector:
    matchLabels:
      app.kubernetes.io/instance: nautible-app-examples-java
      app.kubernetes.io/component: app
  template:
    metadata:
      annotations:
        instrumentation.opentelemetry.io/inject-java: 'true'
...
```

### アプリケーションのデプロイ

上記修正したマニフェストをデプロイする。

## 5. 削除

### Instrumentation リソースの削除

```bash
kubectl delete -f observation/examples/otel/instrumentation.yaml
```

### その他リソースの削除

ArgoCD のコンソール画面より observation の削除を行う。

コマンドラインによる削除を行う場合は、Argo CD CLI を使用して Application リソースを削除する。

```
argocd app delete argocd/observation
```

※ Grafana Loki のポッドは削除に時間がかかるケースがある。

### ServiceAccount の削除

```bash
kubectl delete -f observation/overlays/aws/serviceaccount.yaml
```

### Namespace の削除

```bash
kubectl delete -f observation/overlays/aws/namespace.yaml
```

### AWS リソースの削除

永続データを保存している S3 およびアクセス用のロールを削除する。  
削除の際は必要に応じて S3 の内容をバックアップする。

#### nautible-infra

```bash
cd aws/plugin/env/dev
terraform destroy
```
