# AWS 環境向け 導入手順

## 1. エコシステムの導入

### Grafana Alloy の設定変更

Grafana Alloy の設定は下記 ConfigMap で実施している。各導入環境に合わせて設定内容を変更する。

```bash
observation/base/alloy/alloy-config.yaml
```

### パッチの設定変更

各エコシステムのパッチについても必要に応じて変更を行う。デフォルトでは以下の設定としている。

- observation/overlays/aws/grafana-patch.yaml
  - デフォルト
- observation/overlays/aws/loki-patch.yaml
  - 認証不要
  - filesystem への永続設定
  - シングルバイナリモード
- observation/overlays/aws/mimir-patch.yaml
  - デフォルト
- observation/overlays/aws/tempo-patch.yaml
  - デフォルト

### エコシステムのデプロイ

```bash
kubectl apply -f observation/overlays/local/application.yaml
```

### 2. Auto Instrumentation の有効化

計測を行う namespace に対して Instrumentation リソースをデプロイする。  
サンプルは nautible-app-examples に対して有効化している。

```bash
kubectl apply -f observation/examples/instrumentation/nautible-app-examples.yaml
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

## 3. 削除

### Instrumentation リソースの削除

```bash
kubectl delete -f observation/examples/instrumentation/nautible-app-examples.yaml
```

### その他リソースの削除

ArgoCD のコンソール画面より observation の削除を行う。

コマンドラインによる削除を行う場合は、Argo CD CLI を使用して Application リソースを削除する。

```
argocd app delete argocd/observation
```
