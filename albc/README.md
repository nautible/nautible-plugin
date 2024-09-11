
# AWS LoadBalancer Controller

## 0. 注意事項

本導入手順はnautible-infraの tag:2024.2.0 バージョン以降で構築されたEKSに対応しています。  
（2024.2.0から認証方式をIRSAからPod Identityに変更しています）

2024.2.0より前のバージョンのnautible-infraでEKSを構築している場合、nautible-pluginのバージョンも2024.2.0より前のバージョンを利用してください。

## 1. 概要

AWS LoadBalancer Controllerを導入する。  

以下の理由からAWS LoadBalancer Controllerを導入する。  

- AWSのロードバランサーはCloudfrontからのリクエストのみ受け付けるように制御する(AWS Security Groupで制御)。  
- Classic LoadBalancerは2022年8月で廃止

## 2. 導入

### コントローラーの導入

helm.parameters.valueのclusterNameにALBを導入するクラスタ名を設定する。

application.yaml
```YAML
    helm:
      parameters:
        - name: 'clusterName'
          value: 'nautible-dev-cluster' # 対象のクラスタ名に変更する。
```

AWS LoadBalancer Controllerをデプロイする。

```BASH
$ kubectl apply -f albc/application.yaml
```

### Istio用ロードバランサの導入

Ingressの設定でLoadBalancerに設定するセキュリティグループに変更する。  

albc/ingress/manifest/ingress.yaml
```YAML
metadata:
  annotations:
    alb.ingress.kubernetes.io/security-groups: 'nautible-dev-albc-sg' # 対象のセキュリティグループに変更する。idまたは名称を指定する。
```

Ingressをデプロイする。

```BASH
$ kubectl apply -f albc/ingress/application.yaml
```


## 3. 確認

- AWS LoadBalancer Controllerの確認
```BASH
$ kubectl get deploy -n kube-system
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
aws-load-balancer-controller   2/2     2            2           23h
```

- Ingress(AWSのロードバランサー)の確認
```BASH
$ kubectl get Ingress -n istio-system
NAME                                   CLASS    HOSTS   ADDRESS                                                                        PORTS   AGE
aws-load-balancer-controller-ingress   <none>   *       k8s-nautiblealbingres-e139a26662-1579380625.ap-northeast-1.elb.amazonaws.com   80      49s
```

## 4. 削除

ArgoCDのコンソール画面よりaws-load-balancer-controller-ingressおよびaws-load-balancer-controllerの削除を行う。

コマンドラインによる削除を行う場合は、Argo CD CLIを使用してApplicationリソースを削除する。

```BASH
$ argocd app delete argocd/aws-load-balancer-controller-ingress
$ argocd app delete argocd/aws-load-balancer-controller
```
