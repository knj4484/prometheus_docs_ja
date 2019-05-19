---
title: 代替システムとの比較
sort_rank: 4
---

# 代替システムとの比較

## Prometheus vs. Graphite

### スコープ

[Graphite](http://graphite.readthedocs.org/en/latest/)は、クエリ言語とグラフ化機能を持った受動的な時系列データーベースであることに焦点を当てている。
その他の事は、外部コンポーネントで解決される。

Prometheusは、完全な監視・トレンド分析システムであり、組み込みで能動的なスクレイピング、保存、クエリ、グラフ化、時系列に基づいたアラートを含んでいる。
Prometheusは、世界がどうあるべきか（どのエンドポイントが存在すべきか、時系列のどんなパターンが問題を表しているかなど）について分かっていて能動的に問題を発見しようとする。

### データモデル

Graphiteは、Prometheusと同じように、名前の付いた時系列の数値的なサンプルを保存する。
しかし、Prometheusのメタデータのモデルの方が、表現力が高い。
Graphiteのメトリック名がドット区切りの要素で軸を暗黙的に表現しているのに対して、
Prometheusは、メトリック名に付随するラベルと呼ばれるキーバリューとして軸を明示的に表現する。
これによって、クエリ言語でラベルによってフィルタリング、グルーピング、マッチングが簡単にできるようになる。

さらに、Graphiteが[StatsD](https://github.com/etsy/statsd/)と組み合わせて利用されると、
インスタンスを要素として保存して問題のある個別のインスタンスを深掘りできるようにするのではなく、
監視されているインスタンス全てについて集約されたデータだけを保存するのが普通である。

例えば、レスポンスコードが`500`でメソッドが`POST`でエンドポイント`/tracks`へのAPIサーバーへのリクエスト数を保存するには、Graphite/StatsDでは、普通、以下のようにエンコードされるだろう。

```
stats.api-server.tracks.post.500 -> 93
```

Prometheusでは同じデータは以下のようにエンコードされるだろう（3台のapiサーバーインスタンスを仮定）。

```
api_server_http_requests_total{method="POST",handler="/tracks",status="500",instance="<sample1>"} -> 34
api_server_http_requests_total{method="POST",handler="/tracks",status="500",instance="<sample2>"} -> 28
api_server_http_requests_total{method="POST",handler="/tracks",status="500",instance="<sample3>"} -> 31
```

### ストレージ

Graphiteは、時系列データをローカルディスクに[Whisper](http://graphite.readthedocs.org/en/latest/whisper.html)形式で保存する。
Whisperは、RRDスタイルのデータベースで、サンプルが規則正しい間隔でやってくることを想定している。
各時系列は、別々のファイルに保存され、一定時間後に新しいサンプルが古いサンプルを上書きする。

Prometheusも時系列ごとに1つのローカルファイルを作成するが、スクレイプやルール評価が起きる任意の間隔でサンプルを保存することができる。
新しい値は単純に追加されるので、古いデータは任意の期間保持することが可能である。
Prometheusは短命の頻繁に更新される時系列の集合に対してもうまく機能する。

### まとめ

Prometheusは、既存の環境で実行したり連携するのが簡単であるだけでなく、表現力の高いデータモデルとクエリ言語を提供する。
長期間の過去データを保持できるクラスタ化されたソリューションが必要なら、Graphiteの方が良い選択であろう。

## Prometheus vs. InfluxDB

[InfluxDB](https://influxdata.com/)は、オープンソースの時系列データベースであり、商用のスケーリングとクラスタリングのオプションがある。
InfluxDBプロジェクトはPrometheusの開発開始のほぼ1年後にリリースされたので、我々は当時はそれが代替システムであるとXX
今でも、PrometheusとInfluxDBには大きな違いがあり、両システムは少し違ったユースケースに合わせている。

### スコープ

比較が公平になるように、InfluxDBと共に[Kapacitor](https://github.com/influxdata/kapacitor)について考えなければならない。
なぜなら、それらは組み合わせて、PrometheusとAlertmanagerと同じ領域の問題を解決するからである。

[Graphite](#prometheus-vs-graphite)の場合と同じスコープの違いが、ここでもInfluxDB自体に当てはまる。
さらに、InfluxDBには、Prometheusのレコーディングルールに相当するContinuous Queryがある。

Kapacitorのスコープは、Prometheusのレコーディングルールとアラートルール、Alertmanagerの通知機能の組み合わせである。
Prometheusには、[グラフ化とアラートのためのもっと強力なクエリ言語](https://www.robustperception.io/translating-between-monitoring-languages/)がある。
Alertmanagerは、さらに、グルーピング、重複排除、サイレンスの機能がある。

### データモデル / ストレージ

Prometheusと同様に、InfluxDBのデータモデルはラベルとしてキーバリューペアがあり、タグと呼ばれている。
さらに、InfluxDBは、フィールドと呼ばれる2番目のレベルのラベルをサポートする。
フィールドは、使用にあたってはかなり制限がある。
InfluxDBは、ナノ秒までの解像度のタイムスタンプ、float64/int64/bool/stringのデータ型をサポートする。
それに対して、Prometheusは、限られたサポートの文字列とfloat64のデータ型、ミリ秒の解像度のタイムスタンプをサポートしている。


InfluxDBは、時間でシャードされた[log-structured merge tree for storage with a write ahead log](https://docs.influxdata.com/influxdb/v1.2/concepts/storage_engine/)の変種を使っている。
これは、Prometheusの時系列ごとの追記のみのファイルを使う方法よりもイベントロギングに適している。

[Logs and Metrics and Graphs, Oh My!](https://blog.raintank.io/logs-and-metrics-and-graphs-oh-my/)がイベントロギングとメトリクスの記録の違いに付いて説明している。

### アーキテクチャー

Prometheusサーバーはお互いに独立して稼働し、中核となる機能（スクレイピング、ルールの処理、アラート）にはローカルのストレージだけに依存している。
オープンソース版のInfluxDBも同様である。

商用InfluxDBが提供しているものは、仕様として、ストレージとクエリがたくさんのノードで同時に処理される分散ストレージクラスタである。

商用InfluxDBは、水平にスケールするのが容易ということであるが、分散ストレージシステムの複雑さを最初からなんとかしなければならないということでもある。
Prometheusは稼働させるのが簡単だが、いつかはサーバーを、明示的にスケーラビリティの境界（プロダクト、サービス、データセンターまたは剃られに類する側面）に沿ってシャードしなければならなくなるだろう。
独立したサーバー（並行して冗長に実行できるもの）は、より良い信頼性とエラーの独立性をもたらすだろう。

Kapacitorのオープンソースのリリースは、アラートや通知に対する組み込みの分散された/冗長性のある選択肢はない。
Kapacitorのオープンソースのリリースは、Prometheus自体と同様に、ユーザーによる手動のシャーディングでスケールすることができる。
Influxは、可用性が高く冗長性のあるアラートシステム、[Enterprise Kapacitor](https://docs.influxdata.com/enterprise_kapacitor)を提供する。

PrometheusとAlertmanagerは、対照的に、完全にオープンソースの冗長な選択肢がある。
Prometheusの冗長なレプリカ稼働させて、Alertmanagerの[高可用](https://github.com/prometheus/alertmanager#high-availability)モードを利用することで冗長にする。

### まとめ

この2つのシステムには多くの類似性がある。
どちらにも、多次元メトリクスを効率的にサポートするためのラベルがある（InfluxDBではタグと呼ばれる）。
どちらも基本的には同じ圧縮アルゴリズムを用いる。
どちらも、お互いを含む、幅広いインテグレーションがある。
どちらも、インテグレーションを他のもの（統計ツールでのデータ分析や自動化された処理の実行など）にも広げられるようにフックがある。

InfluxDBの方が良い点は以下の通り。

  * イベントロギングをする場合
  * 商用版はInfluxDBのためのクラスタリングを提供しており、長期のデータ保存にも良い
  * 最終的に一貫性のあるレプリカ間のデータの見え方

Prometheusの方が良い点は以下の通り。

  * メトリクスを第一に利用する場合
  * より強力なクエリ言語、アラーティング、通知機能
  * グラフ化とアラートのための高い可用性と稼働時間

InfluxDBは、オープンコアモデルに従った1つの営利企業によって保守されており、クローズドソースのクラスタリング、ホスティング、サポートなどのプレミアム機能を提供している。
Prometheusは、[完全にオープンソースで独立したプロジェクト](/community/)であり、多くの企業と個人によって保守されている。
そのうちのいくつかの企業は商用のサービスとサポートを提供している。

## Prometheus vs. OpenTSDB

[OpenTSDB](http://opentsdb.net/)は、[Hadoop](http://hadoop.apache.org/)と[HBase](http://hbase.apache.org/)に基づいた分散時系列データベースである。

### スコープ

[Graphite](/docs/introduction/comparison/#prometheus-vs-graphite)の場合と同様のスコープの差がここでも当てはまる。

### データモデル

OpenTSDBのデータモデルは、Prometheusとほぼ一致する。
時系列は、任意のキーバリューの組み合わせ（OpenTSDBのタグがPrometheusのラベルに相当する）で特定される。
1つのメトリックのための全てのデータは、メトリクスのタグの種類を制限しつつ、[一緒に保存](http://opentsdb.net/docs/build/html/user_guide/writing/index.html#time-series-cardinality)される。
ただし、小さな違いはある。
Prometheusはラベル値に任意の文字を許すが、OpenTSDBは制限が強い。
OpenTSDBは、完全なクエリ言語がなく、APIを通じた簡単な集約と計算ができるだけである。

### ストレージ

[OpenTSDB](http://opentsdb.net/)のストレージは、[Hadoop](http://hadoop.apache.org/)と[HBase](http://hbase.apache.org/)の上で実装されている。
これは、OpenTSDBを水平にスケールさせるのが容易であるが、最初からHadoop/HBaseクラスタを運用する全体的な複雑性を受け入れなければならないことを意味している。

Prometheusは、初期の運用は単純だが、1つのノードの容量を越えると明示的にシャーディングする必要がある。

### まとめ

Prometheusの方が、かなり表現力の高いクエリ言語を提供し、ラベル値の種類の多いメトリクスを扱うことができ、完全な監視システムの一部となる。
もし既にHadoopを運用しており、これらの利益より長期ストレージに価値があるなら、OpenTSDBは良い選択である。

## Prometheus vs. Nagios

[Nagios](https://www.nagios.org/)は、90年代にNetSaintとして始まった監視システムである。

### スコープ

Nagios is primarily about alerting based on the exit codes of scripts. These are 
called “checks”. There is silencing of individual alerts, however no grouping, 
routing or deduplication.
Nagiosは、本来、スクリプトの終了コードを元にしたアラートをXXX
これらは「チェック」と呼ばれる。
個々のアラートを黙らせることはできるが、グループ化やルーティング、重複排除はできない。

There are a variety of plugins. For example, piping the few kilobytes of
perfData plugins are allowed to return [to a time series database such as Graphite](https://github.com/shawn-sterling/graphios) or using NRPE to [run checks on remote machines](https://exchange.nagios.org/directory/Addons/Monitoring-Agents/NRPE--2D-Nagios-Remote-Plugin-Executor/details).
様々なプラグインがある。
例えば、
perfDataプラグイン
は、[Graphiteなどの時系列データーベースに送る](https://github.com/shawn-sterling/graphios)ことができる。
[リモートのマシン上でチェックを実行する](https://exchange.nagios.org/directory/Addons/Monitoring-Agents/NRPE--2D-Nagios-Remote-Plugin-Executor/details)ためにNRPEを使う

### データモデル

Nagiosはホストベースである。
各ホストは複数のサービスを持つことができ、各サービスは1つのチェックを実行する。

ラベルやクエリ言語といった概念はない。

### ストレージ

Nagios自体には、現在のチェックの状態を超えたストレージはない。
[可視化のため](https://docs.pnp4nagios.org/)などにデータを保存できるプラグインはある。

### アーキテクチャー

Nagiosサーバーはスタンドアローンである。
チェックの設定は全てファイルで行う。

### まとめ

Nagiosは、ブラックボックスの検査で十分な小さいシステムや静的なシステムの基本的な監視に適している。

ホワイトボックスの監視がしたかったり、動的またはクラウドベースの環境であるなら、Prometheusが良い選択である。

## Prometheus vs. Sensu

[Sensu](https://sensu.io) is a composable monitoring pipeline that can reuse existing Nagios checks.
[Sensu](https://sensu.io)は、既存のNagiosのチェックを再利用できる。

### スコープ

Nagiosの場合と同様の一般的なスコープの差がここでも当てはまる。

アドホックなチェックの結果をSensuにプッシュできるようにする[クライアントソケット](https://docs.sensu.io/sensu-core/latest/reference/clients/#what-is-the-sensu-client-socket)もある。

### データモデル

[Nagios](/docs/introduction/comparison/#prometheus-vs-nagios)と同じ大雑把なデータモデルを持つ。

### ストレージ

Sensuは、Sensuクライアントのレジストリ、チェック結果、チェック実行履歴、現在のイベントのデータを含む監視データを永続化するためにRedisを利用する。

### アーキテクチャー

Sensuには、[多くのコンポーネント](https://docs.sensu.io/sensu-core/latest/overview/architecture/)がある。
メッセージの送受信にRabbitMQ、現在の状態についてはRedis、処理とAPIアクセスのために別のサーバーを使う。

Sensuデプロイの全てのコンポーネント（RabbitMQ、Redis、Sensu Server/API）は、可用性が高く冗長な構成にするためにクラスタ化できる。

### まとめ

そのままスケールさせたい既存のNagiosが構築されていたり、Sensuの自動登録機能を使いたい場合は、Sensuが良い選択である。

ホワイトボックスの監視がしたかったり、動的またはクラウドベースの環境であるなら、Prometheusが良い選択である。
