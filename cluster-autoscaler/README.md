
# Cluster Autoscaler

## 1. 概要

クラスタのノードをオートスケールするための機能を導入する。

AWS（EKS）にはデフォルトで導入されないため、Nodeのオートスケールを行う際は導入する必要がある。

## 2. 導入

helm.parameters.valueの値を対象のクラスタ名、ロールarnに変更する。  
※ロールはterraformで作成されます。

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

ArgoCDのコンソール画面よりclsuter-autoscalerの削除を行う。

コマンドラインによる削除を行う場合は、Argo CD CLIを使用してApplicationリソースを削除する。

```BASH
argocd app delete argocd/clsuter-autoscaler
```
