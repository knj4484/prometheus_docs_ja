---
title: 用語集
sort_rank: 8
---

# **用語集**


### アラート（alert）

アラートは、Prometheusのアラートルールの結果で、firingなものである。 アラートは、PrometheusからAlertmanagerに送信される。

### Alertmanager

[Alertmanager](../../alerting/overview/)は、アラートを取り込み、 それらをグループに集約したり、重複を排除したり、サイレンスを適用したり、抑制したりした上で、eメールやPagerDuty、Slackなどに通知を送信する。

### ブリッジ（bridge）

ブリッジは、クライアントライブラリから値を取得し、Prometheus以外の監視システムに対してそれらを出力するコンポーネントである。 例えば、PythonやGo、JavaのクライアントはGraphiteにメトリクスを出力することができる。

### クライアントライブラリ（client library）

クライアントライブラリは、なんらかの言語（例えば、Go、Java、Python、Ruby）で、独自のコードに対して直接メトリクスを実装したり、他のシステムからメトリクスを取得する独自のコレクターを書いたり、Prometheusにメトリクスを出力することを容易にするためのライブラリである。

### コレクター（collector）

コレクターは、エクスポーター（exporter）の一部で、メトリクスの集合を表す。 コレクターは、直接メトリクスを実装したら1つのメトリクスのこともあるし、他のシステムからメトリクスを取得したらたくさんのメトリクスであることもある。

### 直接のメトリクス組み込み（direct instrumentation）

直接のメトリクス組み込みとは、あるプログラムのソースコードの一部としてインラインで追加されているメトリクスの組み込みのことである。

### エンドポイント（endpoint）

スクレイプされるメトリクスの出どころ。 通常は、1つのプロセスに対応する。

### エクスポーター（exporter）

exporterは、Prometheusのメトリクスを出力するバイナリである。Prometheus以外の形式で出力されているメトリクスを、Prometheusがサポートする形式に変換することがよくある。

### インスタンス（instance）

インスタンスは、ジョブ中でターゲットをユニークに識別するラベルである。

### ジョブ（job）

同じ目的（例えば、スケーラビリティや信頼性のために複製されたプロセスのグループの監視）の監視対象の集合をジョブと呼ぶ。

### 通知（notification）

通知は、1つ以上のアラートのグループを表し、AlertmanagerによってeメールやPagerDuty、Slackなどに送信される。

### Promdash

Promdashは、Prometheusのためにダッシュボードを構築する独自の仕組みだったが、非推奨となり、[Grafana](../../visualization/grafana/)に置き換えられた。

### Prometheus

Prometheusは、通常は、Prometheusシステムのコアのバイナリを指す。 また、Prometheus監視システム全体を指すこともある。

### PromQL

[PromQL](/docs/prometheus/latest/querying/basics/)とは、Prometheusクエリ言語（Prometheus Query Language）のことである。 PromQLによって、集約や予測、結合などを含む幅広い操作が可能となる。

### Pushgateway

[Pushgateway](../../instrumenting/pushing/)は、バッチジョブからpushされた最新のメトリクスを保持する。 これによって、バッチが終了した後でもメトリクスを取得することが可能になる。

### Remote Read

Remote readは、他のシステム（例えば、長期ストレージ）からクエリの一部として時系列を透過的に読み取ることを可能とするPrometheusの機能である。

### Remote Read Adapter

全てのシステムがremote readを直接サポートしている訳ではない。 remote read adapterは、Prometheusと他のシステムの間で時系列のリクエストとレスポンスを変換する。

### Remote Read Endpoint

remote read endpointは、Prometheusがremote readをする際のPrometheusの通信先のことである。

### Remote Write

remote writeは、Prometheusが取得した値を即時に他のシステム（例えば長期ストレージ）に送信する機能である。

### Remote Write Adapter

全てのシステムがremote writeを直接サポートしている訳ではない。 remote write adapterは、Prometheusと他のシステムの間で、remote writeの値を他のシステムが理解可能な形式に変換する。

### Remote Write Endpoint

remote write endpointは、Prometheusがremote writeをする際のPrometheusの通信先のことである。

### サンプル（sample）

サンプルとは、時系列のある時点での1つの値である。

Prometheusでは、各サンプルは、1つのfloat64の値と1つのミリ秒精度のタイムスタンプから成る。

### サイレンス（silence）

Alertmanagerのサイレンスは、設定にマッチしたラベルを持つアラートが通知に含まれることを防ぐ。

### ターゲット（target）

ターゲットは、スクレイプする対象の定義である。 例えば、どんなラベルを適用するか、接続に必要な認証、その他どのようにスクレイプするかを定義する情報である。

