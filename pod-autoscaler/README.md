# pod-autoscaler

## 1. 概要

イベントドリブン型で動作する（キューなどのバックエンドで非同期動作する）Podのオートスケーラ。

[KEDA](https://keda.sh/)を導入し、0～Nのスケーリングに対応する。対応するイベントソースは[公式ドキュメント](https://keda.sh/docs/2.6/scalers/)を参照。

## 2. 導入

```bash
kubectl apply -f pod-autoscaler/application.yaml
```

## 3. 確認

```bash
kubectl get deploy -n keda
```

<pre>
NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
keda-admission-webhooks           1/1     1            1           51s
keda-operator                     1/1     1            1           51s
keda-operator-metrics-apiserver   1/1     1            1           51s
</pre>

## 4. 動作検証

イベントソースにRedisを使用した動作サンプルでKEDAの動作確認を行う。

### 構成

<pre>
examples
├ setup
│ ├ redis.yaml
│ │ └ RedisのDeploymentおよびSerivceリソース
│ ├ scaledobject.yaml
│ │ └ オートスケール設定を記載したリソース（KEDAのScaledObjectリソース）
│ └ receiver.yaml
│   └ KEDAからイベントを受信して起動するサンプルアプリケーション
└ producer.yaml
  └ Redisにイベントを送信するテストジョブ
</pre>

### KEDA ScaleObjectの導入（イベントソースとしてRedisを設定）

```bash
kubectl apply -f examples/setup/.
```

### デプロイ結果の確認

```bash
kubectl get scaledobject
```

<pre>
NAME                 SCALETARGETKIND      SCALETARGETNAME   MIN   MAX   TRIGGERS   AUTHENTICATION   READY   ACTIVE   FALLBACK   PAUSED    AGE
redis-scaledobject   apps/v1.Deployment   receiver          0     4     redis                       True    False    Unknown    Unknown   12s
</pre>

```bash
kubectl get hpa
```

<pre>
NAME                          REFERENCE             TARGETS              MINPODS   MAXPODS   REPLICAS   AGE
keda-hpa-redis-scaledobject   Deployment/receiver   <unknown>/10 (avg)   1         4         0          2s
</pre>

```bash
kubectl get deploy
```

<pre>
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
receiver   0/0     0            0           14s
redis      1/1     1            1           14s
</pre>

### Redisにテストデータ登録

```bash
kubectl apply -f examples/producer.yaml
```

### receiverの起動確認

```bash
kubectl get hpa
```

<pre>
NAME                          REFERENCE             TARGETS      MINPODS   MAXPODS   REPLICAS   AGE
keda-hpa-redis-scaledobject   Deployment/receiver   1/10 (avg)   1         4         1          2m16s
</pre>

```bash
kubectl get po
```

<pre>
NAME                       READY   STATUS      RESTARTS   AGE
keda-test-job-lq2ld        0/1     Completed   0          9s
receiver-f7f5c78c5-cf894   1/1     Running     0          2s
redis-78dbb788cf-ld4ft     1/1     Running     0          110s
</pre>

## 5. 削除

### サンプルアプリケーションの削除

```bash
kubectl delete -f examples/setup/.
```

### KEDAの削除

ArgoCDのコンソール画面よりkedaの削除を行う。

コマンドラインによる削除を行う場合は、Argo CD CLIを使用してApplicationリソースを削除する。

```
argocd app delete argocd/keda
```
