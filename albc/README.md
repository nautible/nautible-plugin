
# AWS LoadBalancer Controller

## 1. 概要

AWS LoadBalancer Controllerを導入する。  

以下の理由からAWS LoadBalancer Controllerを導入する  

- AWSのロードバランサーはCloudfrontからのリクエストのみ受け付けるように制御する(AWS Security Groupで制御)。  
- Classic LoadBalancerは2022年8月で廃止

## 2. 導入

helm.parameters.valueの値をLoadBalancer Controllerのロールarnに変更する。  
※ロールはterraformで作成されます。terraformのoutpoutを参照してください。

application.yaml
```YAML
    helm:
      parameters:
        - name: 'serviceAccount.annotations.eks\.amazonaws\.com/role-arn'
          value: 'arn:aws:iam::XXXXXXXXXXXX:role/XXXXXXXXXXXX-AmazonEKSLoadBalancerControllerRole' # 対象のロールarnに変更する。
```

AWS LoadBalancer Controllerをデプロイする。

```BASH
$ kubectl apply -f albc/application.yaml
```

Ingressの設定でLoadBalancerに設定するセキュリティグループに変更する。  
※ロールはterraformで作成されます。terraformのoutpoutを参照してください。

albc/ingress/manifest/application.yaml
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

```BASH
$ kubectl delete -f albc/ingress/application.yaml
$ kubectl delete -f albc/application.yaml
```
