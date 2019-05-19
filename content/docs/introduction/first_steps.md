---
title: はじめの一歩
sort_rank: 3
---

# Prometheusのはじめの一歩

Prometheusへようこそ！
Prometheusは、監視対象のHTTPエンドポイントをスクレイプすることで監視対象からメトリクスを収集する監視プラットフォームである。
このガイドでは、インストールの仕方、設定の仕方、Prometheusでリソースを監視する方法を示す。
Prometheusをダウンロード、インストール、実行することになる。
また、exporter（ホストとサービスについての時系列を出力するツール）をダウンロード、インストールする。
最初のexporterは、Prometheus自体（メモリ使用、ガベージコレクションなどのホストレベルの様々なメトリクスを提供する）となる。



## Prometheusのダウンロード

自分のプラットフォームに合った[最新リリースをダウンロード](/download)し、展開する。
```language-bash
tar xvfz prometheus-*.tar.gz
cd prometheus-*
```

Prometheusサーバーは、`prometheus`（Microsoft Windows上では`prometheus.exe`）という1つのバイナリである。
`--help`を渡すことで、バイナリを実行してオプションに関するヘルプを見ることができる。

```language-bash
./prometheus --help
usage: prometheus [<flags>]

The Prometheus monitoring server

. . .
```

Prometheusを起動する前に、設定をしよう。


## Prometheusの設定

Prometheusの設定は、[YAML](http://www.yaml.org/start.html)である。
Prometheusをダウンロードすると、`prometheus.yml`という設定例が付属している。

簡潔にするため、このサンプルファイルのほとんどのコメントを削除してある（コメントは`#`で始まる行である）。

```language-yaml
global:
  scrape_interval:     15s
  evaluation_interval: 15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
```

このサンプル設定ファイルには、3つの設定のブロック、`global`、`rule_files`、`scrape_configs`がある。

`global`ブロックは、Prometheusサーバーのグローバルな設定を制御する。
ここでは2つの項目が示されている。
1つ目の`scrape_interval`は、どれぐらい頻繁にPrometheusが監視対象をスクレイプするかを制御する。
これは、個別の監視対象についてこれを上書きすることもできる。
この例では、グローバルな設定としては、15秒ごとにスクレイプすることになる。
`evaluation_interval`は、どれぐらい頻繁にPrometheusがルールを評価するかを制御する。
Prometheusは、新しい時系列を作成したり、アラートを生成するためにルールを使う。

`rule_files`ブロックは、Prometheusサーバーに読み込んで欲しいルールの場所を指定する。
この例では、ルールはない。

最後のブロック`scrape_configs`は、どんなリソースをPrometheusが監視するかを制御する。
Prometheus自身もまた自分についてのデータをHTTPエンドポイントとして出力しているので、自分自身の状態を監視することができる。
デフォルトの設定では、`prometheus`というジョブがあり、Prometheusサーバーが出力する時系列データをスクレイプする。
そのジョブには、静的に設定された1つのターゲット、`localhost`のポート`9090`が含まれている。
Prometheusは、ターゲットのパス`/metrics`でメトリクスが取得可能であることを想定している。
したがって、このデフォルトジョブはURL[http://localhost:9090/metrics](http://localhost:9090/metrics)をスクレイプする。

ここで返される時系列データは、Prometheusサーバーの状態とパフォーマンスの詳細が含まれる。

設定項目の完全な仕様については、[設定のドキュメント](/docs/operating/configuration)を参照すること。

## Prometheusの起動

ここで作成した設定ファイルでPrometheusを起動するには、Prometheusのバイナリを含むディレクトリに移動し、以下のコマンドを実行する。

```language-bash
./prometheus --config.file=prometheus.yml
```

これでPrometheusが起動するはずである。
また、[http://localhost:9090](http://localhost:9090)で、Prometheus自体の状態を見ることができる。
自分自身のHTTPエンドポイントからデータを収集するため約30秒待つ。

メトリクスのエンドポイント[http://localhost:9090/metrics](http://localhost:9090/metrics)に移動することで、
Prometheusが出力しているメトリクスを確認することもできる。

## expressionブラウザの利用

Prometheusが自分自身について収集したデータを見てみよう。
Prometheusの組み込みexpressionブラウザーを利用するには、[http://localhost:9090/graph](http://localhost:9090/graph)に移動し、GraphタブのConsoleビューを選択する。

[http://localhost:9090/metrics](http://localhost:9090/metrics)から取得した通り、Prometheusが自分自身について出力したメトリクスの1つに`promhttp_metric_handler_requests_total`（Prometheusサーバーが返した`/metrics`へのリクエストの数）がある。expressionコンソールに以下の式を入力してみよう。

```
promhttp_metric_handler_requests_total
```

これは、いくつかの異なる時系列を、`promhttp_metric_handler_requests_total`というメトリック名、それぞれ異なるラベル、それぞれ記録された最新の値とともに返すはずである。
これらのラベルは異なるレスポンスステータスを示している。

もし、HTTPコード200になったリクエストに興味があるだけなら、その情報を取得するために以下のクエリを使うことができる。

```
promhttp_metric_handler_requests_total{code="200"}
```

返された時系列の数を数えるために、以下のように入力することができる。

```
count(promhttp_metric_handler_requests_total)
```

クエリ言語についてさらに知りたい場合は、[クエリ言語のドキュメント](/docs/querying/basics/)を参照すること。

## グラフ化インターフェースの利用

式をグラフ化するには、[http://localhost:9090/graph](http://localhost:9090/graph)に移動し、Graphタブを利用する。

例えば、ステータスコード200を返すHTTPリクエストの秒間レートをグラフ化するために、以下の式を入力してみよう。

```
rate(promhttp_metric_handler_requests_total{code="200"}[1m])
```

グラフ幅のパラメーターや他の設定を試すことができる。

## 他のターゲットの監視

Prometheusに何ができるかもっとよく理解するためには、他のexporterについてのドキュメントを調べてみることを推奨する。
まずは、[Node Exporterを用いたLinuxホストのメトリクス監視](/docs/guides/node-exporter)が良いだろう。

## まとめ

このガイドでは、Prometheusをインストールし、リソースを監視するようにPrometheusインスタンスを設定し、Prometheusのexpressionブラウザで時系列データを扱う基本を学んだ。
Prometheusを学習し続けるには、次に何を探すべきか理解するために[概要](/docs/introduction/overview)を見てみると良い。
