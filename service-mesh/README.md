# サービスメッシュ (Istio)

　サービスメッシュはマイクロサービスを接続・保護・監視する統一的な方法を提供します。  
　nautible では、オープンソースの Istio を利用してサービスメッシュを実現します。

## 1. フォルダ構成

```
service-mesh/
├ istio-operator/     : Istio Operator デプロイ用マニフェスト(*1)
├ istio-controlplane/ : コントロールプレーンデプロイ用マニフェスト(*1)
├ istio-base/         : カスタムリソースデプロイ用マニフェスト(*2)
├ istiod/             : istiod サービスデプロイ用マニフェスト(*2)
├ istio-ingress/      : Ingress Gateway デプロイ用マニフェスト(*2)
├ istio-egress/       : Egress Gateway デプロイ用マニフェスト(*2)
├ prometheus/         : Prometheus デプロイ用マニフェスト(*3)
├ grafana/            : Grafana デプロイ用マニフェスト(*3)
├ jaeger/             : Jaeger デプロイ用マニフェスト
├ kiali/              : Kiali デプロイ用マニフェスト
├ application.yaml    : ArgoCD アプリケーションのマニフェスト
├ kustomization.yaml  : デプロイする入れ子のアプリケーションの一覧
└ README.md           : 本ファイル
```

*1: オペレータインストールで使用  
*2: Helm によるインストールで使用  
*3: [Observation](../observation) で実現しているため未使用


## 2. セットアップ

　Istio のインストールには、`istioctl` コマンドの利用、オペレータインストール、Helm の利用などいくつか方法がありますが、nautible では ArgoCD から、オペレータインストールまたは Helm を利用したインストールを行っています。

### 2-1. オペレータインストールの場合

　以下のコマンド相当の処理を行っています。

```bash
$ kubectl create namespace istio-operator
$ helm install istio-operator manifests/charts/istio-operator \
    --set hub="docker.io/istio" --set tag="1.13.3"

$ kubectl create namespace istio-system
$ kubectl apply -f istio-controlplane.yaml
```

istio-controlplane.yaml は以下のようなファイルを用意します。

```yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: istio-controlplane
spec:
  profile: default
  components:
    egressGateways:
    - name: istio-egressgateway
      enabled: true
```

※ 公式ドキュメントに、1.9以降では `hub`、`tag` の指定は不要と記載されているが、values.yaml の設定が、zip ファイルでは `docker.io/istio`、`1.13.3` だが、GitHub のタグでは `gcr.io/istio-testing`、`latest` となっているため、ArgoCD で GitHub 上の Helm Chart を指定する場合、このオプションの指定が必要。

### 2-2. Helm によるインストールの場合

　以下のコマンド相当の処理を行っています。

```bash
$ kubectl create namespace istio-system
$ helm install istio-base manifests/charts/base -n istio-system
$ helm install istiod manifests/charts/istio-control/istio-discovery \
    -n istio-system \
    --set global.hub="docker.io/istio" --set global.tag="1.13.3"
```

※ 公式ドキュメントに、1.9以降では `global.hub`、`global.tag` の指定は不要と記載されているが、values.yaml の設定が、zip ファイルでは `docker.io/istio`、`1.13.3` だが、GitHub のタグでは `gcr.io/istio-testing`、`latest` となっているため、ArgoCD で GitHub 上の Helm Chart を指定する場合、このオプションの指定が必要。


## 3. Traffic management (トラフィック管理)

　以下の機能を提供します。詳細は [公式ドキュメント](https://istio.io/latest/docs/) を参照してください。

- Request Routing:  
  パス、HTTP ヘッダー等に応じたルーティング先の決定 (L4/L7)。  
  カスタムリソース VirtualService の `match` で指定。
- Traffic Shifting:  
  %ベースのルーティング先の振り分け。  
  カスタムリソース VirtualService の `weight` で指定。
- Timeouts:  
  リクエストに対して一定時間応答がない場合、エラーを返す。  
  カスタムリソース VirtualService の `timeout` で指定。
- Retries:  
  リクエストが失敗した場合、指定回数まで再試行を行う。  
  カスタムリソース VirtualService の `retries` で指定。
- Circuit breakers:  
  継続的な障害に対して、タイムアウトを待たずにエラーを返す。  
  カスタムリソース DestinationRule で指定。
- Mirroring:  
  トラフィックのコピーをミラーサービスへ転送する。  
  カスタムリソース VirtualService の `mirror` で指定。
- Fault Injection:  
  テストの目的で、特定の通信を遅延またはエラーにする。  
  カスタムリソース VirtualService の `fault` で指定。


## 4. Security (セキュリティ)

　調査中。


## 5. Observability (可観測性)

### 5-1. Metrics

　Prometheus Operator でインストールした Prometheus で Istio のメトリクスを収集しています。
[observation/monitors/base/istio-monitors.yaml](../observation/monitors/base/istio-monitors.yaml) に設定を記述しています。

```bash
$ kubectl port-forward svc/prometheus-operated -n monitoring 9090:9090
```

を実行後、ブラウザで <http://localhost:9000> にアクセスすると、Prometheus のダッシュボードが起動します。

```bash
$ kubectl port-forward svc/grafana -n monitoring 3000:3000
```

を実行後、ブラウザで <http://localhost:3000> にアクセスすると、Grafana が起動します。

　Grafana が起動したら、以下のダッシュボードをインストールしてください。

- Istio Control Plane Dashboard: 7645
- Istio Mesh Dashboard: 7639
- Istio Performance Dashboard: 11829
- Istio Service Dashboard: 7636
- Istio Wasm Extension Dashboard: 13277
- Istio Workload Dashboard: 7630

### 5-2. Logs

　Istio のインストール時に `meshConfig.accessLogFile` を指定すると、Envoy のアクセスログが有効になります。

オペレータインストールの場合:

```yaml
# istio-controlplane.yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: istio-controlplane
spec:
  profile: default
  meshConfig:
    accessLogFile: /dev/stdout
    accessLogEncoding: JSON
```

Helm によるインストールの場合:

```bash
$ helm install istiod manifests/charts/istio-control/istio-discovery \
    -n istio-system \
    --set meshConfig.accessLogFile="/dev/stdout" \
    --set meshConfig.accessLogEncoding="JSON"
```

### 5-3. Distributed Tracing

　以下のコマンドで Jaeger が起動します。

```bash
$ istioctl dashboard jaeger
```

または、

```bash
$ kubectl port-forward svc/tracing -n istio-system 16686:80
```

を実行後、ブラウザで <http://localhost:16686> にアクセス。

　Istio のインストール時またはアプリケーションのデプロイ時に設定を変更できます。

オペレータインストールの場合:

```yaml
# istio-controlplane.yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: istio-controlplane
spec:
  profile: default
  meshConfig:
    defaultConfig:
      tracing:
        sampling: 20
```

Helm によるインストールの場合:

```bash
$ helm install istiod manifests/charts/istio-control/istio-discovery \
    -n istio-system \
    --set meshConfig.defaultConfig.tracing.sampling="20"
```

### 5-4. Visualizing Your Mesh

　以下のコマンドで Kiali が起動します。

```bash
$ istioctl dashboard kiali
```

または、

```bash
$ kubectl port-forward svc/kiali -n istio-system 20001:20001
```

を実行後、ブラウザで <http://localhost:20001> にアクセス。


## 6. 参考文献・URL

- Istio 公式サイト  
  <https://istio.io/>
- Istio Docs  
  <https://istio.io/latest/docs/>
- Istio - GitHub  
  <https://github.com/istio/istio>

