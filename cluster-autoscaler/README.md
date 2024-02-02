
# Cluster Autoscaler

## 1. 概要

データプレーン（ワーカーノード）のオートスケール機能を導入する。

nautible-infraのTerraformコードからEKSをデプロイした場合、オートスケールの基盤としてEC2AutoScalingGroupを導入している。
Cluster AutoscalerはPodのスケジュールに失敗した場合などにEC2AutoScalingGroupを使用してオートスケールを実現する。

## 2. 導入

helm.parameters.valueの値を対象のクラスタ名、ロールarnに変更する。  
※ロールはterraformで作成されます。terraformのoutputを参照してください。

<pre>
    helm:
      parameters:
        - name: 'autoDiscovery.clusterName'
          value: 'nautible-dev-cluster'      # 対象のクラスタ名に変更する
        - name: 'awsRegion'
          value: 'ap-northeast-1'
        - name: 'rbac.serviceAccount.annotations.eks\.amazonaws\.com/role-arn'
          value: 'arn:aws:iam::XXXXXXXXXXX:role/XXXXXXXXXX-AmazonEKSClusterAutoscalerRole' # 対象のロールarnに変更する。
</pre>

cluster-autoscalerをデプロイする。

```BASH
kubectl apply -f cluster-autoscaler/application.yaml
```

## 3. 確認

```BASH
kubectl get deploy -n autoscaler
```

<pre>
NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
cluster-autoscaler-aws-cluster-autoscaler   1/1     1            1           18d
</pre>

## 4. 削除

ArgoCDのコンソール画面よりcluster-autoscalerの削除を行う。

コマンドラインによる削除を行う場合は、Argo CD CLIを使用してApplicationリソースを削除する。

```BASH
argocd app delete argocd/cluster-autoscaler
```
