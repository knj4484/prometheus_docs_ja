---
title: メトリクスのプッシュ
sort_rank: 3
---

# メトリクスのプッシュ

スクレイプできないコンポーネントを監視しなければならないことがある。
Prometheusでは、[Pushgateway](https://github.com/prometheus/pushgateway)によって、[短命のサービスレベルのバッチジョブ](/docs/practices/pushing/)からPrometheusがスクレイプ可能な中間的なジョブへと時系列をプッシュ出来るようになる。
これとPrometheusの単純なテキストベースのフォーマットとの組み合わせによって、クライアントライブラリなしでシェルスクリプトにメトリクスを組み込むことでさえ簡単になっている。

* Pushgatewayの使い方とUnixシェルからの使い方に関するさらなる情報は、プロジェクトの[README.md](https://github.com/prometheus/pushgateway/blob/master/README.md)を参照

* Javaからの利用に関しては、[PushGateway](https://prometheus.github.io/client_java/io/prometheus/client/exporter/PushGateway.html)クラスを参照

* Goからの利用に関しては、[Push](http://godoc.org/github.com/prometheus/client_golang/prometheus#Push)と[PushAdd](http://godoc.org/github.com/prometheus/client_golang/prometheus#PushAdd)を参照

* Pythonからの利用は、[Exporting to a Pushgateway](https://github.com/prometheus/client_python#exporting-to-a-pushgateway)

* Rubyからの利用に関しては、[Pushgateway documentation](https://github.com/prometheus/client_ruby#pushgateway)を参照
