---
title: Goアプリケーションへのメトリクス組み込み
---

# PrometheusのためのGoアプリケーションへのメトリクス組み込み

Prometheusには、Goアプリケーションにメトリクスを組み込むために利用可能な公式[Goクライアントライブラリ](https://github.com/prometheus/client_golang)がある。
このガイドでは、HTTPでPrometheusメトリクスを出力する簡単なGoアプリケーションを作成する。

NOTE: 網羅的なAPIドキュメントは、Prometheusの各種Goライブラリの[GoDoc](https://godoc.org/github.com/prometheus/client_golang)を参照すること。

## インストール

このガイドで必要なライブラリ`prometheus`、`promauto`、`promhttp`を、[`go get`](https://golang.org/doc/articles/go_command.html)を利用して、インストールすることができる。

```bash
go get github.com/prometheus/client_golang/prometheus
go get github.com/prometheus/client_golang/prometheus/promauto
go get github.com/prometheus/client_golang/prometheus/promhttp
```

## How Go exposition works

GoアプリケーションでPrometheusメトリクスを出力するには、HTTPエンドポイント`/metrics`を提供する必要がある。
ハンドラー関数として、ライブラリ[`prometheus/promhttp`](https://godoc.org/github.com/prometheus/client_golang/prometheus/promhttp)のHTTP [`Handler`](https://godoc.org/github.com/prometheus/client_golang/prometheus/promhttp#Handler)を利用できる。

例えば、この最小限のアプリケーションは、Goアプリケーションのデフォルトのメトリクスを`http://localhost:2112/metrics`で出力する。

```go
package main

import (
        "net/http"

        "github.com/prometheus/client_golang/prometheus/promhttp"
)

func main() {
        http.Handle("/metrics", promhttp.Handler())
        http.ListenAndServe(":2112", nil)
}
```

このアプリケーションを起動するには、以下のコマンドを実行する。

```bash
go run main.go
```

メトリクスにアクセスするには以下のコマンドを実行する。

```bash
curl http://localhost:2112/metrics
```

## 独自メトリクスの追加

上記のアプリケーションは、デフォルトのGoメトリクスのみを出力する。
独自のアプリケーション固有のメトリクスを登録することもできる。
この例では、その時点までに処理された操作数を数える[カウンター](/docs/concepts/metric_types/#counter)`myapp_processed_ops_total`を出力する。
このカウンターは、2秒ごとに1ずつ増加する。

```go
package main

import (
        "net/http"
        "time"

        "github.com/prometheus/client_golang/prometheus"
        "github.com/prometheus/client_golang/prometheus/promauto"
        "github.com/prometheus/client_golang/prometheus/promhttp"
)

func recordMetrics() {
        go func() {
                for {
                        opsProcessed.Inc()
                        time.Sleep(2 * time.Second)
                }
        }()
}

var (
        opsProcessed = promauto.NewCounter(prometheus.CounterOpts{
                Name: "myapp_processed_ops_total",
                Help: "The total number of processed events",
        })
)

func main() {
        recordMetrics()

        http.Handle("/metrics", promhttp.Handler())
        http.ListenAndServe(":2112", nil)
}
```

このアプリケーションを起動するには、以下のコマンドを実行する。

```bash
go run main.go
```

メトリクスにアクセスするには以下のコマンドを実行する。

```bash
curl http://localhost:2112/metrics
```

メトリクスの出力の中で、ヘルプテキスト、型情報、カウンター`myapp_processed_ops_total`の現在の値を見ることができるだろう。

```
# HELP myapp_processed_ops_total The total number of processed events
# TYPE myapp_processed_ops_total counter
myapp_processed_ops_total 5
```

ローカルで稼働しているPrometheusインスタンスがこのアプリケーションからメトリクスを取得するように[設定](/docs/prometheus/latest/configuration/configuration/#<scrape_config>)できる。
`prometheus.yml`の設定例は以下の通り。

```yaml
scrape_configs:
- job_name: myapp
  scrape_interval: 10s
  static_configs:
  - targets:
    - localhost:2112
```

## その他のGoクライアントの機能

このガイドでは、PrometheusのGoクライアントライブラリで利用可能なほんの一握りの機能に触れただけである。
[ゲージ](https://godoc.org/github.com/prometheus/client_golang/prometheus#Gauge)や[ヒストグラム](https://godoc.org/github.com/prometheus/client_golang/prometheus#Histogram)のような他の型のメトリクスを出力することもできるし、[グローバルでないレジストリ](https://godoc.org/github.com/prometheus/client_golang/prometheus#Registry)、[Pushgateway](/docs/instrumenting/pushing/)に[メトリクスをプッシュする](https://godoc.org/github.com/prometheus/client_golang/prometheus/push)関数、Prometheusと[Graphite](https://godoc.org/github.com/prometheus/client_golang/prometheus/graphite)の連携などもある。

## まとめ

このガイドでは、Prometheusにメトリクスを出力する2つのGoアプリケーションのサンプル（デフォルトのGoメトリクスだけを出力するものと独自のPrometheusカウンターも出力するもの）を作成し、Prometheusインスタンスがそれらのアプリケーションからメトリクスを取得するように設定した。
