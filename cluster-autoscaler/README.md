
# Cluster Autoscaler

## 0. 注意事項

本導入手順はnautible-infraの tag:2024.2.0 バージョン以降で構築されたEKSに対応しています。  
（2024.2.0から認証方式をIRSAからPod Identityに変更しています）

2024.2.0より前のバージョンのnautible-infraでEKSを構築している場合、nautible-pluginのバージョンも2024.2.0より前のバージョンを利用してください。

## 1. 概要

データプレーン（ワーカーノード）のオートスケール機能を導入する。

nautible-infraのTerraformコードからEKSをデプロイした場合、オートスケールの基盤としてEC2AutoScalingGroupを導入している。
Cluster AutoscalerはPodのスケジュール失敗や別ノードへの再スケジュールをトリガーにEC2AutoScalingGroupを使用してオートスケールを実現する。  
（Karpenterを利用してオートスケールを実現する場合、本機能は必要ありません。）

## 2. 導入

helm.parameters.valueのautoDiscovery.clusterNameにクラスタ名を設定する。

```YAML
    helm:
      parameters:
        - name: 'autoDiscovery.clusterName'
          value: 'nautible-dev-cluster'      # 対象のクラスタ名に変更する
        - name: 'awsRegion'
          value: 'ap-northeast-1'
```

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
