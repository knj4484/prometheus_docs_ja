---
title: Basic認証
---

# Basic認証でPrometheusのAPIとUIを安全にする

Prometheusは、expressionブラウザとHTTP APIの接続に対して、[Basic認証](https://ja.wikipedia.org/wiki/Basic認証)を直接サポートしていない。
それらの接続にBasic認証を要求したいなら、Prometheusをリバースプロキシと組み合わせて使い、プロキシレイヤーで認証を適用することを推奨する。
自分の好きなリバースプロキシをPrometheusと共に利用することができるが、このガイドでは、[nginxの例](nginxの例)を提供する。

NOTE: PrometheusインスタンスへのBasic認証での接続はサポートされていないが、Prometheus*から*[監視対象](../prometheus/latest/configuration/configuration/#<scrape_config>)への接続ではBasic認証がサポートされている。

## nginxの例

`localhost:12321`で稼働しているnginxサーバーの後ろでPrometheusインスタンスを運用し、全てのPrometheusのエンドポイントが`/prometheus`を通して利用可能にしたいとする。
つまり、Prometheusのエンドポイント`/metrics`に対する完全なURLは、以下のようになるだろう。

```
http://localhost:12321/prometheus/metrics
```

また、Prometheusインスタンスにアクセスする全てのユーザーにユーザー名とパスワードを要求したいとする。
この例では、ユーザー名として`admin`を用い、好きなパスワードを選ぶこととする。

まず、[`htpasswd`](https://httpd.apache.org/docs/2.4/programs/htpasswd.html)を利用し、ユーザー名/パスワードを保存するためのファイル`.htpasswd`を作成し、ディレクトリ`/etc/nginx`に保存する。

```bash
mkdir -p /etc/nginx
htpasswd -c /etc/nginx/.htpasswd admin
```

NOTE: この例では、`.htpasswd`を含むnginxの設定ファイルの場所として、`/etc/nginx`を用いるが、インストールの仕方によって異なるであろう。他のよくあるnginxの設定ディレクトリは、`/usr/local/nginx/conf`と`/usr/local/etc/nginx`が挙げられる。

## nginxの設定

設定ファイル[`nginx.conf`](https://www.nginx.com/resources/wiki/start/topics/examples/full/)の例を以下に示す。
この設定で、nginxは、Prometheusへプロキシするエンドポイント`/prometheus`への全ての接続にBasic認証を要求するようになる。

```conf
http {
    server {
        listen 12321;

        location /prometheus {
            auth_basic           "Prometheus";
            auth_basic_user_file /etc/nginx/.htpasswd;

            proxy_pass           http://localhost:9090/;
        }
    }
}

events {}
```

上記の設定を用いて、nginxを起動する。

```bash
nginx -c /etc/nginx/nginx.conf
```

## Prometheusの設定

nginxプロキシの後ろでPrometheusを稼働させる際には、外部URLを`http://localhost:12321/prometheus`に、ルートプリフィックスを`/`にセットする必要がある。


```bash
prometheus \
  --config.file=/path/to/prometheus.yml \
  --web.external-url=http://localhost:12321/prometheus \
  --web.route-prefix="/"
```

## テスト

cURLを利用してローカルで構築したnginx/Prometheusに接続できる。このリクエストで試してみよう。

```bash
curl --head http://localhost:12321/prometheus/graph
```

これは、正当なユーザー名とパスワードを提供していないので、レスポンス`401 Unauthorized`を返すだろう。
このレスポンスは、ヘッダー`WWW-Authenticate: Basic realm="Prometheus"`も返すだろう。
これは、パラメーター`auth_basic`で指定されたBasic認証の領域`Prometheus`が要求されていることを示す。

Basic認証によるPrometheusエンドポイントへのアクセスが成功するには、適切なユーザー名をフラグ`-u`で指定し、プロンプトでパスワードを入力する。

```bash
curl -u admin http://localhost:12321/prometheus/metrics
Enter host password for user 'admin':
```

何か以下のようなPrometheusの出力を返すはずである。

```
# HELP go_gc_duration_seconds A summary of the GC invocation durations.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0.0001343
go_gc_duration_seconds{quantile="0.25"} 0.0002032
go_gc_duration_seconds{quantile="0.5"} 0.0004485
...
```

## まとめ

このガイドでは、ユーザー名とパスワードを`.htpasswd`に保存し、PrometheusのHTTPエンドポイントにアクセスするユーザーを認証するためにそのファイルの情報を使うようにnginxを設定し、Prometheusをリバースプロキシのために設定した。
