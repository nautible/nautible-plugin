# サンプルアプリケーション

kong-apigatewayで受信したリクエストを流すサンプルアプリケーション

## configmap.yaml

configmapに以下2つのhtmlデータを格納

- index-html
  - index.htmlを受信した際に応答するHTMLファイル
- healthz-html
  - healthz.htmlを受信した際に応答するHTMLファイル

## deployment.yaml

nginx:alpineコンテナを起動  
応答HTMLはconfigmapで定義したものを/usr/share/nginx/html/kong/にマウントしている

## service.yaml

8080番ポートで待ち受けて、nginxにリクエストを流すService
