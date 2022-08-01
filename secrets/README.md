# Secrets

クラウドの機密情報管理サービス（AWS:SecretsManager / Azure:KeyVault / GoogleCloud:SecretManager）で管理している機密情報にKubernetesからアクセスする仕組みを提供する。

## 1. 概要

Kubernetesで機密情報を扱うための仕組みとしてSecretリソースがあるが、Secretのマニフェストは値をbase64エンコーディングして格納するだけで暗号化の仕組みはない。そのため、マニフェストをGit等で管理する場合Git上に機密情報が登録されるためリスクが高い。

そのため、Secretの管理機構としてexternal-secret-operatorを利用し、機密情報の実体をクラウド上の機密情報管理サービスで安全に管理した上で、KubernetesのSecretリソースを動的に作成する仕組みを導入する。

external-secret-operatorの構成図等は[公式ドキュメント](https://external-secrets.io/)を参照。

## 2. 導入

### external-secrets-operatorの導入

```bash
kubectl apply -f secrets/external-secrets/application.yaml
```

### SecretStoreをデプロイ

機密情報を格納しているサービスへのアクセス情報をデプロイする。ExternalSecretsはこのSecretStoreからアクセス情報を取得して機密情報にアクセスし、Secretリソースを作成する流れになる。全namespaceから共通で利用する場合はClusterSecretStoreリソース、namespaceごとにアクセスできるキーを絞る場合はSecretStoreリソースを利用する。なお、namespece単位のSecretStoreの場合はアクセスできる機密情報をSecretStoreごとに制限する。

サンプルではnamedspace単位のSecretStoreをデプロイする。

#### AWS（SecretsManager）

SecretStoreを作成する。

```bash
ACCOUNT_ID=<AWSアカウントID> && eval "echo \"$(cat secrets/external-secrets/aws/secretstore.yaml)\"" | kubectl apply -f -
```

なお、紐づくロールについてはnautible-infraプロジェクトのaws/app-ms/modules/common/main.tf内にあるapp_secret_access_role及びapp_secret_access_role_policyを参照。（事前にこのロール及びポリシーをTerraformで作成しておく）

#### Azure（AzureKeyVault）

external-secrets-operatorからAzure Key vaultへ接続するためのk8s secretおよびSecretStoreを作成する。CLIENTIDにはAzureコンソール＞AzureAD＞アプリのアプリケーション (クライアント) IDの値を設定。CLIENTSECRETにはAzureコンソール＞AzureAD＞アプリの登録＞証明書とシークレットでクライアントシークレットを登録して値を設定する。

```bash
kubectl create secret generic external-secrets-azure-credentials -n nautible-app-ms --from-literal=$CLIENTID --from-literal=$CLIENTSECRET
```

SecretStoreを作成する。

TENANT_IDにはAzureコンソール＞AzureAD＞テナントIDの値を設定、APP_MS_VAULT_URLにはAzureコンソール＞キー コンテナー＞nautibledevappms＞コンテナーのURIの値を設定する。

```bash
TENANT_ID=<テナントID> && APP_MS_VAULT_URL=<AzureKeyVaultURL> && eval "echo \"$(cat secrets/external-secrets/azure/secretstore.yaml)\"" | kubectl apply -f -
```

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

### ExternalSecretリソースの導入

app-msの稼働に必要なシークレットを導入する

AWS

```bash
kubectl apply -f secrets/secret-parameter/aws/application.yaml
```

Azure

```bash
kubectl apply -f secrets/secret-parameter/azure/application.yaml
```

## 3. 確認（AWSでの確認例）

### external-secrets-operatorの導入確認

```bash
kubectl get deploy -n external-secrets

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
external-secrets-operator                   1/1     1            1           3d1h
external-secrets-operator-cert-controller   1/1     1            1           3d1h
external-secrets-operator-webhook           1/1     1            1           3d1h
```

### ExternalSecretsおよびSecretの導入確認

```bash
kubectl get externalsecrets -n nautible-app-ms

NAME                                            AGE   STATUS
secretstore.external-secrets.io/secret-app-ms   3d    Valid

NAME                                                                   STORE           REFRESH INTERVAL   STATUS
externalsecret.external-secrets.io/secret-nautible-app-ms-order        secret-app-ms   1h                 SecretSynced
externalsecret.external-secrets.io/secret-nautible-app-ms-product-db   secret-app-ms   1h                 SecretSynced
```

```bash
kubectl get secrets -n nautible-app-ms

NAME                                TYPE                                  DATA   AGE
default-token-hh2jx                 kubernetes.io/service-account-token   3      20d
secret-nautible-app-ms-order        Opaque                                1      3d
secret-nautible-app-ms-product-db   Opaque                                2      3d
secretstore-token-qqwx8             kubernetes.io/service-account-token   3      3d
```

## 4. 削除

前提：事前にシークレットを利用しているアプリケーションの削除が完了していること

### AWS

- ArgoCDのコンソールよりsecret-parameterを削除

- SecretStoreのマニフェストを削除

```bash
kubectl delete -f secrets/external-secrets/aws/secretstore.yaml
```

- ArgoCDのコンソールよりexternal-secrets-operatorを削除

### Azure

- ArgoCDのコンソールよりsecret-parameterを削除

- SecretStoreのマニフェストを削除

```bash
kubectl delete -f secrets/external-secrets/azure/secretstore.yaml
```

- ArgoCDのコンソールよりexternal-secrets-operatorを削除
