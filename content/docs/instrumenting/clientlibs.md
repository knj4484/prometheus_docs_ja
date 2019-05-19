---
title: クライアントライブラリ
sort_rank: 1
---

# クライアントライブラリ

自分のサービスの監視をするには、Prometheusクライアントライブラリのどれかを利用して、そのコードにメトリクス組み込みを追加する必要がある。
それらのライブラリは、Prometheusの[メトリック型](/docs/concepts/metric_types/)を実装している。

アプリケーションが書かれている言語に合わせてPrometheusクライアントライブラリを選択すること。
これによって、内部メトリクスを定義し、アプリケーションのインスタンス上でHTTPのエンドポイントで出力出来るようになる。

* [Go](https://github.com/prometheus/client_golang)
* [Java or Scala](https://github.com/prometheus/client_java)
* [Python](https://github.com/prometheus/client_python)
* [Ruby](https://github.com/prometheus/client_ruby)

非公式のサードパーティのクライアントライブラリ

* [Bash](https://github.com/aecolley/client_bash)
* [C++](https://github.com/jupp0r/prometheus-cpp)
* [Common Lisp](https://github.com/deadtrickster/prometheus.cl)
* [Elixir](https://github.com/deadtrickster/prometheus.ex)
* [Erlang](https://github.com/deadtrickster/prometheus.erl)
* [Haskell](https://github.com/fimad/prometheus-haskell)
* [Lua](https://github.com/knyar/nginx-lua-prometheus) for Nginx
* [Lua](https://github.com/tarantool/prometheus) for Tarantool
* [.NET / C#](https://github.com/andrasm/prometheus-net)
* [Node.js](https://github.com/siimon/prom-client)
* [Perl](https://metacpan.org/pod/Net::Prometheus)
* [PHP](https://github.com/Jimdo/prometheus_client_php)
* [Rust](https://github.com/pingcap/rust-prometheus)

クライアントライブラリは、PrometheusがHTTPのエンドポイントをscrapeすると、追跡中の全てのメトリクスのその時点の状態をPrometheusに送信する。

もし利用可能なクライアントライブラリがなかったり、依存を避けたいのであれば、
自分で[出力フォーマット](/docs/instrumenting/exposition_formats/)の一つを実装してメトリクスを出力することも可能である。

新しいPrometheusクライアントライブラリを実装する際は、[クライアントの書き方のガイドライン](https://prometheus.io/docs/instrumenting/writing_clientlibs)に従ってください。
[開発メーリングリスト](https://groups.google.com/forum/#!forum/prometheus-developers)に相談することも検討して下さい。
どのようにしてあなたのライブラリを可能な限り便利で安定にするかアドバイス出来れば幸いです。
