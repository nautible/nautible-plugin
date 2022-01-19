
# Metrics Server

クラスタ内のリソース使用状況を取得するサーバー

## 対応環境

|環境|対応|
|:--|:--|
|AWS|〇|
|Azure|〇|
|GCP|-|

## 導入

```
$ kubectl apply -f metrics-server/application.yaml
```

## 確認

コマンド
```
$kubectl get deployment metrics-server -n kube-system
```

出力
```
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
metrics-server   1/1     1            1           6m
```

## 削除

```
$ kubectl delete -f metrics-server/application.yaml
```
