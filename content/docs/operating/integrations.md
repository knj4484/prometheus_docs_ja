---
title: インテグレーション
sort_rank: 5
---

# インテグレーション

[クライアントライブラリ](/docs/instrumenting/clientlibs/)、[エクスポーターおよび関連ライブラリ](/docs/instrumenting/exporters/)に加えて、Prometheusには一般的な連携ポイントが多くある。 このページでは、それらとのインテグレーションのうちいくつかを記載する。

機能が重なっていたり、開発中のものもあるので、全てのインテグレーションがここに記載されているわけではない。 [exporterデフォルトポート](https://github.com/prometheus/prometheus/wiki/Default-port-allocations)Wikiもこの種のexporter以外のインテグレーションのいくつかを含んでいる。

## ファイルサービスディスカバリー

Prometheusがサポートしていないサービスディスカバリーの仕組みに対しては、[ファイルベースのサービスディスカバリー](/docs/operating/configuration/#%3Cfile_sd_config%3E)が連携のインターフェースを提供している。

 * [Docker Swarm](https://github.com/ContainerSolutions/prometheus-swarm-discovery)
 * [Scaleway](https://github.com/scaleway/prometheus-scw-sd)

## Remote Endpoints and Storage

Prometheusの[remote write](/docs/operating/configuration/#%3Cremote_write%3E)と[remote read](/docs/operating/configuration/#%3Cremote_read%3E)の機能によって、値を透過的に送信・受信することができる。 これは、長期ストレージを主な目的としている。 ここに挙げられたソリューションが自分のデータ量を捌けるかを慎重に評価することをお勧めする。

  * [AppOptics](https://github.com/solarwinds/prometheus2appoptics): write
  * [Chronix](https://github.com/ChronixDB/chronix.ingester): write
  * [Cortex](https://github.com/cortexproject/cortex): read and write
  * [CrateDB](https://github.com/crate/crate_adapter): read and write
  * [Elasticsearch](https://github.com/infonova/prometheusbeat): write
  * [Gnocchi](https://gnocchi.xyz/prometheus.html): write
  * [Graphite](https://github.com/prometheus/prometheus/tree/master/documentation/examples/remote_storage/remote_storage_adapter): write
  * [InfluxDB](https://docs.influxdata.com/influxdb/latest/supported_protocols/prometheus): read and write
  * [IRONdb](https://github.com/circonus-labs/irondb-prometheus-adapter): read and write
  * [Kafka](https://github.com/Telefonica/prometheus-kafka-adapter): write
  * [M3DB](https://m3db.github.io/m3/integrations/prometheus): read and write
  * [OpenTSDB](https://github.com/prometheus/prometheus/tree/master/documentation/examples/remote_storage/remote_storage_adapter): write
  * [PostgreSQL/TimescaleDB](https://github.com/timescale/prometheus-postgresql-adapter): read and write
  * [SignalFx](https://github.com/signalfx/metricproxy#prometheus): write
  * [Splunk](https://github.com/lukemonahan/splunk_modinput_prometheus#prometheus-remote-write): write
  * [TiKV](https://github.com/bragfoo/TiPrometheus): read and write
  * [VictoriaMetrics](https://github.com/VictoriaMetrics/VictoriaMetrics): write
  * [Wavefront](https://github.com/wavefrontHQ/prometheus-storage-adapter): write

## Alertmanager Webhook Receiver

Alertmanagerがサポートしていない通知の仕組みに対しては、[webhook receiver](/docs/alerting/configuration/#webhook_config)によって連携することができる。

  * [AWS SNS](https://github.com/DataReply/alertmanager-sns-forwarder)
  * [DingTalk](https://github.com/timonwong/prometheus-webhook-dingtalk)
  * [IRC Bot](https://github.com/multimfi/bot)
  * [JIRAlert](https://github.com/free/jiralert)
  * [Phabricator / Maniphest](https://github.com/knyar/phalerts)
  * [prom2teams](https://github.com/idealista/prom2teams): Microsoft Teamsに通知を転送する
  * [SMS](https://github.com/messagebird/sachet): supports [multiple providers](https://github.com/messagebird/sachet/blob/master/examples/config.yaml)
  * [SNMP traps](https://github.com/maxwo/snmp_notifier)
  * [Telegram bot](https://github.com/inCaller/prometheus_bot)
  * [XMPP Bot](https://github.com/jelmer/prometheus-xmpp-alerts)

## 設定管理

Prometheusには、既存システムと連携できるようにしたり、既存システムの上に構築したりするための設定管理機能はない。

  * [Prometheus Operator](https://github.com/coreos/prometheus-operator): Kubernetes上でのPrometheusの管理をする
  * [Promgen](https://github.com/line/promgen): PrometheusとAlertmanagerのためのWeb UIおよび設定ジェネレーター

## その他

  * [karma](https://github.com/prymitive/karma): アラートダッシュボード
  * [PushProx](https://github.com/RobustPerception/PushProx): NATや類似のネットワーク構成を通過するためのプロキシ
  * [Promregator](https://github.com/promregator/promregator): Cloud Foundryのアプリケーションの検出とスクレイプ
