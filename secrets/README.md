# Secrets

```
Deplicated : kubernetes-external-secretsはDeplicatedになっているため、External Secrets Operatorを利用するように変更する。
```

Kubernetesのsecret機構は暗号化されないため（base64のみ）、シークレットを暗号化された環境で管理するための仕組みを提供する。

## 1. 概要

### シークレットの管理機構

シークレットの管理にはkubernetes-external-secretsを利用する。

構成図等は[公式ドキュメント](https://github.com/external-secrets/kubernetes-external-secrets)を参照。

### シークレットの作成と管理

シークレットは各クラウドサービスのシークレット管理機能上で作成および管理する。

### シークレットの取得

kubernetes-external-secretsを導入し、シークレットをクラウドサービスのシークレット管理機能から取得する。

## 2. 導入

### kubernetes-external-secretsの導入

AWS

```
$ kubectl apply external-secrets/aws/application.yaml
```

Azure

```
$ kubectl apply external-secrets/azure/application.yaml
```

### ExternalSecretリソースの導入

app-msの稼働に必要なシークレットを導入する

AWS

```
$ kubectl apply -f secrets/secret-parameter/aws/application.yaml
```

Azure

```
$ kubectl apply -f secrets/secret-parameter/azure/application.yaml
```

## 3. 確認

### kubernetes-external-secretsの導入確認

```
$ kubectl get deploy -n kubernetes-external-secrets
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-external-secrets   1/1     1            1           18d
```

### ExternalSecretsおよびSecretの導入確認

default
```
$ kubectl get ExternalSecrets
NAME         LAST SYNC   STATUS    AGE
secret-sqs   5s          SUCCESS   17d

$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-99nmc   kubernetes.io/service-account-token   3      271d
secret-sqs            Opaque                                2      17d
```

nautible-app-ms
```
$ kubectl get ExternalSecrets -n nautible-app-ms
NAME                                LAST SYNC   STATUS    AGE
secret-nautible-app-ms-product-db   5s          SUCCESS   17d
$ kubectl get secrets -n nautible-app-ms
NAME                                TYPE                                  DATA   AGE
default-token-m8rtv                 kubernetes.io/service-account-token   3      17d
secret-nautible-app-ms-product-db   Opaque                                2      17d
```

## 4. 削除

### ExternalSecretリソースの削除

前提：事前にシークレットを利用しているアプリケーションの削除が完了していること

AWS

```
$ kubectl delete -f secrets/secret-parameter/aws/application.yaml
```

Azure

```
$ kubectl delete -f secrets/secret-parameter/azure/application.yaml
```