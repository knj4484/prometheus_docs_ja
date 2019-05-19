<!-- # statsd exporter [![Build Status](https://travis-ci.org/prometheus/statsd_exporter.svg)][travis] -->
# statsd exporterの使い方
<!-- 
[![CircleCI](https://circleci.com/gh/prometheus/statsd_exporter/tree/master.svg?style=shield)][circleci]
[![Docker Repository on Quay](https://quay.io/repository/prometheus/statsd-exporter/status)][quay]
[![Docker Pulls](https://img.shields.io/docker/pulls/prom/statsd-exporter.svg)][hub]
 -->
<!-- `statsd_exporter` receives StatsD-style metrics and exports them as Prometheus metrics. -->
`statsd_exporter`はStatsD形式のメトリクスを受け取ってPrometheusメトリクスを出力する。

<!-- ## Overview -->
## 概要

### With StatsD

<!-- To pipe metrics from an existing StatsD environment into Prometheus, configure
StatsD's repeater backend to repeat all received metrics to a `statsd_exporter`
process. This exporter translates StatsD metrics to Prometheus metrics via
configured mapping rules. -->
既存のStatsD環境からPrometheusにメトリクスを送信するには、StatsDリピーターバックエンドが受信したメトリクスを全て`statsd_exporter`プロセスに再送信するように設定する。
このexporterは、マッピングルールの設定に従って、StatsDメトリクスをPrometheusメトリクスに変換する。

    +----------+                         +-------------------+                        +--------------+
    |  StatsD  |---(UDP/TCP repeater)--->|  statsd_exporter  |<---(scrape /metrics)---|  Prometheus  |
    +----------+                         +-------------------+                        +--------------+

### Without StatsD

<!-- Since the StatsD exporter uses the same line protocol as StatsD itself, you can
also configure your applications to send StatsD metrics directly to the exporter.
In that case, you don't need to run a StatsD server anymore. -->
StatsD exporterは、StatsD事態と同じ行プロトコルを利用するので、アプリケーションがStatsDメトリクスをexporterに直接送信するように設定することもできる。
そのような場合には、StatsDサーバーを運用する必要がなくなる。

<!-- We recommend this only as an intermediate solution and recommend switching to
[native Prometheus instrumentation](http://prometheus.io/docs/instrumenting/clientlibs/)
in the long term. -->
この方法は中間的な解決策としてだけ推奨され、長期的には[Prometheusメトリクスを直接実装する](http://prometheus.io/docs/instrumenting/clientlibs/)ように切り替えることを推奨する。

### DogStatsD extensions

<!-- The exporter will convert DogStatsD-style tags to prometheus labels. See
[Tags](http://docs.datadoghq.com/guides/dogstatsd/#tags) in the DogStatsD
documentation for the concept description and
[Datagram Format](http://docs.datadoghq.com/guides/dogstatsd/#datagram-format)
for specifics. It boils down to appending
`|#tag:value,another_tag:another_value` to the normal StatsD format.  Tags
without values (`#some_tag`) are not supported. -->
このexporterは、DogStatsD形式のタグをPrometheusのラベルに変換する。
この概念の説明についてはDogStatsDドキュメントの[Tags](http://docs.datadoghq.com/guides/dogstatsd/#tags)を、
仕様については[Datagram Format](http://docs.datadoghq.com/guides/dogstatsd/#datagram-format)を参照すること。
要約すると、通常のStatsDフォーマットに`|#tag:value,another_tag:another_value`を追加する。
値のないタグ`#some_tag`は、サポートされていない。

<!-- ## Building and Running -->
## ビルドと実行

<!-- NOTE: Version 0.7.0 switched to the [kingpin](https://github.com/alecthomas/kingpin) flags library. With this change, flag behaviour is POSIX-ish: -->
バージョン0.7.0で、[kingpin](https://github.com/alecthomas/kingpin)フラグライブラリを使うようになった。
この変更でフラグの挙動は、以下のようにPOSIX的になった。
<!-- 
* long flags start with two dashes (`--version`)
* multiple short flags can be combined (but there currently is only one)
* flag processing stops at the first `--`
 -->
* ロングフラグは2つのダッシュで始まる（`--version`）
* 複数のショートフラグはまとめられる（ただし現状では1つしかない）
* フラグ処理は最初の`--`で終了する

    ```
    $ go build
    $ ./statsd_exporter --help
    usage: statsd_exporter [<flags>]

    Flags:
      -h, --help               Show context-sensitive help (also try --help-long and --help-man).
          --web.listen-address=":9102"
                               The address on which to expose the web interface and generated Prometheus metrics.
          --web.telemetry-path="/metrics"
                               Path under which to expose metrics.
          --statsd.listen-udp=":9125"
                               The UDP address on which to receive statsd metric lines. "" disables it.
          --statsd.listen-tcp=":9125"
                               The TCP address on which to receive statsd metric lines. "" disables it.
          --statsd.listen-unixgram=""
                               The Unixgram socket path to receive statsd metric lines in datagram. "" disables it.
          --statsd.unixsocket-mode="755"
                               The permission mode of the unix socket.
          --statsd.mapping-config=STATSD.MAPPING-CONFIG
                               Metric mapping configuration file name.
          --statsd.read-buffer=STATSD.READ-BUFFER
                               Size (in bytes) of the operating system's transmit read buffer associated with the UDP or Unixgram connection. Please make sure the kernel parameters net.core.rmem_max is set to a value greater than the value specified.
          --debug.dump-fsm=""  The path to dump internal FSM generated for glob matching as Dot file.
          --log.level="info"   Only log messages with the given severity or above. Valid levels: [debug, info, warn, error, fatal]
          --log.format="logger:stderr"
                               Set the log target and format. Example: "logger:syslog?appname=bob&local=7" or "logger:stdout?json=true"
          --version            Show application version.
    ```

## Tests

    $ go test

<!-- ## Metric Mapping and Configuration -->
## メトリクスのマッピングと設定

<!-- The `statsd_exporter` can be configured to translate specific dot-separated StatsD
metrics into labeled Prometheus metrics via a simple mapping language. The config
file is watched for changes and automatically reloaded. -->
`statsd_exporter`は、簡単なマッピング言語を用いて、ドット区切りのStatsDメトリクスをPrometheusのラベル付きのメトリクスに変換するように設定できる。
設定ファイルの変更は監視されていて、自動的に再読み込みされる。

<!-- A mapping definition starts with a line matching the StatsD metric in question,
with `*`s acting as wildcards for each dot-separated metric component. The
lines following the matching expression must contain one `label="value"` pair
each, and at least define the metric name (label name `name`). The Prometheus
metric is then constructed from these labels. `$n`-style references in the
label value are replaced by the n-th wildcard match in the matching line,
starting at 1. Multiple matching definitions are separated by one or more empty
lines. The first mapping rule that matches a StatsD metric wins. -->
マッピングの定義は、対象とするStatsDメトリックにマッチングする行で始まる。
`*`は、ドット区切りのメトリックの要素に対するワイルドカードのように作用する。
マッチング式に続く行はそれぞれ、`label="value"`の組みを1つ含まなければならない。
また、少なくともメトリック名（ラベル名`name`）を定義しなければならない。
こうしてこれらのラベルからPrometheusメトリクスが構成される。
ラベル値の`$n`形式の参照は、マッチング行のn番目（1始まり）のワイルドカードのマッチで置き換えられる。
複数のマッチング定義は、1つ以上の空行で分けられる。
マッチした最初のマッピングルールが採用される。

<!-- Metrics that don't match any mapping in the configuration file are translated
into Prometheus metrics without any labels and with any non-alphanumeric
characters, including periods, translated into underscores. -->
設定ファイルのどのマッピングにもマッチしないメトリクスは、ラベルなしで、英数字出ない文字を除いたPrometheusメトリクスに変換される。ピリオドはアンダースコアに変換される。

<!-- In general, the different metric types are translated as follows: -->
一般的に、各種メトリック型は、以下のように変換される。

    StatsD gauge   -> Prometheus gauge

    StatsD counter -> Prometheus counter

    StatsD timer   -> Prometheus summary                    <-- indicates timer quantiles
                   -> Prometheus counter (suffix `_total`)  <-- indicates total time spent
                   -> Prometheus counter (suffix `_count`)  <-- indicates total number of timer events

<!-- An example mapping configuration: -->
マッピングの設定のサンプルを以下に示す。

```yaml
mappings:
- match: test.dispatcher.*.*.*
  name: "dispatcher_events_total"
  labels:
    processor: "$1"
    action: "$2"
    outcome: "$3"
    job: "test_dispatcher"
- match: *.signup.*.*
  name: "signup_events_total"
  labels:
    provider: "$2"
    outcome: "$3"
    job: "${1}_server"
```

<!-- This would transform these example StatsD metrics into Prometheus metrics as
follows: -->
この設定は、以下のように、StatsDメトリクスをPrometheusメトリクスに変換する。

    test.dispatcher.FooProcessor.send.success
     => dispatcher_events_total{processor="FooProcessor", action="send", outcome="success", job="test_dispatcher"}

    foo_product.signup.facebook.failure
     => signup_events_total{provider="facebook", outcome="failure", job="foo_product_server"}

    test.web-server.foo.bar
     => test_web_server_foo_bar{}

<!-- Each mapping in the configuration file must define a `name` for the metric. The
metric's name can contain `$n`-style references to be replaced by the n-th
wildcard match in the matching line. That allows for dynamic rewrites, such as: -->
設定ファイルの各マッピングは、メトリックに対して`name`を1つ定義しなければいけない。
メトリック名は、マッチングの行のn番目のワイルドカードにマッチした部分を`$n`形式の参照を含むことができる。
それによって、以下のような動的な書き換えが可能になる。

```yaml
mappings:
- match: test.*.*.counter
  name: "${2}_total"
  labels:
    provider: "$1"
```

<!-- The metric name can also contain references to regex matches. The mapping above
could be written as: -->
メトリック名は、正規表現マッチへの参照を含むこともできる。
上記のマッピングは、以下のように書くこともできる。

```
mappings:
- match: test\.(\w+)\.(\w+)\.counter
  match_type: regex
  name: "${2}_total"
  labels:
    provider: "$1"
```

<!-- Please note that metrics with the same name must also have the same set of
label names. -->
同じ名前のメトリクスは同じラベル名の集合を持たなければならないことに注意すること。

<!-- If the default metric help text is insufficient for your needs you may use the YAML
configuration to specify a custom help text for each mapping: -->
デフォルトのヘルプテキストが自分の要件に不十分な場合は、各マッピングに対して独自のヘルプテキストを指定するためのYAML設定を利用することができる。

```yaml
mappings:
- match: http.request.*
  help: "Total number of http requests"
  name: "http_requests_total"
  labels:
    code: "$1"
```

### StatsD timers

<!-- By default, statsd timers are represented as a Prometheus summary with
quantiles. You may optionally configure the [quantiles and acceptable
error](https://prometheus.io/docs/practices/histograms/#quantiles): -->
statsd timerは、デフォルトで、Prometheusのサマリーとして表される。
[分位数と許容誤差](https://prometheus.io/docs/practices/histograms/#quantiles)を設定することも出来る。

```yaml
mappings:
- match: test.timing.*.*.*
  timer_type: summary
  name: "my_timer"
  labels:
    provider: "$2"
    outcome: "$3"
    job: "${1}_server"
  quantiles:
    - quantile: 0.99
      error: 0.001
    - quantile: 0.95
      error: 0.01
    - quantile: 0.9
      error: 0.05
    - quantile: 0.5
      error: 0.005
```

<!-- The default quantiles are 0.99, 0.9, and 0.5. -->
デフォルトの分位数は、0.99、0.9、0.5である。

<!-- In the configuration, one may also set the timer type to "histogram". The
default is "summary" as in the plain text configuration format.  For example,
to set the timer type for a single metric: -->
この設定では、timerをヒストグラムにすることもできる。
The default is "summary" as in the plain text configuration format.
例えば、timerを1つのメトリックにするには、以下のようにする。

```yaml
mappings:
- match: test.timing.*.*.*
  timer_type: histogram
  buckets: [ 0.01, 0.025, 0.05, 0.1 ]
  name: "my_timer"
  labels:
    provider: "$2"
    outcome: "$3"
    job: "${1}_server"
```

### Regular expression matching

<!-- Another capability when using YAML configuration is the ability to define matches
using raw regular expressions as opposed to the default globbing style of match.
This may allow for pulling structured data from otherwise poorly named statsd
metrics AND allow for more precise targetting of match rules. When no `match_type`
paramter is specified the default value of `glob` will be assumed: -->
YAML設定を利用する際のもう一つの機能は、デフォルトのグロブ形式のマッチングの代わりに、正規表現を利用したマッチングの定義である。
これによって、名前の付け方の悪いStatsDメトリクスから構造化されたデータを引き出すことができ、より細やかなマッチングルールのターゲティングが可能になる。
パラメーター`match_type`が指定されていない場合、デフォルト値`glob`だと仮定される。

```yaml
mappings:
- match: (.*)\.(.*)--(.*)\.status\.(.*)\.count
  match_type: regex
  name: "request_total"
  labels:
    hostname: "$1"
    exec: "$2"
    protocol: "$3"
    code: "$4"
```

<!-- Note, that one may also set the histogram buckets.  If not set, then the default
[Prometheus client values](https://godoc.org/github.com/prometheus/client_golang/prometheus#pkg-variables) are used: `[.005, .01, .025, .05, .1, .25, .5, 1, 2.5, 5, 10]`. `+Inf` is added
automatically. -->
ヒストグラムのバケットも指定することが可能である。
指定されていない場合は、デフォルトの[Prometheusクライアントの値](https://godoc.org/github.com/prometheus/client_golang/prometheus#pkg-variables)`[.005, .01, .025, .05, .1, .25, .5, 1, 2.5, 5, 10]`が利用される。
`+Inf`は自動的に追加される。

<!-- `timer_type` is only used when the statsd metric type is a timer. `buckets` is
only used when the statsd metric type is a timerand the `timer_type` is set to
"histogram." -->
`timer_type`は、StatsDメトリック型がtimerの時にだけ利用される。
`buckets`は、StatsDメトリック型がtimerで`timer_type`が"histogram"に指定された時にだけ利用される。

### Global defaults

<!-- One may also set defaults for the timer type, buckets or quantiles, and match_type. These will be used
by all mappings that do not define these. -->
timer type、buckets、quantiles、match_typeに対してデフォルトを指定することもできる。
それらのデフォルト値は、その値が定義されなかったマッピング全てにおいて利用される。

<!-- An option that can only be configured in `defaults` is `glob_disable_ordering`, which is `false` if omitted. By setting this to `true`, `glob` match type will not honor the occurance of rules in the mapping rules file and always treat `*` as lower priority than a general string. -->
`defaults`内でのみ設定出来る項目は、`glob_disable_ordering`である。
これは、省略されると`false`となる。
`true`に設定すると、マッピングルールファイル内でのルールの出現順を守らなくなり、`*`を通常の文字列よりも常に低い優先順で扱うようになる。

```yaml
defaults:
  timer_type: histogram
  buckets: [.005, .01, .025, .05, .1, .25, .5, 1, 2.5 ]
  match_type: glob
  glob_disable_ordering: false
  ttl: 0 # metrics do not expire
mappings:
# This will be a histogram using the buckets set in `defaults`.
- match: test.timing.*.*.*
  name: "my_timer"
  labels:
    provider: "$2"
    outcome: "$3"
    job: "${1}_server"
# This will be a summary timer.
- match: other.timing.*.*.*
  timer_type: summary
  name: "other_timer"
  labels:
    provider: "$2"
    outcome: "$3"
    job: "${1}_server_other"
```

<!-- ### Choosing between glob or regex match type -->
### match typeとしてglobかregexの選択

<!-- Despite from the missing flexibility of using regular expression in mapping and
formatting labels, `glob` matching is optimized to have better performance than
`regex` in certain use cases. In short, glob will have best performance if the
rules amount is not so less and captures (using of `*`) is not to much in a
single rule. Whether disabling ordering in glob or not won't have a noticable
effect on performance in general use cases. In edge cases like the below however,
disabling ordering will be beneficial: -->
`glob`マッチングは、マッピングとラベルのフォーマットで正規表現を利用するような柔軟性に欠けてはいるが、ある種のユースケースで`regex`よりもパフォーマンスがよい。
端的に言うと、ルールの量が少なくなく、1つのルールでのキャプチャー（`*`の使用）があまり多くないなら、グロブが最善のパフォーマンスになる。
一般的なユースケースでは、`glob_disable_ordering`を変更しても、注目に値するほどのパフォーマンスへの効果はない。
ただし、以下のようなエッジケースでは、`glob_disable_ordering`をtrueにすると効果がある。

    a.*.*.*.*
    a.b.*.*.*
    a.b.c.*.*
    a.b.c.d.*

<!-- The reason is that the list assignment of captures (using of `*`) is the most
expensive operation in glob. Honoring ordering will result in up to 10 list
assignments, while without ordering it will need only 4 at most. -->
理由は、キャプチャー（`*`の使用）のリストへの割り当て操作がグロブの中で最もコストが高いからである。
出現順を守ると、10回ものリストへの割り当てをすることになるが、出現順を守らないなら、多くても4回だけになる。

<!-- For details, see [pkg/mapper/fsm/README.md](pkg/mapper/fsm/README.md).
Running `go test -bench .` in **pkg/mapper** directory will produce
a detailed comparison between the two match type. -->
詳細は、[pkg/mapper/fsm/README.md](https://github.com/prometheus/statsd_exporter/blob/master/pkg/mapper/fsm/README.md)を参照すること。
ディレクトリ **pkg/mapper** で`go test -bench .`を実行すると、この2つのマッチタイプの詳細な比較が生成される。

### `drop` action

<!-- You may also drop metrics by specifying a "drop" action on a match. For
example: -->
マッチに対してdropアクションを指定することで、メトリクスを削除することもできる。

```yaml
mappings:
# This metric would match as normal.
- match: test.timing.*.*.*
  name: "my_timer"
  labels:
    provider: "$2"
    outcome: "$3"
    job: "${1}_server"
# Any metric not matched will be dropped because "." matches all metrics.
- match: .
  match_type: regex
  action: drop
  name: "dropped"
```

<!-- You can drop any metric using the normal match syntax.
The default action is "map" which does the normal metrics mapping. -->
通常のmatchの構文を利用して、どんなメトリックでも削除することができる。
デフォルトのactionはmapであり、通常のメトリクスのマッピングを行う。

<!-- ### Explicit metric type mapping -->
### 明示的なメトリック型のマッピング

<!-- StatsD allows emitting of different metric types under the same metric name,
but the Prometheus client library can't merge those. For this use-case the
mapping definition allows you to specify which metric type to match:
 -->StatsDは、同じメトリック名で異なるメトリック型を送信することができるが、Prometheusクライアントライブラリは、それらをまとめることはできない。
このようなユースケースでは、どちらのメトリック型にマッチするかをマッピング定義で指定することができる。

```
mappings:
- match: test.foo.*
  name: "test_foo"
  match_metric_type: counter
  labels:
    provider: "$1"
```

<!-- Possible values for `match_metric_type` are `gauge`, `counter` and `timer`. -->
`match_metric_type`で利用可能な値は、`gauge`、`counter`、`timer`である。

<!-- ### Time series expiration -->
### 時系列の有効期限

<!-- The `ttl` parameter can be used to define the expiration time for stale metrics.
The value is a time duration with valid time units: "ns", "us" (or "µs"),
. For example, `ttl: 1m20s`. `0` value is used to indicate
metrics that do not expire. -->
パラメーター`ttl`は、古くなったメトリクスが無効になるまでの時間を定義するために利用できる。
値は、時間間隔と有効な単位（"ns"、"us"（つまり"µs"）"ms"、"s"、"m"、"h"）である。
例えば、`ttl: 1m20s`とする。値に`0`を使うことで、時間切れしないメトリクスを表す。

<!--  TTL configuration is stored for each mapped metric name/labels combination
 whenever new samples are received. This means that you cannot immediately
 expire a metric only by changing the mapping configuration. At least one
 sample must be received for updated mappings to take effect. -->
TTLの設定は、マップされたそれぞれのメトリック名/ラベルの組み合わせに対して、サンプルが受信されるたびに保存される。
これは、マッピングの設定を変えただけですぐにメトリクスを無効にできないことを意味する。
更新されたマッピングが効果を得るためには、少なくとも1つのサンプルを受信する必要がある。

## Using Docker

<!-- You can deploy this exporter using the [prom/statsd-exporter](https://registry.hub.docker.com/u/prom/statsd-exporter/) Docker image. -->
Dockerイメージ[prom/statsd-exporter](https://registry.hub.docker.com/u/prom/statsd-exporter/)を使ってこのexporterをデプロイすることができる。

<!-- For example: -->
例
```bash
docker pull prom/statsd-exporter

docker run -d -p 9102:9102 -p 9125:9125 -p 9125:9125/udp \
        -v $PWD/statsd_mapping.yml:/tmp/statsd_mapping.yml \
        prom/statsd-exporter --statsd.mapping-config=/tmp/statsd_mapping.yml
```

<!-- 
[travis]: https://travis-ci.org/prometheus/statsd_exporter
[circleci]: https://circleci.com/gh/prometheus/statsd_exporter
[quay]: https://quay.io/repository/prometheus/statsd-exporter
[hub]: https://hub.docker.com/r/prom/statsd-exporter/
 -->