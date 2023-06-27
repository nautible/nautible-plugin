# セットアップ

以下のプロダクトをKubernetesにデプロイする

- モニタリング
  - Prometheus-Operator
    - Prometheus
    - Node-Exporter
    - kube-state-metrics
- アラート
  - Alertmanager
- ロギング
  - Grafana Loki
  - Promtail
- トレーシング
  - Grafana Tempo
- 可視化
  - Grafana

## 前提

ローカルでkubectlが実行できること  
kubernetesにArgoCDがデプロイされていること（ServerSideApplyを利用するためv2.5以上）

## 事前準備

### metrics-server

EKSの場合、デフォルトでメトリクスサーバーのPodがデプロイされていないため、デプロイしておく。

[導入手順はmetrics-serverのドキュメントを参照](../../metrics-server/README.md)

### cert-manager

KubernetesにCertManagerが導入されていない場合は事前に導入しておく

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.1/cert-manager.yaml
```

※ v1.12.1の部分は導入時の最新バージョンに置き換える

## リポジトリのclone

```bash
git clone https://github.com/nautible/nautible-plugin.git
cd nautible-plugin
```

## エコシステムの導入

### AWS

```bash
kubectl apply -f observation/manifests/overlays/aws/application.yaml
```

### Azure

```bash
kubectl apply -f observation/manifests/overlays/azure/application.yaml
```

## 動作確認

### Prometheus確認

```bash
kubectl port-forward svc/kube-prometheus-stack-prometheus -n monitoring 9090:9090
```

- ブラウザでアクセス

```bash
http://localhost:9090
```

デフォルトではアラート確認用にWatchdogが常にエラーとして検出される  
※環境によりそのほかのアラートが出る場合もある（メモリの使い過ぎ、Podが多すぎるなど）  

![prometheus1](./img/prometheus1.png)

### Alertmanager確認

```bash
kubectl port-forward svc/kube-prometheus-stack-alertmanager -n monitoring 9093:9093
```

- ブラウザでアクセス

```bash
http://localhost:9093
```

Prometheusで確認したWatchdogのアラートがこちらも表示される  

![alertmanager1](./img/alertmanager1.png)

![alertmanager2](./img/alertmanager2.png)

### Grafana確認

```bash
kubectl port-forward svc/kube-prometheus-stack-grafana -n monitoring 8080:80
```

- ブラウザでアクセス

```bash
http://localhost:8080
```

デフォルトのログインID/PWはadmin/prom-operator

![grafana1](./img/grafana1.png)

デフォルトでPrometheusが設定されている

![grafana2](./img/grafana2.png)

### GrafanaにLokiのデータソースを追加

Lokiのデータソースはloki-gatewayから接続する。Serviceの確認は以下の通り。

```bash
kubectl get svc -n monitoring

NAME                                             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
loki-gateway                                     ClusterIP   10.100.213.93    <none>        80/TCP                       5h48m
```

上記で確認したサービス名:PORTでPrometheusのデータソースにLokiを追加する  

![loki1](./img/loki1.png)

![loki2](./img/loki2.png)

## 各種サンプル設定

### Istio

```bash
kubectl apply -f observation/examples/istio/application.yaml
```

### nautible-app-ms（マイクロサービステンプレートアプリケーション）

```bash
kubectl apply -f observation/examples/nautible-app-ms/application.yaml
```
