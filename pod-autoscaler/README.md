# pod-autoscaler

## 1. 概要

イベントドリブン型で動作する（キューなどのバックエンドで非同期動作する）Podのオートスケーラ

[KEDA](https://keda.sh/)を導入し、0～Nのスケーリングに対応する。対応するイベントソースは[公式ドキュメント](https://keda.sh/docs/2.6/scalers/)を参照。

## 2. 導入

```
$ kubectl apply -f pod-autoscaler/application.yaml
```

## 3. 確認

```
$ kubectl get deploy -n keda
NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
keda-operator                     1/1     1            1           7d23h
keda-operator-metrics-apiserver   1/1     1            1           7d23h
```

## 4. 削除

```
$ kubectl delete -f pod-autoscaler/application.yaml
```
