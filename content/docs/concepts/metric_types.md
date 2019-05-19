---
title: メトリクスの型
sort_rank: 2
---

# メトリクスの型

Prometheusクライアントライブラリは、4つのメトリック型を提供している。 これらは、現在は、クライアントライブラリの中で（それらの特定の型の利用方法に応じたAPIを可能にするため）、および通信プロトコルの中でのみ区別されている。 Prometheusサーバーは、型情報をまだ利用しておらず、全てのデータを型なしの時系列データに押しつぶしている。 これは、将来的には変更されるかもしれない。

## カウンター

_カウンター_ は、累積的なメトリクスであり、単一の[単調増加するカウンター](https://en.wikipedia.org/wiki/単調写像)であり、その値は増加させるか起動時にゼロにリセットすることしかできない。 例えば、カウンターは、応答したリクエスト数、完了したタスクの数、エラーの数などに利用できる。

減少することがありうる値を出力するためにカウンターを使わないこと。 例えば、実行中のプロセス数にはカウンターを使わず、ゲージを使うこと。

クライアントライブラリのカウンターの利用方法ドキュメントは以下の通り。

   * [Go](http://godoc.org/github.com/prometheus/client_golang/prometheus#Counter)
   * [Java](https://github.com/prometheus/client_java/blob/master/simpleclient/src/main/java/io/prometheus/client/Counter.java)
   * [Python](https://github.com/prometheus/client_python#counter)
   * [Ruby](https://github.com/prometheus/client_ruby#counter)

## ゲージ

_ゲージ_ は、自由に増加したり減少しうる単一の数値を表すメトリックである。

_ゲージ`_ は、典型的には温度やメモリ使用量のような計測値に利用されるが、同時リクエスト数のような増えたり減ったりするカウントにも利用される。

クライアントライブラリのゲージの利用方法ドキュメントは以下の通り。

   * [Go](http://godoc.org/github.com/prometheus/client_golang/prometheus#Gauge)
   * [Java](https://github.com/prometheus/client_java/blob/master/simpleclient/src/main/java/io/prometheus/client/Gauge.java)
   * [Python](https://github.com/prometheus/client_python#gauge)
   * [Ruby](https://github.com/prometheus/client_ruby#gauge)

## ヒストグラム

_ヒストグラム_ は、観測値（通常はリクエスト持続時間やレスポンスサイズのようなもの）のサンプリングをし、設定可能なバケット毎にカウントする。 全ての観測値の合計も提供する。

基本となるメトリック名が`<basename>`のヒストグラムは、以下のような複数の時系列を出力する。

  * 観測値のバケット毎の累積カウントを`<basename>_bucket{le="<upper inclusive bound>"}`として出力する
  * 全ての観測値の合計を`<basename>_sum`として出力する
  * 観測されたイベント数を`<basename>_count`として出力する（上記の`<basename>_bucket{le="+Inf"}`と同じ）

ヒストグラムや集約されたヒストグラムから分位数(quantile)を計算するには、[`histogram_quantile()`](/docs/prometheus/latest/querying/functions/#histogram_quantile)を利用する。 ヒストグラムは、[Apdex score](http://en.wikipedia.org/wiki/Apdex)を計算するのにも適している。 バケットを処理する際には、ヒストグラムが[累積的](https://ja.wikipedia.org/wiki/ヒストグラム#累積度数図)であることを忘れずに。 ヒストグラムの利用方法およびサマリーとの違いの詳細に関しては、ヒストグラムとサマリーを参照すること。

クライアントライブラリのヒストグラムの利用方法ドキュメントは以下の通り。

   * [Go](http://godoc.org/github.com/prometheus/client_golang/prometheus#Histogram)
   * [Java](https://github.com/prometheus/client_java/blob/master/simpleclient/src/main/java/io/prometheus/client/Histogram.java)
   * [Python](https://github.com/prometheus/client_python#histogram)
   * [Ruby](https://github.com/prometheus/client_ruby#histogram)

## サマリー

_サマリー_ は、_ヒストグラム_ と同様に、観測値（通常はリクエスト持続時間やレスポンスサイズのようなもの）のサンプリングをする。観測の全カウントおよび観測値の合計を提供すると同時に、設定可能な分位数を計算する。

基本となるメトリック名が`<basename>`のサマリーは、以下のような複数の時系列を出力する。

  * 観測されたイベントの **φ分位数** (0 ≤ φ ≤ 1)を`<basename>{quantile="<φ>"}`として出力する
  * 全ての観測値の **合計** を`<basename>_sum`として出力する
  * 観測されたイベントの **数** を`<basename>_count`として出力する

φ分位数やサマリーの利用方法、[ヒストグラム](#histogram)との違いの詳細な説明については、[ヒストグラムとサマリー](/docs/practices/histograms)を参照すること。

クライアントライブラリのサマリーの利用方法ドキュメントは以下の通り。

   * [Go](http://godoc.org/github.com/prometheus/client_golang/prometheus#Summary)
   * [Java](https://github.com/prometheus/client_java/blob/master/simpleclient/src/main/java/io/prometheus/client/Summary.java)
   * [Python](https://github.com/prometheus/client_python#summary)
   * [Ruby](https://github.com/prometheus/client_ruby#summary)
