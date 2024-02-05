# Secrets

クラウドの機密情報管理サービス（AWS: AWS Secrets Manager / Azure: Azure Key Vault / GoogleCloud: SecretManager）で管理している機密情報にKubernetesからアクセスする仕組みを提供する。

## 1. 概要

Kubernetesで機密情報を扱うための仕組みとしてSecretリソースがあるが、Secretのマニフェストは値をbase64エンコーディングして格納するだけで暗号化の仕組みはない。そのため、マニフェストをGit等で管理する場合Git上に機密情報が登録されるためリスクが高い。

そこで、Secretの管理機構としてExternal Secretsを利用し、機密情報の実体をクラウド上の機密情報管理サービスで安全に管理した上で、KubernetesのSecretリソースを動的に作成する仕組みを導入する。

External Secretsの構成図等は[公式ドキュメント](https://external-secrets.io/)を参照。

## 2. external-secrets-operatorの導入

External Secretsを利用するための公式オペレーターを導入する。

```bash
kubectl apply -f secrets/application.yaml
```

## 3. シークレットのデプロイ

[リファレンスアプリケーション](https://github.com/nautible/docs/blob/main/referenceapp-architecture/README.md)で利用するシークレットのデプロイを例に導入手順を記載する。

なお、本手順ではSecretをnautible-app-msネームスペースにデプロイする。実行時にnautible-app-msネームスペースがまだない場合、先にネームスペースを作成する。

```bash
kubectl create ns nautible-app-ms
```

### AWS（AWS Secrets Manager）

#### IAMの作成

KubernetesからAWS Secrets Managerへアクセスするためのロール及びポリシーを作成する。

ロール及びポリシーの作成はTerraformで実施する。デプロイ手順は[nautible-infra/aws/app-ms](https://github.com/nautible/nautible-infra/tree/main/aws/app-ms)を参照

#### ServiceAccount、SecretStoreのデプロイ

[リファレンスアプリケーション](https://github.com/nautible/docs/blob/main/referenceapp-architecture/README.md)用のServiceAccount、SecretStoreをデプロイする。

- ServiceAccount
  - 上記で作成したIAMロールをKubernetes上のリソースで利用するためのServiceAccount
- SecretStore
  - AWS Secrets Managerへアクセスするリソース。ServiceAccountを紐づけて利用する。全namespaceから共通で利用する場合はClusterSecretStoreリソース、namespaceごとにアクセスできるキーを絞る場合はSecretStoreリソースとなる。アクセスできるシークレットの範囲はServiceAccountに紐づけているIAMロールのポリシーで制御する。

※ 定義はapp-ms/overlays/aws/secretstore.yamlを参照。

```bash
ACCOUNT_ID=<AWSアカウントID> && eval "echo \"$(cat app-ms/overlays/aws/secretstore.yaml)\"" | kubectl apply -f -
```

#### ExternalSecretのデプロイ

AWS Secrets Managerの値をKubernetesのSecretリソースにマッピングするExternalSecretをデプロイする。

[リファレンスアプリケーション](https://github.com/nautible/docs/blob/main/referenceapp-architecture/README.md)では、必要なシークレットをkustomizeファイルで定義してArgoCDで管理する方式をとっているため、application.yamlをデプロイする。

```bash
kubectl apply -f app-ms/overlays/aws/secret-parameter/application.yaml
```

### Azure（Azure Key Vault）

#### Azure Key Vaultへ接続するためのSecretの作成

external-secrets-operatorからAzure Key vaultへ接続するためのk8s secretおよびSecretStoreを作成する。CLIENTIDにはAzureコンソール＞AzureAD＞アプリのアプリケーション (クライアント) IDの値を設定。CLIENTSECRETにはAzureコンソール＞AzureAD＞アプリの登録＞証明書とシークレットでクライアントシークレットを登録して値を設定する。

```bash
kubectl create secret generic external-secrets-azure-credentials -n nautible-app-ms --from-literal=clientid=$CLIENTID --from-literal=clientsecret=$CLIENTSECRET
```

#### SecretStoreのデプロイ

[リファレンスアプリケーション](https://github.com/nautible/docs/blob/main/referenceapp-architecture/README.md)用のSecretStoreをデプロイする。

TENANT_IDにはAzureコンソール＞AzureAD＞テナントIDの値を設定、APP_MS_VAULT_URLにはAzureコンソール＞キー コンテナー＞nautibledevappms＞コンテナーのURIの値を設定する。

※ 定義はapp-ms/overlays/azure/secretstore.yamlを参照。

```bash
TENANT_ID=<テナントID> && APP_MS_VAULT_URL=<AzureKeyVaultURL> && eval "echo \"$(cat app-ms/overlays/azure/secretstore.yaml)\"" | kubectl apply -f -
```

#### ExternalSecretのデプロイ

Azure Key Vaultの値をKubernetesのSecretリソースにマッピングするExternalSecretをデプロイする。

リファレンスアプリケーションでは、必要なシークレットをkustomizeファイルで定義してArgoCDで管理する方式をとっているため、application.yamlをデプロイする。

```bash
kubectl apply -f app-ms/overlays/azure/secret-parameter/application.yaml
```

## 4. 確認


### external-secrets-operatorの導入確認

```bash
kubectl get deploy -n external-secrets
```

<pre>
NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
external-secrets-operator                   1/1     1            1           3d1h
external-secrets-operator-cert-controller   1/1     1            1           3d1h
external-secrets-operator-webhook           1/1     1            1           3d1h
</pre>

### Secretの確認

```bash
kubectl get secret -n nautible-app-ms
```

<pre>
NAME                                TYPE     DATA   AGE
secret-nautible-app-ms-order        Opaque   1      7m4s
secret-nautible-app-ms-product-db   Opaque   2      7m4s
</pre>

## 5. 削除

### ExternalSecretの削除

注）Secretを利用しているアプリケーションがないことを確認の上削除すること。

ArgoCDのコンソール画面よりsecret-app-msの削除を行う。

コマンドラインによる削除を行う場合は、Argo CD CLIを使用してApplicationリソースを削除する。

```bash
argocd app delete argocd/secret-app-ms
```


### SecretStoreの削除

#### AWS

```bash
kubectl delete -f app-ms/overlays/aws/secretstore.yaml
```

#### Azure

```bash
kubectl delete -f app-ms/overlays/azure/secretstore.yaml
```

### external-secrets-operatorの削除

ArgoCDのコンソール画面よりexternal-secrets-operatorの削除を行う。

コマンドラインによる削除を行う場合は、Argo CD CLIを使用してApplicationリソースを削除する。

```bash
argocd app delete argocd/external-secrets-operator
```
