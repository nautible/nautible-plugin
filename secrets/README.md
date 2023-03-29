# Secrets

クラウドの機密情報管理サービス（AWS:SecretsManager / Azure:KeyVault / GoogleCloud:SecretManager）で管理している機密情報にKubernetesからアクセスする仕組みを提供する。

## 1. 概要

Kubernetesで機密情報を扱うための仕組みとしてSecretリソースがあるが、Secretのマニフェストは値をbase64エンコーディングして格納するだけで暗号化の仕組みはない。そのため、マニフェストをGit等で管理する場合Git上に機密情報が登録されるためリスクが高い。

そのため、Secretの管理機構としてexternal-secret-operatorを利用し、機密情報の実体をクラウド上の機密情報管理サービスで安全に管理した上で、KubernetesのSecretリソースを動的に作成する仕組みを導入する。

external-secret-operatorの構成図等は[公式ドキュメント](https://external-secrets.io/)を参照。

## 2. 導入

### external-secrets-operatorの導入

```bash
kubectl apply -f secrets/application.yaml
```

### SecretStoreをデプロイ

機密情報を格納しているサービスへのアクセス情報をデプロイする。ExternalSecretsはこのSecretStoreからアクセス情報を取得して機密情報にアクセスし、Secretリソースを作成する流れになる。全namespaceから共通で利用する場合はClusterSecretStoreリソース、namespaceごとにアクセスできるキーを絞る場合はSecretStoreリソースを利用する。なお、namespece単位のSecretStoreの場合はアクセスできる機密情報をSecretStoreごとに制限する。

サンプルではnamedspace単位のSecretStoreをデプロイする。

#### AWS（SecretsManager）

SecretStoreを作成する。

```bash
ACCOUNT_ID=<AWSアカウントID> && eval "echo \"$(cat <secretstore.yamlへのパス>)\"" | kubectl apply -f -

# 例
ACCOUNT_ID=<AWSアカウントID> && eval "echo \"$(cat app-ms/overlays/aws/secretstore.yaml)\"" | kubectl apply -f -
```

なお、紐づくロールについてはnautible-infraプロジェクトのaws/app-ms/modules/common/main.tf内にあるapp_secret_access_role及びapp_secret_access_role_policyを参照。（事前にこのロール及びポリシーをTerraformで作成しておく）

#### Azure（AzureKeyVault）

external-secrets-operatorからAzure Key vaultへ接続するためのk8s secretおよびSecretStoreを作成する。CLIENTIDにはAzureコンソール＞AzureAD＞アプリのアプリケーション (クライアント) IDの値を設定。CLIENTSECRETにはAzureコンソール＞AzureAD＞アプリの登録＞証明書とシークレットでクライアントシークレットを登録して値を設定する。

```bash
kubectl create secret generic external-secrets-azure-credentials -n nautible-app-ms --from-literal=clientid=$CLIENTID --from-literal=clientsecret=$CLIENTSECRET
```

SecretStoreを作成する。

TENANT_IDにはAzureコンソール＞AzureAD＞テナントIDの値を設定、APP_MS_VAULT_URLにはAzureコンソール＞キー コンテナー＞nautibledevappms＞コンテナーのURIの値を設定する。

```bash
TENANT_ID=<テナントID> && APP_MS_VAULT_URL=<AzureKeyVaultURL> && eval "echo \"$(cat <secretstore.yamlへのパス>)\"" | kubectl apply -f -

# 例
TENANT_ID=<テナントID> && APP_MS_VAULT_URL=<AzureKeyVaultURL> && eval "echo \"$(cat app-ms/overlays/aws/secretstore.yaml)\"" | kubectl apply -f -
```

## 3. 確認


### external-secrets-operatorの導入確認（AWSでの確認例）

```bash
kubectl get deploy -n external-secrets

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
external-secrets-operator                   1/1     1            1           3d1h
external-secrets-operator-cert-controller   1/1     1            1           3d1h
external-secrets-operator-webhook           1/1     1            1           3d1h
```

## 4. 削除

### SecretStoreの削除

```bash
kubectl delete -f <secretstore.yamlへのパス>

# 例
kubectl delete -f app-ms/overlays/aws/secretstore.yaml
```

### external-secrets-operatorの削除

ArgoCDのコンソールよりexternal-secrets-operatorを削除
