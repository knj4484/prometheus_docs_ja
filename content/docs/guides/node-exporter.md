---
title: Node Exporterを用いたLinuxホストのメトリクス監視
---

# Node Exporterを用いたLinuxホストのメトリクス監視

[**Node Exporter**](https://github.com/prometheus/node_exporter)は、ハードウェアとカーネル関係の幅広いメトリクスをexposeする。

このガイドでは、

* `localhost`でNode Exporterを起動する
* そのNode Exporterからメトリクスをscrapeするように設定したPrometheusインスタンスを`localhost`で起動する

NOTE: Node Exporterが*nixのためにあるのに対して、Windowsに対して[WMI exporter](https://github.com/martinlindhe/wmi_exporter)が同じような目的で使われる

## Node Exporterのインストールと実行

Node Exporterは、[tarballからインストール](#tarball-installation)できる単一のスタティックなバイナリである。
Prometheusの[ダウンロードページ](/download#node_exporter)からtarballをダウンロード、展開し、実行する。

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v*/node_exporter-*.*-amd64.tar.gz
tar xvfz node_exporter-*.*-amd64.tar.gz
cd node_exporter-*.*-amd64
./node_exporter
```

以下のように、Node Exporterが動いていて9100ポートでメトリクスをexposeしていることを示す出力が見られるはずである。

```
INFO[0000] Starting node_exporter (version=0.16.0, branch=HEAD, revision=d42bd70f4363dced6b77d8fc311ea57b63387e4f)  source="node_exporter.go:82"
INFO[0000] Build context (go=go1.9.6, user=root@a67a9bc13a69, date=20180515-15:53:28)  source="node_exporter.go:83"
INFO[0000] Enabled collectors:                           source="node_exporter.go:90"
INFO[0000]  - boottime                                   source="node_exporter.go:97"
...
INFO[0000] Listening on :9100                            source="node_exporter.go:111"
```

## Node Exporterのメトリクス

Node Exporterがインストール、実行されると、curlでエンドポイント`/metrics`にアクセスすることで、メトリクスが出力されていることを確認できる。

```bash
curl http://localhost:9100/metrics
```

以下のような出力が見られるはずである。

```
# HELP go_gc_duration_seconds A summary of the GC invocation durations.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 3.8996e-05
go_gc_duration_seconds{quantile="0.25"} 4.5926e-05
go_gc_duration_seconds{quantile="0.5"} 5.846e-05
# etc.
```

成功！これでPrometheusがscrapeできるメトリクスをNode Exporterがexposeしており、出力のかなり下の方に`node_`で始まるシステムの様々なメトリクスが含まれている。
これらのシステムメトリクスを（ヘルプおよび型情報と共に）見るためには以下のコマンドを実行する

```bash
curl http://localhost:9100/metrics | grep "node_"
```

## Prometheusの設定

Node Exporterのメトリクスにアクセスするために、ローカルで実行しているPrometheusインスタンスが適切に設定されている必要がある。
設定ファイル`prometheus.yml`にある以下の[`scrape_config`](../prometheus/latest/configuration/configuration/#<scrape_config>)は、`localhost:9100`を通してNode Exporterからメトリクスを取得するように指定している。

<a id="config"></a>

```yaml
scrape_configs:
- job_name: 'node'
  static_configs:
  - targets: ['localhost:9100']
```

Prometheusをインストールするには、自分のプラットフォームの[最新リリースをダウンロード](/download)し、tarを展開する。

```bash
wget https://github.com/prometheus/prometheus/releases/download/v*/prometheus-*.*-amd64.tar.gz
tar xvf prometheus-*.*-amd64.tar.gz
cd prometheus-*.*
```

インストールが終わったら、[上](#config)で作成した設定ファイルを`--config.file`フラグで指定することでPrometheusを起動することが出来る。

```bash
./prometheus --config.file=./prometheus.yml
```

## expressionブラウザによるNode Exporterメトリクスの調査

これでPrometheusがNode Exporterインスタンスからメトリクスを取得しているので、PrometheusのUI（[expressionブラウザ](/docs/visualization/expression-browser)として知られる）を使ってそれらのメトリクスを調べることが出来る。
ブラウザで`localhost:9090/graph`を開いて、ページ上部のmain expression barを使って、expressionを入力する。
expression barは、このように見える。

![Prometheus expressions browser](/assets/prometheus-expression-bar.png)

Node Exporter特有のメトリクスは、`node_cpu_seconds_total`や`node_exporter_build_info`のように`node_`というプリフィックスが付いている。

下記のリンクをクリックすると、いくつかのメトリクスの例を見ることが出来る。

Metric | Meaning
:------|:-------
[`rate(node_cpu_seconds_total{mode="system"}[1m])`](http://localhost:9090/graph?g0.range_input=1h&g0.expr=rate(node_cpu_seconds_total%7Bmode%3D%22system%22%7D%5B1m%5D)&g0.tab=1) | 新1分間のシステムモードの秒間CPU消費時間の平均
[`node_filesystem_avail_bytes`](http://localhost:9090/graph?g0.range_input=1h&g0.expr=node_filesystem_avail_bytes&g0.tab=1) | rootユーザー以外に利用可能なファイルシステムの空き
[`rate(node_network_receive_bytes_total[1m])`](http://localhost:9090/graph?g0.range_input=1h&g0.expr=rate(node_network_receive_bytes_total%5B1m%5D)&g0.tab=1) | 最新1分間の秒間ネットワークトラフィック平均(byte)
