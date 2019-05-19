---
title: 監視対象検出のためのファイルベースサービスディスカバリーの利用
---

# 監視対象検出のためのファイルベースサービスディスカバリーの利用

Prometheusは、監視対象を検出するための[様々なサービスディスカバリーの選択肢](https://github.com/prometheus/prometheus/tree/master/discovery)（[Kubernetes](/docs/prometheus/latest/configuration/configuration/#<kubernetes_sd_config>)や[Consul](/docs/prometheus/latest/configuration/configuration/#<consul_sd_config>)、その他）を提供している。
現在サポートされていないサービスディスカバリーを利用する必要がある場合には、Prometheusの[ファイルベースのサービスディスカバリー](/docs/prometheus/latest/configuration/configuration/#<file_sd_config>)が最も役立つであろう。
ファイルベースのサービスディスカバリーによって、JSONファイルで監視対象をメタデータと共にリストすることが可能になる。

このガイドでは、

* ローカルでPrometheus Node Exporterをインストール、実行する
* そのNode Exporterのホストとポートを指定する`targets.json`を作成する
* その`targets.json`を使ってNode Exporterを検出するように設定されたPrometheusインスタンスをインストール、実行する

## Node Exporterのインストールと実行

[Node Exporterを用いたLinuxホストのメトリクス監視](../node-exporter)の[Node Exporterのインストールと実行](../node-exporter#installing-and-running-the-node-exporter)の部分を参照して行う。

```bash
curl http://localhost:9100/metrics
```

出力されるメトリクスは、このようになるはずである。

```
# HELP go_gc_duration_seconds A summary of the GC invocation durations.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0
go_gc_duration_seconds{quantile="0.25"} 0
go_gc_duration_seconds{quantile="0.5"} 0
...
```

## Prometheusのインストールと設定、実行

Node Exporterと同じように、Prometheusは一つの静的なバイナリで、tarballからインストールできる。
プラットフォームに合った[最新リリースをダウンロード](/download#prometheus)し、展開する。

```bash
wget https://github.com/prometheus/prometheus/releases/download/v*/prometheus-*.*-amd64.tar.gz
tar xvf prometheus-*.*-amd64.tar.gz
cd prometheus-*.*
```

展開したディレクトリには設定ファイル`prometheus.yml`がある。そのファイルを以下の内容に置き換える。

```yaml
scrape_configs:
- job_name: 'node'
  file_sd_configs:
  - files:
    - 'targets.json'
```

この設定は、`node`という名前（Node Exporterのため）のジョブがあり、そのジョブが、`targets.json`というファイルからNode Exporterインスタンスのホストとポートの情報を取得するように指定している。

`targets.json`というファイルを作成し、以下の内容を追加する。

```json
[
  {
    "labels": {
      "job": "node"
    },
    "targets": [
      "localhost:9100"
    ]
  }
]
```

NOTE: このガイドでは、簡潔にするために、サービスディスカバリーの設定のJSONを手動で編集しているが、JSONを生成するプロセスやツールを代わりに使うことを推奨する。

この設定は、一つの対象`localhost:9100`を持つ`node`というジョブを指定している。

これで、Prometheusを起動することができる。

```bash
./prometheus
```

Prometheusがうまく起動したら、ログに下記のような行が見られるはずである。

```
level=info ts=2018-08-13T20:39:24.905651509Z caller=main.go:500 msg="Server is ready to receive web requests."
```

## 検出されたサービスのメトリクス調査

稼働中のPrometheusによって、`node`サービスで出力されたメトリクスを[expressionブラウザ](/docs/visualization/browser)を使って調査することが出来る。
例えば、[`up{job="node"}`](http://localhost:9090/graph?g0.range_input=1h&g0.expr=up%7Bjob%3D%22node%22%7D&g0.tab=1)というメトリックを調査すると、Node Exporterが適切に検出されていることが分かる。

## 監視対象リストの動的な変更

Prometheusのファイルベースのサービスディスカバリーを利用しているときには、Prometheusはファイルの変更をリッスンしており、再起動を必要とすることなく、監視対象リストを自動的に更新する。
この説明のために、二つ目のNode Exporterを9200ポートで起動する。
まず、Node Exporterのバイナリがあるディレクトリに移動し、このコマンドを新しいターミナルで実行する。

```bash
./node_exporter --web.listen-address=":9200"
```

以下のように新しいNode Exporterのエントリを追加することで`targets.json`の設定を変更する。

```json
[
  {
    "targets": [
      "localhost:9100"
    ],
    "labels": {
      "job": "node"
    }
  },
  {
    "targets": [
      "localhost:9200"
    ],
    "labels": {
      "job": "node"
    }
  }
]
```

変更を保存することで、自動的にPrometheusに新しい監視対象リストが知らされる。
メトリック[`up{job="node"}`](http://localhost:9090/graph?g0.range_input=1h&g0.expr=up%7Bjob%3D%22node%22%7D&g0.tab=1)は、ラベル`instance`が`localhost:9100`と`localhost:9200`の二つのインスタンスを表示するはずである。

## まとめ

このガイドでは、Node Exporterをインストール、実行し、ファイルベースのサービスディスカバリーでそのNode Exporterを検出しメトリクスを取得するように、Prometheusを設定した。
