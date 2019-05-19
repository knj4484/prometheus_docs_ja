---
title: TLS暗号
sort_rank: 1
---

# TLS暗号でPrometheusのAPIとUIエンドポイントを安全にする

Prometheusは、expressionブラウザと[HTTP API](../../prometheus/latest/querying/api)の接続に対して、[Transport Layer Security](https://ja.wikipedia.org/wiki/Transport_Layer_Security) (TLS)暗号を直接サポートしていない。 
それらの接続にTLSを要求したいなら、Prometheusを[リバースプロキシ](https://www.nginx.com/resources/glossary/reverse-proxy-server/)と組み合わせて使い、プロキシレイヤーでTLSを適用することを推奨する。
自分の好きなリバースプロキシをPrometheusと共に利用することができるが、このガイドでは、[nginxの例](#nginx-example)を提供する。

NOTE: Prometheusインスタンス*への*TLS接続はサポートされていないが、Prometheus*から*[監視対象](../prometheus/latest/configuration/configuration/#<tls_config>)への接続ではTLSがサポートされている。

## nginxの例

ドメイン`example.com`でアクセス可能な[nginx](https://www.nginx.com/)サーバーの後ろでPrometheusインスタンスを運用し、全てのPrometheusのエンドポイントが`/prometheus`を通して利用可能にしたいとする。
つまり、Prometheusのエンドポイント`/metrics`に対する完全なURLは、以下のようになるだろう。

```
https://example.com/prometheus/metrics
```

また、既に、[OpenSSL](https://www.digitalocean.com/community/tutorials/openssl-essentials-working-with-ssl-certificates-private-keys-and-csrs)または類似ツールで以下のファイルを生成してあるとする。

* SSL証明書 `/root/certs/example.com/example.com.crt`
* SSLキー `/root/certs/example.com/example.com.key`

自己証明書と秘密鍵はこのコマンドで生成することができる。

```bash
mkdir -p /root/certs/example.com && cd /root/certs/example.com
openssl req \
  -x509 \
  -newkey rsa:4096 \
  -nodes \
  -keyout example.com.key \
  -out example.com.crt
```

プロンプトで適切な情報を入力し、`Common Name`のプロンプトには`example.com`を入力する。

## nginxの設定

設定ファイル[`nginx.conf`](https://www.nginx.com/resources/wiki/start/topics/examples/full/)の例を以下に示す。この設定で、nginxは、

* 用意した証明書と鍵でTLS暗号を要求する
* エンドポイント`/prometheus`への全ての接続を、同じホストで稼働しているPrometheusサーバーへプロキシする（同時にURLから`/prometheus`を削除する）

```conf
http {
    server {
        listen              443 ssl;
        server_name         example.com;
        ssl_certificate     /root/certs/example.com/example.com.crt;
        ssl_certificate_key /root/certs/example.com/example.com.key;

        location /prometheus {
            proxy_pass http://localhost:9090/;
        }
    }
}

events {}
```

nginxが443ポートをバインドする必要があるので、rootとしてnginxを起動する。

```bash
sudo nginx -c /usr/local/etc/nginx/nginx.conf
```

NOTE: この例では、`.htpasswd`を含むnginxの設定ファイルの場所として、`/etc/nginx`を用いるが、インストールの仕方によって異なるであろう。他の[よくあるnginxの設定ディレクトリ](http://nginx.org/en/docs/beginners_guide.html)は、`/usr/local/nginx/conf`と`/usr/local/etc/nginx`が挙げられる。

## Prometheusの設定

nginxプロキシの後ろでPrometheusを稼働させる際には、外部URLを`http://example.com/prometheus`に、ルートプリフィックスを`/`にセットする必要がある。

```bash
prometheus \
  --config.file=/path/to/prometheus.yml \
  --web.external-url=http://example.com/prometheus \
  --web.route-prefix="/"
```

## テスト

If you'd like to test out the nginx proxy locally using the `example.com` domain, you can add an entry to your `/etc/hosts` file that re-routes `example.com` to `localhost`:

```
127.0.0.1     example.com
```

そうすると、cURLを利用してローカルで構築したnginx/Prometheusに接続できる。

```bash
curl --cacert /root/certs/example.com/example.com.crt \
  https://example.com/prometheus/api/v1/label/job/values
```

フラグ`--insecure`または`-k`を使うことで、証明書を指定せずにnginxサーバーに接続することができる。

```bash
curl -k https://example.com/prometheus/api/v1/label/job/values
```
