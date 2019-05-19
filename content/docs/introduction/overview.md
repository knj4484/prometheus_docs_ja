---
title: 概要
sort_rank: 1
---

# 概要

## Prometheusとは

[Prometheus](https://github.com/prometheus)は、オープンソースのシステム監視およびアラートのツールキットであり、もともとSoundCloudで開発されていた。
2012年の発足以来、多くの企業と組織がPrometheusを採用し、プロジェクトは活発な開発者と[ユーザーコミュニティ](https://prometheus.io/community/)を擁している。
現在は独立したオープンソースプロジェクトであり、いかなる企業からも独立して保守されている。
このことを強調するため、またプロジェクトのガバナンス構造を明確にするために、Prometheusは、[Cloud Native Computing Foundation](https://cncf.io/)に2016年に、[Kubernetes](http://kubernetes.io/)に次ぐ2番目のプロジェクトとして参加した。

さらに詳しいPrometheusの概要は、[メディア](/docs/introduction/media/)からリンクされているリソースを参照すること。

### 機能

Prometheusの主な機能は、

* メトリック名とキーバリューで特定される時系列データによる多次元[データモデル](/docs/concepts/data_model/)
* PromQL 上記の多次元データから最大限の価値を引き出す[柔軟なクエリ言語](/docs/prometheus/latest/querying/basics/)
* 分散ストレージに依存していないこと。単一のサーバーノードが自律している
* HTTPを通したpullモデルによる時系列データの収集
* 中間ゲートウェイによる[時系列データpush](/docs/instrumenting/pushing/)のサポート
* サービスディスカバリーと静的な設定による監視対象の検出
* 複数の方式でのグラフ化とダッシュボードのサポート

### コンポーネント

Prometheusのエコシステムは、複数のコンポーネントからなり、その多くは必須ではない

* 中心となる[Prometheusサーバー](https://github.com/prometheus/prometheus)、時系列データを取得・保存する
* アプリケーションコードをinstrumentするための[クライアントライブラリ](/docs/instrumenting/clientlibs/)
* 短命のジョブをサポートするための[Pushgateway](https://github.com/prometheus/pushgateway)
* HAProxyやStatsD、Graphiteなどのサービスのための特殊用途の[exporter](/docs/instrumenting/exporters/)
* アラートを処理する[alertmanager](https://github.com/prometheus/alertmanager)
* 各種サポートツール

ほとんどのPrometheusのコンポーネントは、静的なバイナリとして簡単にビルドとデプロイできるように、[Go](https://golang.org/)で書かれている

### アーキテクチャー

Prometheusとそのエコシステムのコンポーネントのアーキテクチャーを以下の図に示す

![Prometheus architecture](/assets/architecture.png)

Prometheusは、instrumentされたジョブから、直接（または短命のジョブに対してはPushgateway経由で）メトリクスを取得する。
取得したサンプルは全てローカルに保存され、集約したり、新しい時系列データを記録したり、アラートを生成するために、このデータに対してルールが実行される。
収集されたデータを可視化するために[Grafana](https://grafana.com/)や他のAPIコンシューマーを使うことが出来る。

## Prometheusが適しているのは

Prometheusは、数値のみから成るあらゆる時系列データの記録に対してうまく機能する。
マシン中心の監視にもサービス指向の動的なアーキテクチャの監視にも適している。
マイクロサービスの世界では、多次元データの収集とクエリのサポートが特に強みである。

Prometheusは、障害時に素早く問題を診断できるようなシステムとなるべく、信頼性に重きを置いて設計されている。
各Prometheusサーバーは、スタンドアローンで、ネットワークストレージやリモートのサービスに依存していない。
インフラの他の部分に障害があっても利用可能であり、Prometheusを利用するために大規模なインフラを用意する必要がない。


## Prometheusが適していないのは

Prometheusは信頼性に重きを置いている。システムについての統計を、いつでも、障害がおきている状況でも、見ることができる。
もし、リクエストごとの請求金額などのために、100%の精度が必要ならば、Prometheusは良い選択ではない。
なぜなら、収集されたデータはおそらく十分に詳細かつ完全ではないからである。
そのような場合には、請求額データの収集と分析には他のシステムを使い、残りの監視にPrometheusを使うのが良い。
