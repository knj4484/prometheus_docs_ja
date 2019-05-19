---
title: FAQ
sort_rank: 5
toc: full-width
---

# よくある質問

## 一般

### Prometheusとは何か？

Prometheusは、活発なエコシステムを持つ、オープンソースのシステム監視およびアラートのツールキットである。
[概要](/docs/introduction/overview/)を参照。


### Prometheusを他の監視システムと比較してどうか？

[比較](/docs/introduction/comparison/)のページを参照。

### Prometheusはどのような依存があるか？

Prometheusサーバーはスタンドアローンで稼働し、外部の依存はない。

### Prometheusを高可用にすることはできるか？

はい。同一のPrometheusサーバーを2つ以上の別々のマシンで稼働させればよい。
同一のアラートは[Alertmanager](https://github.com/prometheus/alertmanager)が重複排除をする。

[Alertmanagerを高可用にする](https://github.com/prometheus/alertmanager#high-availability)には、[Mesh cluster](https://github.com/weaveworks/mesh)内で複数のインスタンスを実行し、Prometheusがそれぞれのインスタンスに通知を送信するように設定することができる。

### Prometheusはスケールしないと言われました

実際には、Prometheusをスケールさせたり連合(federate)させる様々な方法がある。
Robust Perception blogの[Scaling and Federating Prometheus](https://www.robustperception.io/scaling-and-federating-prometheus/)を参照すること。

### Prometheusはどの言語で書かれていますか？

PrometheusのほとんどのコンポーネントはGoで書かれている。
いくつかはJava、Python、Rubyでも書かれている。

### Prometheusの機能やストレージの形式、APIはどれぐらい安定していますか？

GitHubのPrometheusオーガニゼーションにあるバージョン1.0.0以上の全てのリポジトリは[セマンティック バージョニング](https://semver.org/lang/ja/)に従っている。
破壊的な変更は、メジャーバージョンのインクリメントで表される。
実験的なコンポーネントでは、例外がありうるが、アナウンスでその旨が明確に記される。

バージョン1.0.0にまだ達していないリポジトリでも、一般的に、かなり安定している。
各リポジトリで適切なリリースプロセスと1.0.0リリースを目指している。
破壊的な変更は何れにしてもリリースノートで示され（`[CHANGE]`と記される）、正式にリリースされていないコンポーネントに対しては、明確に伝えられる。

### なぜpushではなくpullなのか？

HTTPを通してpullすることには、たくさんの利点がある。

* 開発中に自分のラップトップから監視を実行できる
* 監視対象がダウンしていることがより簡単に分かる
* Webブラウザで監視対象を開き、手動でその状態を調べることができる

概して、pushよりもpullする方が少しだけ良いと信じているが、監視システムを検討する際の重要な点ではないであろう。

pushしなければならない場合のために、[Pushgateway](/docs/instrumenting/pushing/)を提供している。

### どうやってPrometheusにログを取り込みますか？

短い答え：やらないで下さい。[ELK stack](https://www.elastic.co/jp/products/)のようなものを使うこと。

長い答え：Prometheusはメトリクスを集めて処理するシステムであって、イベントロギングシステムではない。
Raintankのブログ記事[Logs and Metrics and Graphs, Oh My!](https://blog.raintank.io/logs-and-metrics-and-graphs-oh-my/)で、ログとメトリクスの違いについて詳細が説明されている。

アプリケーションのログからPrometheusのメトリクスを抽出したい場合は、Googleの[mtail](https://github.com/google/mtail)が役に立つであろう。

### 誰がPrometheusを書きましたか？

Prometheusは、最初は、[Matt T. Proud](http://www.matttproud.com)と[Julius Volz](http://juliusv.com)によって個人的に始められた。
初期の開発の大部分は[SoundCloud](https://soundcloud.com)に支援されていた。

今は、幅広い企業や個人によって保守・拡張されている。

### どのようなライセンスの下でPrometheusはリリースされていますか？

Prometheusは[Apache 2.0](https://github.com/prometheus/prometheus/blob/master/LICENSE)ライセンスの下でリリースされている。

### Prometheusの複数形は？

[広範囲の調査](https://youtu.be/B_CDeYrqxjQ)の後で、正しいPrometheusの複数形はPrometheisであると確定した。

### Prometheusの設定をリロードすることはできますか？

はい、Prometheusのプロセスに`SIGHUP`を送るか、エンドポイント`/-/reload`にHTTP POSTリクエスト送ると、設定ファイルをリロードして適用します。
様々なコンポーネントが、変更の失敗をうまく扱うように試みます。

### アラートを送信できますか？

[Alertmanager](https://github.com/prometheus/alertmanager)でできる。

現在、以下の外部システムがサポートされている。

* Email
* Generic Webhooks
* [HipChat](https://www.hipchat.com/)
* [OpsGenie](https://www.opsgenie.com/)
* [PagerDuty](http://www.pagerduty.com/)
* [Pushover](https://pushover.net/)
* [Slack](https://slack.com/)

### ダッシュボードを作成できますか？

できる。プロダクションでの利用には[Grafana](/docs/visualization/grafana/)を推奨している。
[コンソールテンプレート](/docs/visualization/consoles/)もある。

### タイムゾーンを変更できるか？なぜ全てUTCなのか？

タイムゾーンに関するあらゆる混乱（特に、いわゆるdaylight saving timeに関わる混乱）を避けるために、Prometheusの全てのコンポーネントで、内部的にはUnix timeのみを、表示にはUTCのみを用いると決められている。UIには、慎重に選ばれたタイムゾーンを導入することができるかもしれない。この試みに関する現状は[issue #500](https://github.com/prometheus/prometheus/issues/500)を参照のこと。

## メトリクス組み込み

### どの言語にメトリクスを組み込むためのライブラリがありますか？

自分のサービスにPrometheusのメトリクスを組み込むためのライブラリはたくさんある。
詳細は[クライアントライブラリ](/docs/instrumenting/clientlibs/)のドキュメントを参照すること。

新しい言語のためのクライアントライブラリに貢献する気がある場合、[出力フォーマット](/docs/instrumenting/exposition_formats/)を参照すること。

### マシンを監視することはできますか？

できる。[Node Exporter](https://github.com/prometheus/node_exporter)は、Linuxや他のUnixシステムのCPU使用率、メモリ、ディスク利用率、ファイルシステム、ネットワーク帯域のような広範にわたるマシンレベルのメトリクスを出力する。

### ネットワークデバイスを監視することはできますか？

できる。[SNMP Exporter](https://github.com/prometheus/snmp_exporter)によってSNMPをサポートしているデバイスを監視することができる。

### バッチジョブを関することはできますか？

[Pushgateway](/docs/instrumenting/pushing/)を使うことでできる。
バッチジョブの監視について[ベストプラクティス](/docs/instrumenting/pushing/)も参照すること。

### どんなアプリケーションがPrometheusですぐに監視できますか？

[exporterとインテグレーションの一覧](/docs/instrumenting/exporters/)を参照すること。

### JMXでJVMアプリケーションを監視できますか？

Javaクライアントで直接メトリクスを組み込めないアプリケーションに対しては、スタンドアロンでもJava Agentとしてでも[JMX Exporter](https://github.com/prometheus/jmx_exporter)を利用することができる。

### メトリクスを組み込んだ場合のパフォーマンスへの影響は？

パフォーマンスは、クライアントライブラリや言語によって様々であろう。
Javaに関しては、[ベンチマーク](https://github.com/prometheus/client_java/blob/master/benchmark/README.md)によると、Javaクライアントのカウンター/ゲージを一つインクリメントするのに12〜17nsかかることが示唆されている。
これは、レイテンシーが致命的なほとんどのコード以外全てで無視できる。

## 問題解決

### バージョン1.xのPrometheusサーバーが起動するのに長い時間がかかり、クラッシュ復旧に関するおびただしいログを出力します

Prometheusは`SIGTERM`の後で綺麗にシャットダウンしなければならず、激しく使われているサーバーではそれにしばらく時間がかかることもある。サーバーがクラッシュしたり、強制的に終了させられたり（例えば、メモリ不足でカーネルにkillされたり、Prometheusのシャットダウンの途中でランレベルシステムが待ちきれなくなったり）すると、クラッシュからの復旧が実行される。
これは普通の状況下では1分もかからないが、ある種の状況下ではかなり長い時間がかかることもあり得る。
詳細は、[crash recovery](/docs/prometheus/1.8/storage/#crash-recovery)を参照すること。

### Prometheus 1.xのサーバーがメモリ不足になります

Prometheusが利用可能なメモリ量を設定するために、[メモリ利用についてのセクション](/docs/prometheus/1.8/storage/#memory-usage)を参照すること。

### Prometheus 1.xのサーバーが“rushed mode”であるあるいは“storage needs throttling”だとレポートします

ストレージの負荷が高くなっている。
パフォーマンスをよくするためにどう設定を調整するか理解するために[ローカルストレージの設定についてのセクション](/docs/prometheus/1.8/storage/)を読むこと。

## 実装

### 全ての値が64-bit floatなのはなぜですか？integerがいいんですが…

設計を単純にするために64-bitのfloatに制限している。
[IEEE 754 倍精度浮動小数点数](https://ja.wikipedia.org/wiki/倍精度浮動小数点数)は、2^53までの精度の整数をサポートしている。
ネイティブの64bit integerをサポートしても、2^53〜2^63の精度の整数が必要な場合に役に立つだけである。
原理的には、他の型（64 bitよりも大きな整数の方を含む）のサポートは、実装可能ではあるが、現時点の優先事項ではない。
カウンターは、秒間100万回インクリメントされても、285年以上経たないと精度の問題にぶつからない。

### なぜPrometheusサーバーはTLSや認証をサポートしないのですか？追加できますか？

注意：Prometheusチームは、2018年8月11日のdevelopment summitでこれに関する姿勢を変更した。
提供しているエンドポイントのTLSと認証のサポートは[プロジェクトのロードマップ](/docs/introduction/roadmap/#tls-and-authentication-in-http-serving-endpoints)に記載されている。
コードが変更されたら、このドキュメントは更新されるだろう。

TLSと認証はよくリクエストされる機能ではあるが、意図的に、Prometheusのサーバーサイドのコンポーネントに実装してこなかった。
どちらも非常にたくさんの選択肢とパラメーターがある（TLSだけでも10以上ある）ので、サーバーコンポーネントで完全に一般的なTLSと認証をサポートするのではなく、可能な限り最善の監視システムを構築することに集中しようと決めていた。

TLSや認証が必要な場合、Prometheusの前にリバースプロキシを置くことを推奨する。
例えば、[Adding Basic Auth to Prometheus with
Nginx](https://www.robustperception.io/adding-basic-auth-to-prometheus-with-nginx/)を参照すること。

これはPrometheusへの接続に限った話である。
Prometheusは、[TLSや認証がかかったターゲットのスクレイプ](/docs/operating/configuration/#%3Cscrape_config%3E)をサポートしている。他のPrometheusコンポーネントにも同様のサポートがある。