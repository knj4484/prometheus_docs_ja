---
title: ジョブとインスタンス
sort_rank: 3
---

# ジョブとインスタンス

Prometheus用語で、スクレイプすることができるエンドポイントは _インスタンス_ と呼ばれ、通常は1つのプロセスに対応している。 同じ目的のインスタンス（例えば、スケーラビリティや信頼性のために複製されたプロセス）の集合は _ジョブ_ と呼ばれる。

例えば、4つの複製されたインスタンスから成るAPIサーバーのジョブは、以下のように構成される。

   * job: `api-server`
      * instance 1: `1.2.3.4:5670`
      * instance 2: `1.2.3.4:5671`
      * instance 3: `5.6.7.8:5670`
      * instance 4: `5.6.7.8:5671`

## 自動生成されるラベルと時系列

Prometheusが監視対象をスクレイプするとき、監視対象を識別できるように、以下のラベルを時系列に自動的に付与する。

* `job`: その監視対象が含まれるジョブの名前
* `instance`: スクレイプされたURLの`<host>:<port>`の部分

スクレイプしたデータにこれらのいずれかのラベルが既にあった場合の挙動は、`honor_labels`の設定値によって異なる。 詳細は、[スクレイプの設定のドキュメント](/docs/operating/configuration/#%3Cscrape_config%3E)を参照すること。

それぞれのインスタンスのスクレイプで、Prometheusは以下の時系列に[サンプル](/docs/introduction/glossary#sample)を保存する。

* `up{job="<job-name>", instance="<instance-id>"}`: スクレイプが成功したら`1`、失敗したら`0`
* `scrape_duration_seconds{job="<job-name>", instance="<instance-id>"}`:
   スクレイプの時間
* `scrape_samples_post_metric_relabeling{job="<job-name>", instance="<instance-id>"}`:
   メトリックのリラベルが適用された後で残っているサンプルの数
* `scrape_samples_scraped{job="<job-name>", instance="<instance-id>"}`:
   監視対象が出力したサンプルの数

時系列`up`は、インスタンスの可用性を監視するのに利用できる。