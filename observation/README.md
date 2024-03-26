## 事前準備

## AWS

### Terraform で必要なリソースを作成

### エコシステムの導入

### ServiceAccount の登録

Grafana Loki / Grafana Tempo に S3 へのアクセス権限を付与するため、ServiceAccount の登録を行う。

```bash
kubectl apply -f observation/overlays/aws/namespace.yaml
ACCOUNT_ID=<AWSアカウントID> && eval "echo \"$(cat observation/overlays/aws/serviceaccount.yaml)\"" | kubectl apply -f -
```

なお、本手順ではコミットファイルに AWS ACCOUNT_ID を含めない前提の手順としているため、手動で serviceaccount.yaml の登録を手動で行っている。コミットファイルに AWS ACCOUNT_ID を含めてもいい場合は、observation/setup/overlays/aws/{loki-patch.yaml|tempo-patch.yaml} の HELM パラメータに annotations で直接ロールの紐づけを行うことで本手順はスキップできる。

### 可観測エコシステムの導入

- Prometheus
- Alert Manager
- Grafana
- Grafana Loki
- Grafana Tempo
- Promtail
- OpentTelemetry Operator

```bash
kubectl apply -f observation/overlays/aws/application.yaml
```
