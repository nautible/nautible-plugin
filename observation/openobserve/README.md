# OpenObserve

## 1. 概要

クラスタ内のリソース使用状況を取得するメトリクスサーバー。ノード単位、Pod単位の基本的なメトリクス（CPU、メモリなど）を取得することができる。

AWS（EKS）にはデフォルトで導入されないため、メトリクスを取得する際は導入する必要がある。

## 2. 導入

AWS

```bash
kubectl apply -f observation/openobserve/overlays/aws/application.yaml
```

### prometheus operator crd

```bash
kubectl create -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/prometheus-operator-crd/monitoring.coreos.com_servicemonitors.yaml
kubectl create -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/prometheus-operator-crd/monitoring.coreos.com_podmonitors.yaml
kubectl create -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/refs/heads/main/example/prometheus-operator-crd/monitoring.coreos.com_scrapeconfigs.yaml
kubectl create -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/refs/heads/main/example/prometheus-operator-crd/monitoring.coreos.com_probes.yaml
```

### OpenTelemetry Operator

```bash
kubectl apply -f https://github.com/open-telemetry/opentelemetry-operator/releases/latest/download/opentelemetry-operator.yaml
```

### Collector

```bash
kubectl create ns openobserve-collector
helm --namespace openobserve-collector \
  install o2c openobserve/openobserve-collector \
  --set k8sCluster=nautible-dev-cluster-v1_32  \
  --set exporters."otlphttp/openobserve".endpoint=http://o2-openobserve-router.openobserve.svc.cluster.local:5080/api/default  \
  --set exporters."otlphttp/openobserve".headers.Authorization="Basic XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"  \
  --set exporters."otlphttp/openobserve_k8s_events".endpoint=http://o2-openobserve-router.openobserve.svc.cluster.local:5080/api/default  \
  --set exporters."otlphttp/openobserve_k8s_events".headers.Authorization="Basic XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX" \
  --create-namespace

```

## 3. 確認

### メトリクスサーバーの動作確認

```bash
kubectl get deployment metrics-server -n kube-system
```

<pre>
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
metrics-server   1/1     1            1           6m
</pre>

### メトリクスの確認

```bash
kubectl top node
```

<pre>
NAME                                               CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
ip-192-168-1-71.ap-northeast-1.compute.internal    26m          1%     626Mi           19%       
ip-192-168-2-120.ap-northeast-1.compute.internal   25m          1%     576Mi           18%       
ip-192-168-2-90.ap-northeast-1.compute.internal    25m          1%     701Mi           22%       
</pre>

```bash
kubectl top pod -n kube-system
```

<pre>
NAME                                  CPU(cores)   MEMORY(bytes)   
aws-node-dsnl5                        3m           56Mi            
aws-node-m82bm                        2m           56Mi            
aws-node-nppl7                        2m           57Mi            
coredns-6c4ffd9cc7-98wch              1m           12Mi            
coredns-6c4ffd9cc7-xsj6t              1m           12Mi            
ebs-csi-controller-5cd87b44cd-n5wwq   2m           52Mi            
ebs-csi-controller-5cd87b44cd-scw8r   3m           51Mi            
ebs-csi-node-hmm9h                    1m           19Mi            
ebs-csi-node-klpj5                    1m           19Mi            
ebs-csi-node-mf2js                    1m           19Mi            
kube-proxy-hpjnr                      1m           11Mi            
kube-proxy-l9stm                      1m           11Mi            
kube-proxy-xkp8j                      1m           11Mi            
metrics-server-967bc8b5c-h6pw7        2m           14Mi       
</pre>

## 4. 削除

ArgoCDのコンソール画面よりmetrics-serverの削除を行う。

コマンドラインによる削除を行う場合は、Argo CD CLIを使用してApplicationリソースを削除する。

```
argocd app delete argocd/metrics-server
```
