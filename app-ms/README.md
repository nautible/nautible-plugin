# app-ms

## 1. 概要

マイクロサービスサンプルアプリケーション

### ディレクトリ構成

base

|ディレクトリ名|内容|
|:--|:--|
|common|共通設定（namespace,role）|
|customer|ユーザー管理|
|delivery|出荷管理|
|order|受注管理|
|payment|支払い管理|
|product|商品管理|
|stock|在庫管理|

overlays

|ディレクトリ名|内容|
|:--|:--|
|aws|AWSへの導入アプリケーション|
|azure|AZureへの導入アプリケーション|

## 2. 導入

### クラウドサービスへシークレットを登録する

app-msの稼働に必要なシークレットを登録する。AWSの場合はSecretsManager、Azureの場合はAzureKeyvaultに登録する。

| name | value | aws/azure | 備考 |
| ---- | ---- | ---- | ---- |
| nautible-app-ms-product-db-user | 商品サービスDBのユーザー | aws/azure | |
| nautible-app-ms-product-db-password | 商品サービスDBのパスワード | aws/azure | |
| nautible-app-ms-order-dapr-statestore-password | 注文サービスelasticacheのパスワード（Token） | aws/azure | |
| nautible-app-ms-cosmosdb-user | Cosmosdbのアクセスユーザー | azure | |
| nautible-app-ms-cosmosdb-password | Cosmosdbのパスワード | azure | |
| nautible-app-ms-servicebus-connectionstring| Azure Servicebus 接続文字列  | azure | Azureの管理コンソール＞Service Bus＞共有アクセスポリシー＞RootManageSharedAccessKey 参照 |

※AzureのKeyvaultのシークレット編集方法については[こちら](../docs/azure/keyvault/README.md)を参照してください
### SecretStoreの導入

手順は[secretsのドキュメント](../secrets/README.md)を参照

### アプリケーションの導入

AWS

```bash
kubectl apply -f app-ms/overlays/aws/application.yaml
```

Azure

```bash
kubectl apply -f app-ms/overlays/azure/application.yaml
```

## 3. 確認

```bash
kubectl get deploy -n nautible-app-ms

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
nautible-app-ms-customer              2/2     2            2           18d
nautible-app-ms-delivery              2/2     2            2           18d
nautible-app-ms-order                 2/2     2            2           18d
nautible-app-ms-payment-bff           1/1     1            1           18d
nautible-app-ms-payment-cash          1/1     1            1           18d
nautible-app-ms-payment-convenience   1/1     1            1           18d
nautible-app-ms-payment-credit        1/1     1            1           18d
nautible-app-ms-product               1/1     1            1           18d
nautible-app-ms-stock                 2/2     2            2           18d
```

## 4. 削除

### アプリケーションの削除

ArgoCDのコンソールより各アプリケーションおよびapplication-rootを削除

### ExternalSecretsおよびSecretの削除

ArgoCDのコンソールよりsecret-app-msを削除

### SecretStoreの削除

手順は[secretsのドキュメント](../secrets/README.md)を参照
