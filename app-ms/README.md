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
|stockbatch|在庫管理(バッチ)|

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

[secretsのドキュメント](../secrets/README.md)の手順を参考に以下のリソースを作成する。

- AWS（SecretsManager）
  ```bash
  ACCOUNT_ID=<AWSアカウントID> && eval "echo \"$(cat app-ms/overlays/aws/secretstore.yaml)\"" | kubectl apply -f -
  ```
  ```bash
  kubectl appy -f app-ms/overlays/azure/secret-parameter/application.yaml
  ```

- Azure（AzureKeyVault）
  ```bash
  kubectl create secret generic external-secrets-azure-credentials -n nautible-app-ms --from-literal=clientid=$CLIENTID --from-literal=clientsecret=$CLIENTSECRET
  ```
  ```bash
  TENANT_ID=<テナントID> && APP_MS_VAULT_URL=<AzureKeyVaultURL> && eval "echo \"$(cat app-ms/overlays/azure/secretstore.yaml)\"" | kubectl apply -f -
  ```

### データ登録
- [商品サービス](https://github.com/nautible/nautible-app-ms-product/blob/main/testdata.md#b-dev%E7%92%B0%E5%A2%83)
- [在庫サービス](https://github.com/nautible/nautible-app-ms-stock/blob/feature/issue113/testdata.md#%E3%83%9E%E3%82%B9%E3%82%BF%E3%83%BC%E3%83%87%E3%83%BC%E3%82%BF%E7%99%BB%E9%8C%B2)

- 共通  
  - AWS  
    aws cliで以下を実行する
    ```bash
    aws dynamodb put-item --table-name Sequence --item '{ "Name": { "S": "CreditPayment" }, "SequenceNumber": { "N": "0" }}'
    aws dynamodb put-item --table-name Sequence --item '{ "Name": { "S": "Customer" }, "SequenceNumber": { "N": "0" }}'
    aws dynamodb put-item --table-name Sequence --item '{ "Name": { "S": "Delivery" }, "SequenceNumber": { "N": "0" }}'
    aws dynamodb put-item --table-name Sequence --item '{ "Name": { "S": "Order" }, "SequenceNumber": { "N": "0" }}'
    aws dynamodb put-item --table-name Sequence --item '{ "Name": { "S": "Payment" }, "SequenceNumber": { "N": "0" }}'
    aws dynamodb put-item --table-name Sequence --item '{ "Name": { "S": "Stock" }, "SequenceNumber": { "N": "0" }}'
    aws dynamodb put-item --table-name Sequence --item '{ "Name": { "S": "StockAllocateHistory" }, "SequenceNumber": { "N": "0" }}'
    ```

  - Azure  
    Azure CosmosDBコンソール＞データエクスプローラー＞Common選択＞NewShell
    以下を実行する
    ```
    db.Sequence.insertMany([
    { "_id": "CreditPayment","SequenceNumber":0},
    { "_id": "Customer","SequenceNumber":0},
    { "_id": "Delivery","SequenceNumber":0},
    { "_id": "Order","SequenceNumber":0},
    { "_id": "Payment","SequenceNumber":0},
    { "_id": "Stock","SequenceNumber":0},
    { "_id": "StockAllocateHistory","SequenceNumber":0}
    ])
    ```

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
