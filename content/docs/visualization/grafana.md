---
title: Grafana
sort_rank: 2
---

# GrafanaのPrometheusサポート

[Grafana](http://grafana.org/)はPrometheusのクエリをサポートしている。
Prometheus用のGrafanaデータソースは、Grafana 2.5.0 (2015-10-28)から含まれている。

以下にPrometheusからデータを取得するGrafanaのダッシュボードの例を示す。

[![Grafana screenshot](/assets/grafana_prometheus.png)](/assets/grafana_prometheus.png)

## インストール

完全なGrafanaのインストール手順は、[Grafana公式ドキュメント](http://docs.grafana.org/installation/)を参照すること。

例えば、Linuxでは、Grafanaのインストールは以下のようになる

```bash-lang
# Download and unpack Grafana from binary tar (adjust version as appropriate).
curl -L -O https://grafanarel.s3.amazonaws.com/builds/grafana-2.5.0.linux-x64.tar.gz
tar zxf grafana-2.5.0.linux-x64.tar.gz

# Start Grafana.
cd grafana-2.5.0/
./bin/grafana-server web
```

## 利用方法

デフォルトでは、Grafanaは[http://localhost:3000](http://localhost:3000)をリッスンしている。デフォルトのログインは、"admin" / "admin"である。

### Creating a Prometheus data source

Prometheusデータソースを作成するには

1. Grafanaのロゴをクリックしてサイドバーメニューを開く
2. サイドバーの「Data Sources」をクリックする
3. 「Add New」をクリックする
4. TypeでPrometheusを選択する
5. 適切なPrometheusサーバーのURL（例えば`http://localhost:9090/`）を入力する
6. 必要に応じて他のデータソースの設定を調整する（例えばプロキシアクセスをoffにする）
7. 「Add」をクリックして保存する

以下にデータソースの設定例を示す。

[![Data source configuration](/assets/grafana_configuring_datasource.png)](/assets/grafana_configuring_datasource.png)

### Prometheusのグラフの作成

Follow the standard way of adding a new Grafana graph. Then:
Grafanaの標準的なグラフ追加の手順に従うこと。そして

1. グラフタイトルをクリックし、「Edit」をクリックする
2. 「Metrics」タブで、Prometheusデータソース（右下）を選択する
3. 「Metric」欄で自動補完を通してメトリックを調べながら「Query」欄にPrometheusの式を入力する。
4- 時系列の凡例をフォーマットするために、「Legend format」を利用する。例えば、メソッドとクエリの結果のステータスだけダッシュで区.って表示したい場合は、「Legend format」に文字列`{{method}} - {{status}}`を使う
5. グラフ化がうまく行くまで他のグラフ設定を調整する

以下にPrometheusグラフの設定例を示す。
[![Prometheus graph creation](/assets/grafana_qps_graph.png)](/assets/grafana_qps_graph.png)

### Grafana.comからダッシュボードをインポートする

Grafana.comは、[共有されたダッシュボードの収集](https://grafana.com/dashboards)し、保守している。
それらは、ダウンロードし、Grafanaのスタンドアローンのインスタンスで利用できる。
Grafana.comの「Filter」機能を使って、「Prometheus」がデータソースのもののみを表示する。

現在、ダウンロードしたJSONファイルを手動で編集して、Prometheusサーバーに付けたGrafanaデーターソース名を反映するように`datasource:`の項目を修正しなければならない。
"Dashboards" → "Home" → "Import"を使って、編集したそのダッシュボードファイルをインポートする。