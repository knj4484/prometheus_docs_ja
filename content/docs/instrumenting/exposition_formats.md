---
title: 出力フォーマット
sort_rank: 6
---

# 出力フォーマット

メトリクスは、単純な[テキストベース](#text-based-format)の出力フォーマットを利用してPrometheusに出力することができる。
このフォーマットを実装した様々なクライアントライブラリがある。
好みの言語にクライアントライブラリがなければ、[自分で実装する](/docs/instrumenting/writing_clientlibs/)ことができる。

NOTE: 古いバージョンのPrometheusには、現在のテキストベースのフォーマットに加えて、[Protocol Buffers](https://developers.google.com/protocol-buffers/)（Protobuf）をベースにした出力フォーマットをサポートしているバージョンもあるが、
バージョン2.0からは、PrometheusはProtobufベースのフォーマットをサポートしない。
この変更の背後にある理由については、[このドキュメント](https://github.com/RichiH/OpenMetrics/blob/master/markdown/protobuf_vs_text.md)で読むことができる。

## テキストベースのフォーマット

Prometheusバージョン2.0から、Prometheusにメトリクスを出力する全てのプロセスは、テキストベースのフォーマットを利用しなければならない。
ここでは、[テキストフォーマット詳細](#テキストフォーマット詳細)に加えて、このフォーマットの[基本情報](#基本情報)を示す。

### 基本情報

| 項目 | 説明 |
|--------|-------------|
| **策定時期** | April 2014  |
| **サポートバージョン** |  Prometheus version `>=0.4.0` |
| **送信方法** | HTTP |
| **エンコーディング** | UTF-8, 改行コード `\n` |
| **HTTP `Content-Type`** | `text/plain; version=0.0.4` (`version`の値がないと、最新のテキストフォーマットのバージョンで代替される) |
| **Optional HTTP `Content-Encoding`** | `gzip` |
| **利点** | <ul><li>人間の可読性</li><li>組み立てるのが容易、特に最小のケース（入れ子が不要）</li><li>行ごとに読む事ができる（型のヒントやdocstringは除く）</li></ul> |
| **制約** | <ul><li>冗長</li><li>型とdocstringが構文の必須要素ではない。したがって、メトリックの契約の検証が少ししか（あるいは全く）ない</li><li>パースのコスト</li></ul>|
| **サポートしているメトリック型** | <ul><li>Counter</li><li>Gauge</li><li>Histogram</li><li>Summary</li><li>Untyped</li></ul> |

### テキストフォーマット詳細

Prometheusのテキストベースフォーマットは、行指向である。
行は、line feed文字（`\n`）で分割される。
最後の行は、line feed文字で終わらなければならない。
空行は無視される。

#### 行のフォーマット

行の中で、トークンは、任意の数の空白やタブで分割することができる（最低1つ以上、さもなければ直前のトークンにまとめられる）。
最初や最後のホワイトスペースは無視される。

#### コメント、ヘルプテキスト、型情報

最初のホワイトスペース以外の文字が`#`の行は、コメントである。
それらは、`#`の直後のトークンが、`HELP`や`TYPE`でなければ、無視される。
そうである行は、次のように処理される。
そのトークンが`HELP`なら、少なくとももう1つのトークン（メトリック名）が期待されている。
残り全てのトークンは、そのメトリック名に対するdocstringと見なされる。
`HELP`行は、メトリック名の後ろに、UTF-8の文字の任意の並びを含んで良いが、バックスラッシュとline feedは、それぞれ`\\`や`\n`のようにエスケープする必要がある。
どのメトリック名に対しても、1つの`HELP`行だけ存在して良い。

そのトークンが`TYPE`なら、ちょうど2つのトークンが期待されている。
1つ目がメトリック名、2つ目は`ounter`、`gauge`、`histogram`、`summary`、`untyped`のいずれかでそのメトリック名の型を定義する。
`TYPE`行は、そのメトリック名が出力される最初のサンプルより前に現れなければならない。
もし、メトリック名に対して`TYPE`行がなければ、その型は`untyped`にセットされる。

残りの行は、以下の構文（[EBNF](https://ja.wikipedia.org/wiki/EBNF)）に従って、１行につ1サンプルで、サンプルを記述する。

```
metric_name [
  "{" label_name "=" `"` label_value `"` { "," label_name "=" `"` label_value `"` } [ "," ] "}"
] value [ timestamp ]
```

サンプル行の構文は、以下の通り。

* `metric_name`と`label_name`は、通常のPrometheusの制限に従う
* `label_value`は、UTF-8文字の任意の並びで良いが、バックスラッシュ、ダブルクオート、line feed文字は、それぞれ`\\`、`\"`、`\n`のようにエスケープしなければならない
* `value`は、Goの関数`ParseFloat()`の仕様に従ったfloatである。標準的な数値に加えて、`Nan`、`+Inf`、`-Inf`が有効な値で、それぞれ数値でない、正の無限、負の無限を表す
* `timestamp`は、Goの関数[`ParseInt()`](https://golang.org/pkg/strconv/#ParseInt)の仕様に従ったint64（エポック、つまり1970-01-01 00:00:00 UTCからの、うるう秒を除く、ミリ秒）である

#### グループ化とソート

あるメトリックのための全ての行は、オプションで`HELP`と`TYPE`の行を伴って、1つのグループとして提供されていなければならない。
さらに、繰り返して出力される際に、再現性のある並び順になっていることが望ましいが、必須ではない。言い換えると、計算コストが高価な場合にはソートしないこと。

各行は、メトリック名とラベルのユニークな組み合わせになっていなければならない。
そうでなければ、それを取り込む際の振る舞いは定義されていない。

#### ヒストグラムとサマリー

ヒストグラムとサマリーはテキストフォーマットで表すのが難しい。
以下の慣例を適用する。

* `x`という名前のサマリーやヒストグラムのサンプルの合計は、`x_sum`という名前の別のサンプルとして与えられる
* `x`という名前のサマリーやヒストグラムのサンプルの個数は、`x_count`という名前の別のサンプルとして与えられる
* `x`という名前のサマリーの分位数は、同じ名前`x`とラベル`{quantile="y"}`を持つ別のサンプル行として与えられる
- `x`という名前のバケットのカウントは、名前`x_bucket`とラベル`{le="y"}`（`y`はバケットの上限）を持つ別のサンプル行とし*与えられる
* ヒストグラムは、`{le="+Inf"}`のバケットを持たなければならない。その値は`x_count`の値と等しくなければならない
* ヒストグラムのバケットとサマリーの分位数は、それぞれ`le`、`quantile`に対する値が数値的に増加する順で現れなければならな*

### テキストフォーマット例

コメント、`HELP`、`TYPE`、ヒストグラム、サマリー、エスケープなど、一通り揃ったPrometheusのメトリクスの出力を以下に示す。

```
# HELP http_requests_total The total number of HTTP requests.
# TYPE http_requests_total counter
http_requests_total{method="post",code="200"} 1027 1395066363000
http_requests_total{method="post",code="400"}    3 1395066363000

# Escaping in label values:
msdos_file_access_time_seconds{path="C:\\DIR\\FILE.TXT",error="Cannot find file:\n\"FILE.TXT\""} 1.458255915e9

# Minimalistic line:
metric_without_timestamp_and_labels 12.47

# A weird metric from before the epoch:
something_weird{problem="division by zero"} +Inf -3982045

# A histogram, which has a pretty complex representation in the text format:
# HELP http_request_duration_seconds A histogram of the request duration.
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{le="0.05"} 24054
http_request_duration_seconds_bucket{le="0.1"} 33444
http_request_duration_seconds_bucket{le="0.2"} 100392
http_request_duration_seconds_bucket{le="0.5"} 129389
http_request_duration_seconds_bucket{le="1"} 133988
http_request_duration_seconds_bucket{le="+Inf"} 144320
http_request_duration_seconds_sum 53423
http_request_duration_seconds_count 144320

# Finally a summary, which has a complex representation, too:
# HELP rpc_duration_seconds A summary of the RPC duration in seconds.
# TYPE rpc_duration_seconds summary
rpc_duration_seconds{quantile="0.01"} 3102
rpc_duration_seconds{quantile="0.05"} 3272
rpc_duration_seconds{quantile="0.5"} 4773
rpc_duration_seconds{quantile="0.9"} 9001
rpc_duration_seconds{quantile="0.99"} 76656
rpc_duration_seconds_sum 1.7560473e+07
rpc_duration_seconds_count 2693
```

## 過去のバージョン

過去のフォーマットのバージョンの詳細は、[Client Data Exposition Format](https://docs.google.com/document/d/1ZjyKiKxZV83VI9ZKAXRGKaUKK2BIWCT7oiGBKDBpjEY/edit?usp=sharing)を参照すること。
