# Karpenter

## 1. 概要

データプレーン（ワーカーノード）のオートスケール機能を導入する。

XXX

## 2. 導入

### リソース準備

### コントローラー

Karpenter のカスタムコントローラーを導入する。
導入先ネームスペースは Terraform の Karpenter モジュールと合わせておくこと。（Terraform で環境構築時に指定したネームスペースに karpenter 用の PodIdentity サービスアカウントが導入されるので、コントローラーの導入先を合わせる必要がある）
なお、Terraform で未指定の場合は kube-system となる。

```bash
helm registry logout public.ecr.aws
export KARPENTER_NAMESPACE=kube-system
export KARPENTER_VERSION=0.37.0
helm upgrade --install karpenter oci://public.ecr.aws/karpenter/karpenter --version "${KARPENTER_VERSION}" --namespace "${KARPENTER_NAMESPACE}" --create-namespace -f ./karpenter/values.yaml --wait
```

### CRD

EC2NodeClass および NodePool のカスタムリソースを導入する。
どのようなノードスペックを対象とするかは NodePool の設定でカスタマイズする。

```bash
kubectl apply -f karpenter/node.yaml
```

## 3. 確認

Karpenter の Pod ログを表示し、エラーが出ていないことを確認しておく。

```bash
kubectl logs -n kube-system <KarpenterのPod>
```

エラーが出ていないことが確認出来たら、任意のアプリケーションやエコシステムを導入してみて新規にノードが作成されることを確認する。

## 4. 削除

XXX
