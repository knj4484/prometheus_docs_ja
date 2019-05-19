---
title: コンソールテンプレート
sort_rank: 3
---

# コンソールテンプレート

コンソールテンプレートによって、[Go templating language](http://golang.org/pkg/text/template/)で書かれた任意のコンソールを作成することができる。

コンソールテンプレートは、簡単にバージョン管理できるテンプレートを作成する最も強力な方法である。ただ、学習コストは高いので、このスタイルの監視に慣れていないユーザーは、まず[Grafana](/docs/visualization/grafana/)を試すのが良い。

## Getting started

Prometheusには、コンソールのサンプル集がある。これらは稼働しているPrometheusの`/consoles/index.html.example`で見ることができ、
Node Exporterが`job="node"`というラベルでスクレイプしているなら、Node Exporterのコンソールを表示する。

このサンプルコンソールは、5つの部分からなる

1. 上部のナビゲーションバー
2. 左側のメニュー
3. 下部の時間コントロール
4. 中央のメインコンテンツ（普通はグラフ）
5. 右側の表

ナビゲーションバーは、他のPrometheusなどの他システム、ドキュメントなどへのリンクである。メニューは、そのPrometheusサーバー自体の中のナビゲーションに用い、コンソールを新しいタブで開いて情報の相互関連性を見るために便利である。どちらも`console_libraries/menu.lib`で設定することが出来る。

時間コントロールによって、グラフの間隔と幅を変更することが出来る。コンソールのURLは共有することが出来て、他の人にも同じグラフを表示する。

メインコンテンツは、普通はグラフである。設定可能なJavaScriptのグラフライブラリが提供されていて、Prometheusからのデータのリクエストと[Rickshaw](https://tech.shutterstock.com/rickshaw/)を通した描画が出来る。

最後に、右側の表は、グラフよりコンパクトに統計情報を表示するために利用できる。

## コンソールの例

これは、基本的なコンソールで、右側の表にタスクの数、いくつのタスクがupか、平均メモリ使用量を表示し、メインコンテンツとして毎秒のクエリ数のグラフを表示する。

```
{{template "head" .}}

{{template "prom_right_table_head"}}
<tr>
  <th>MyJob</th>
  <th>{{ template "prom_query_drilldown" (args "sum(up{job='myjob'})") }}
      / {{ template "prom_query_drilldown" (args "count(up{job='myjob'})") }}
  </th>
</tr>
<tr>
  <td>CPU</td>
  <td>{{ template "prom_query_drilldown" (args
      "avg by(job)(rate(process_cpu_seconds_total{job='myjob'}[5m]))"
      "s/s" "humanizeNoSmallPrefix") }}
  </td>
</tr>
<tr>
  <td>Memory</td>
  <td>{{ template "prom_query_drilldown" (args
       "avg by(job)(process_resident_memory_bytes{job='myjob'})"
       "B" "humanize1024") }}
  </td>
</tr>
{{template "prom_right_table_tail"}}


{{template "prom_content_head" .}}
<h1>MyJob</h1>

<h3>Queries</h3>
<div id="queryGraph"></div>
<script>
new PromConsole.Graph({
  node: document.querySelector("#queryGraph"),
  expr: "sum(rate(http_query_count{job='myjob'}[5m]))",
  name: "Queries",
  yAxisFormatter: PromConsole.NumberFormatter.humanizeNoSmallPrefix,
  yHoverFormatter: PromConsole.NumberFormatter.humanizeNoSmallPrefix,
  yUnits: "/s",
  yTitle: "Queries"
})
</script>

{{template "prom_content_tail" .}}

{{template "tail"}}
```

`prom_right_table_head`と`prom_right_table_tail`テンプレートは、右側の表を含んでいる。これは必須ではない。

`prom_query_drilldown`は、渡された式を評価し、フォーマットし、[expressionブラウザ](/docs/visualization/browser/)の式へリンクするテンプレートである。
第1引数が式で、第2引数は単位、第3引数は出力のフォーマット方法である。第1引数のみ必須である。

`prom_query_drilldown`の第3引数の正当な出力フォーマットは以下の通り。

* 指定しない: デフォルトのGoのディスプレイ出力
* `humanize`: 結果を[SI接頭辞](https://ja.wikipedia.org/wiki/SI%E6%8E%A5%E9%A0%AD%E8%BE%9E)を用いて表示
- `humanizeNoSmallPrefix`: 1より大きな値は[SI接頭辞](https://ja.wikipedia.org/wiki/SI%E6%8E%A5%E9%A0%AD%E8%BE%9E)を用いて表示、1より小さな値は3桁表示。これは、`humanize`で生成されるmilliqueries *er secondのような単位を避けるために便利である
- `humanize1024`: 1000ではなく1024を底としたhumanize同様の表示をする。これは、`KiB`や`MiB`のような単位を生成するため*、`B`を第2引数にしたときによく利用される。
* `printf.3g`: 3桁表示

カスタムのフォーマットも定義できる。サンプルは、[prom.lib](https://github.com/prometheus/prometheus/blob/master/console_libraries/prom.lib)を参照のこと。

## グラフライブラリ

グラフライブラリは次のように呼び出す。

```
<div id="queryGraph"></div>
<script>
new PromConsole.Graph({
  node: document.querySelector("#queryGraph"),
  expr: "sum(rate(http_query_count{job='myjob'}[5m]))"
})
</script>
```

`head`テンプレートが必要なJavaScriptとCSSをロードする。

グラフライブラリのパラメーターは以下の通り。

| 名前          | 説明
| ------------- | -------------
| expr          | 必須。グラフにする式。リストも可
| node          | 必須。描画するためのDOMノード
| duration      | オプション。グラフの期間。デフォルトは1時間
| endTime       | オプション。グラフの最後のunixtime。デフォルトは現在
| width         | オプション。グラフの幅（タイトルを除く）。デフォルトは、自動検出
| height        | オプション。グラフの高さ（タイトルと凡例を除く）。デフォルトは200ピクセル
| min           | オプション。X軸最小値。デフォルトは、データの最低値
| max           | オプション。Y軸最大値。デフォルトは、データの最高値
| renderer      | オプション。グラフの種類。`line`または`area`。デフォルトは、`line`
| name          | オプション。凡例とhover detailで使われるプロットのタイトル。文字列が渡された場合、`[[ label ]]`はラベルの値で置換される。関数が渡された場合、ラベルのマップが渡されるので、文字列として名前を返すこと。リストも可
| xTitle        | オプション。X軸のタイトル。デフォルトは`Time`
| yUnits        | オプション。Y軸の単位。デフォルトは、空
| yTitle        | オプション。Y軸のタイトル。デフォルトは、空
| yAxisFormatter | オプション。Y軸の数値のフォーマッター。デフォルトは、`PromConsole.NumberFormatter.humanize`
| yHoverFormatter | オプション。hover detailの数値のフォーマッター。デフォルトは、`PromConsole.NumberFormatter.humanizeExact`
| colorScheme   | オプション。プロットで利用されるカラースキーム。hex color codeのリストまたはRickshawでサポートされている[カラースキーム名](https://github.com/shutterstock/rickshaw/blob/master/src/js/Rickshaw.Fixtures.Color.js)。デフォルトは、`'colorwheel'`

`expr`と`name`の両方がリストの場合、両者は同じ長さでなければならない。`name`は、対応する式のプロットに適用される。

`yAxisFormatter`と`yHoverFormatter`の正当なオプションは以下の通り。

* `PromConsole.NumberFormatter.humanize`: [SI接頭辞](https://ja.wikipedia.org/wiki/SI%E6%8E%A5%E9%A0%AD%E8%BE%9E)を用いたフォーマット
* `PromConsole.NumberFormatter.humanizeNoSmallPrefix`: 1より大きな値は[SI接頭辞](https://ja.wikipedia.org/wiki/SI%E6%8E%A5%E9%A0%AD%E8%BE%9E)を用いてフォーマット、1より小さな値は3桁を用いたフォーマット。これは、`humanize`で生成されるmilliqueries per secondのような単位を避けるために便利である
* `PromConsole.NumberFormatter.humanize1024`: 1000ではなく1024を底としたhumanize同様のフォーマット
