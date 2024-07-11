# Karpenter

## 1. 概要

データプレーン（ワーカーノード）のオートスケール機能を導入する。

XXX

## 2. 導入

### リソース準備

### ロール

### コントローラー

```
helm registry logout public.ecr.aws
export KARPENTER_NAMESPACE=karpenter
export KARPENTER_VERSION=0.37.0
helm upgrade --install karpenter oci://public.ecr.aws/karpenter/karpenter --version "${KARPENTER_VERSION}" --namespace "${KARPENTER_NAMESPACE}" --create-namespace -f ./karpenter/values.yaml --wait
```

### CRD

helm.parameters.value の値を対象のクラスタ名、ロール arn に変更する。  
※ロールは terraform で作成されます。terraform の output を参照してください。

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

cluster-autoscaler をデプロイする。

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

ArgoCD のコンソール画面より cluster-autoscaler の削除を行う。

コマンドラインによる削除を行う場合は、Argo CD CLI を使用して Application リソースを削除する。

```BASH
argocd app delete argocd/cluster-autoscaler
```
