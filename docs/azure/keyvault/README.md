# Keyvault

## Keyvaultのリソース編集

Keyvaultをセキュアに管理するためパブリックアクセスを無効にして、プライベートエンドポイントからアクセスできるように構成している。Azureコンソールから操作もブロックされるため以下に記載する方法で編集する。

- AKSにAzureのcli用のコンテナを作成しAzコマンドを実行できるようにする
```bash
$ kubectl run azure-cli --tty -i --rm=true --image=mcr.microsoft.com/azure-cli sh
```

- az loginを実行し、コンソールに表示される方法でログインする
```bash
$ az login
```

- az cliでkeyvaultのリソースを編集する。以下は、シークレットの作成・更新。
```bash
$ az keyvault secret set --vault-name $KeyvaultName --name $SecretName  --value $SecretValue
```

- az cliでシークレットの一覧取得
```bash
$ az keyvault secret list --vault-name $KeyvaultName
```

- az cliでシークレットの表示
```bash
$ az keyvault secret show --vault-name $KeyvaultName  --name $SecretName
```
