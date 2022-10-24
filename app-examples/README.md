
# app-examples

## 1. 概要

サンプルアプリケーション

### ディレクトリ構成

base

|ディレクトリ名|内容|
|:--|:--|
|common|共通設定（namespace）|
|examples-java|Javaのサンプルアプリケーション設定|

overlays

|ディレクトリ名|内容|
|:--|:--|
|aws|AWSへの導入アプリケーション|

## 2. 導入

aws

```
$ kubectl apply -f app-examples/overlays/aws/application.yaml
```

## 3. 確認

```
$ kubectl get deploy -n nautible-app-examples
NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
nautible-app-examples-java         2/2     2            2           1d
```

## 4. 削除

aws

```
$ kubectl delete -f app-examples/overlays/aws/application.yaml
```
