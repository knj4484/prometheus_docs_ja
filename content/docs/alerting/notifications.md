---
title: 通知テンプレートのリファレンス
sort_rank: 7
---
# 通知テンプレートのリファレンス

Prometheusはアラートを作成しAlertmanagerに送信する。Alertmanagerは、アラートのラベルに基づいて様々なレシーバーに通知を送る。レシーバーは、Slack、PagerDuty、eメール、一般的なwebhookインターフェースを介した連携先の一つである。

レシーバーの送信される通知は、テンプレートによって構成される。Alertmanagerには、デフォルトのテンプレートが備えられているが、テンプレートをカスタマイズすることもできる。
Prometheusでアラートルールのラベル/アノテーションにもテンプレーティングが含まれるが、Alertmanagerテンプレートは[Prometheusでのテンプレーティング](https://prometheus.io/docs/visualization/template_reference/)とは別物であることに注意するのが、混乱を避けるために重要である。

Alertmanagerの通知のテンプレートは、[Go templating](http://golang.org/pkg/text/template)を基にしている。テキストとして評価されるフィールドもあるし、エスケープに影響するHTMLとして評価されるフィールドもあることに注意すること。

# データ構造

## Data

`Data`は、通知テンプレートとwebhookに渡されるデータ構造である。

| 名前          | 型     | 備考    |
| ------------- | ------------- | -------- |
| Receiver | string | 通知が送信されるレシーバー（Slack、eメールなど）の名前を定義する |
| Status | string | 少なくとも一つのアラートがfiringであればfiring、そうでなければresolvedと定義される |
| Alerts | [Alert](#alert) | [下記のAlert](#alert)オブジェクトのリスト |
| GroupLabels | [KV](#kv) | これらのアラートがグルーピングされるラベル |
| CommonLabels | [KV](#kv) | 全てのアラートに共通のラベル |
| CommonAnnotations | [KV](#kv) | 全てのアラートの共通のアノテーション。より長く追加的なアラートに関する情報の文字列に利用される |
| ExternalURL | string | 通知を送ったAlertmanagerへ戻るためのリンク |

## Alert

`Alert`は、通知のテンプレートのための一つのアラートの情報を持っている。

| 名前          | 型     | 備考    |
| ------------- | ------------- | -------- |
| Status | string | アラートがresolvedなのかfiringなのかを定義する |
| Labels | [KV](#kv) | アラートに付与されたラベルの集合 |
| Annotations | [KV](#kv) | アラートのアノテーションの集合 |
| StartsAt | time.Time | アラートがfiringになり始めた時刻。もしなければ、Alertmanagerによって現在時刻が付与される |
| EndsAt | time.Time | アラートの終了時刻が分かる場合のみセットされる。そうでなければ、最後にアラートが受信されてからの設定可能なタイムアウトの間隔にセットされる |
| GeneratorURL | string | このアラートを起こしたエンティティを特定するリンク |

## KV

`KV`は、ラベルとアノテーションを表すためのkey/valueの文字列ペアの集合である。

```
type KV map[string]string
```

二つのアノテーションを含むアノテーションの例

```
{
  summary: "alert summary",
  description: "alert description",
}
```

`KV`として保存されているデータへの直接のアクセスに加えて、ソート、削除、LabelSetsを見るためのメソッドがある。

### KVのメソッド
|  名前         | 引数          | 戻り値   | 備考     |
| ------------- | ------------- | -------- | -------- |
| SortedPairs | - | Pairs (list of key/value string pairs.) | ソートされたkey/valueペアのリストを返す |
| Remove | []string | KV | 指定されたキーを削除したkey/valueのマップのコピーを返す |
| Names | - | []string | LabelSet中のラベル名を返す |
| Values | - | []string | LabelSet中の値のリストを返す |

# 関数

Go templatingによって[default functions](http://golang.org/pkg/text/template/#hdr-Functions)も提供されていることに注意。

## 文字列

|  名前         | 引数          | 戻り値   | 備考     |
| ------------- | ------------- | -------- | -------- |
| title | string |[strings.Title](http://golang.org/pkg/strings/#Title), 各単語の最初の文字を大文字にする |
| toUpper | string | [strings.ToUpper](http://golang.org/pkg/strings/#ToUpper), 全ての文字を大文字にする |
| toLower | string | [strings.ToLower](http://golang.org/pkg/strings/#ToLower), 全ての文字を小文字にする |
| match | pattern, string | [Regexp.MatchString](https://golang.org/pkg/regexp/#MatchString). Regexpを利用して文字列のマッチングをする |
| reReplaceAll | pattern, replacement, text | [Regexp.ReplaceAllString](http://golang.org/pkg/regexp/#Regexp.ReplaceAllString) 正規表現置換。アンカーなし |
| join | sep string, s []string | [strings.Join](http://golang.org/pkg/strings/#Join), `s`の要素を連結して一つの文字列を作る。結果の文字列で、要素間には文字列`sep`が置かれる。テンプレートでのパイプラインを簡単にするために、引数の順序が逆になっていることに注意すること |
| safeHtml | text string | [html/template.HTML](https://golang.org/pkg/html/template/#HTML), 自動エスケープを必要としないように、文字列をHTMLとしてマークする |
