
# Cluster Autoscaler

## 1. 概要

クラスタのノードをオートスケールするための機能を導入する。

AWS（EKS）にはデフォルトで導入されないため、Nodeのオートスケールを行う際は導入する必要がある。

## 2. 導入

helm.parameters.valueの値を対象のクラスタ名に変更する。

```
    helm:
      parameters:
        - name: 'autoDiscovery.clusterName'
          value: 'nautible-dev-cluster'      # 対象のクラスタ名に変更する
        - name: 'awsRegion'
          value: 'ap-northeast-1'
```

cluster-autoscalerをデプロイする。

```
$ kubectl apply -f cluster-autoscaler/application.yaml
```

## 3. 確認

```
$ kubectl get deploy -n autoscaler
NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
cluster-autoscaler-aws-cluster-autoscaler   1/1     1            1           18d
```

## 4. 削除

```
$ kubectl delete -f cluster-autoscaler/application.yaml
```
