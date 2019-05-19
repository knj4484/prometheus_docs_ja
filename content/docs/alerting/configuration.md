---
title: 設定
sort_rank: 3
nav_icon: sliders
---

# 設定

[Alertmanager](https://github.com/prometheus/alertmanager)は、コマンドラインフラグと設定ファイルを通して設定される。
コマンドラインフラグは、不変のシステムパラメーターを設定し、設定ファイルは、inhibitionルール、通知ルーティング、通知レシーバーを定義する。

[visual editor](/webtools/alerting/routing-tree-editor)が、ルーティングツリーの構築の補助となる。

利用可能な全てのコマンドラインフラグを見るには、`alertmanager -h`を実行する。

Alertmanagerは、実行時に設定を読み込み直すことが出来る。
もし、新しい設定が正しい形式でなければ、変更は適用されず、エラーがログされる。
設定を読み込み直させるには、AlertmanagerプロセスにSIGHUPを送るか、エンドポイント`/-/reload`にHTTP POSTリクエストを送る。

## 設定ファイル

どの設定ファイルを読み込むか指定するには、フラグ`--config.file`を使う。

```bash
./alertmanager --config.file=simple.yml
```

設定ファイルは、以下に記すスキームで定義された[YAML形式](http://en.wikipedia.org/wiki/YAML)で書かれる。 ブラケットは、パラメーターがオプションであることを表す。 リストでないパラメーターは、指定されたデフォルト値にセットされる。

一般的なプレースホルダーは以下の通り。

* `<duration>`: 正規表現`[0-9]+(ms|[smhdwy])`にマッチする時間幅
* `<labelname>`: 正規表現`[a-zA-Z_][a-zA-Z0-9_]*`にマッチする文字列
* `<labelvalue>`: Unicodeの文字列
* `<filepath>`: カレントワーキングディレクトリのパス
* `<boolean>`: 値として`true`または`false`を取ることが出来るブーリアン
* `<string>`: 文字列
* `<secret>`: パスワードのような秘密の文字列
* `<tmpl_string>`: 利用前にテンプレートで展開される文字列
* `<tmpl_secret>`: 利用前にテンプレートで展開される秘密の文字列

そのほかのプレースホルダーは、個別に定義される。

A valid example file can be found [here](https://github.com/prometheus/alertmanager/blob/master/doc/examples/simple.yml).

グローバルな設定は、他の全ての部分で有効なパラメーターを指定する。
これらは、他の部分での設定のデフォルト値にもなる。

```yaml
global:
  # ResolveTimeoutは、アラートが更新されていない場合に、
  # 解消されたと宣言するまでの時間である
  [ resolve_timeout: <duration> | default = 5m ]

  # デフォルトのSMTPのFromヘッダー
  [ smtp_from: <tmpl_string> ]
  # The default SMTP smarthost used for sending emails, including port number.
  # ポート番号は、普通は25で、（時にSTARTTLSと呼ばれる）SMTP over TLSなら587である。
  # Example: smtp.example.org:587
  [ smtp_smarthost: <string> ]
  # SMTPサーバーを特定するためのデフォルトのホスト名
  [ smtp_hello: <string> | default = "localhost" ]
  [ smtp_auth_username: <string> ]
  # SMTP Auth using LOGIN and PLAIN.
  [ smtp_auth_password: <secret> ]
  # SMTP Auth using PLAIN.
  [ smtp_auth_identity: <string> ]
  # SMTP Auth using CRAM-MD5. 
  [ smtp_auth_secret: <secret> ]
  # The default SMTP TLS requirement.
  [ smtp_require_tls: <bool> | default = true ]

  # The API URL to use for Slack notifications.
  [ slack_api_url: <secret> ]
  [ victorops_api_key: <secret> ]
  [ victorops_api_url: <string> | default = "https://alert.victorops.com/integrations/generic/20131114/alert/" ]
  [ pagerduty_url: <string> | default = "https://events.pagerduty.com/v2/enqueue" ]
  [ opsgenie_api_key: <secret> ]
  [ opsgenie_api_url: <string> | default = "https://api.opsgenie.com/" ]
  [ hipchat_api_url: <string> | default = "https://api.hipchat.com/" ]
  [ hipchat_auth_token: <secret> ]
  [ wechat_api_url: <string> | default = "https://qyapi.weixin.qq.com/cgi-bin/" ]
  [ wechat_api_secret: <secret> ]
  [ wechat_api_corp_id: <string> ]

  # The default HTTP client configuration
  [ http_config: <http_config> ]

# 独自の通知テンプレートの定義が読み込まれるファイル
# 最後の部分はワイルドカードマッチャーを利用可。例 'templates/*.tmpl'
templates:
  [ - <filepath> ... ]

# The root node of the routing tree.
route: <route>

# A list of notification receivers.
receivers:
  - <receiver> ...

# A list of inhibition rules.
inhibit_rules:
  [ - <inhibit_rule> ... ]
```

## `<route>`

`route`ブロックは、ルーティングツリーの一つのノードおよびその子ノードを定義する。オプションの設定項目は、設定されていなければ、親ノードから引き継ぐ。

各アラートは、トップレベルの`route`からルーティングツリーに入り、子ノードをtraverseする。`continue`がfalseの場合、最初にマッチした子ノードの後で停止する。`continue`がマッチしたノードでtrueの場合、アラートは後続のノードに対してもマッチングを継続する。もし、どの子ノードにもマッチしなければ（マッチする子ノードがない場合も子ノード自体がない場合も）、アラートはカレントノードの設定に基づいて処理される。

```yaml
[ receiver: <string> ]
# これらのラベルで入ってきたアラートがまとめられる。例えば、cluster=Aと
# alertname=LatencyHighに対して入ってきた複数のアラートが1つのグループにまとめられる。
#
# あり得るラベル全てでまとめるためには、特殊値'...'を1つのラベル名として使う。
# 例えば、以下のようにする。
# group_by: ['...'] 
# これは、全てのアラートをそのまま通過させ、実質的に集約を完全に無効にする。
# アラートの量が非常に少なかったり、上流のシステムがグルーピングをするのでなければ、これは望んでいることではないだろう。
[ group_by: '[' <labelname>, ... ']' ]

# アラートが後続の兄弟ノードとのマッチングを継続すべきかどうか
[ continue: <boolean> | default = false ]

# このノードとマッチするためにアラートが満たすべき等値マッチャー集合
match:
  [ <labelname>: <labelvalue>, ... ]

# このノードとマッチするためにアラートが満たすべき正規表現マッチャー集合
match_re:
  [ <labelname>: <regex>, ... ]

# アラートのグループが通知を送信するために最初に待つべき時間の長さ。
# アラートの抑制が同じグループに初期アラートをより多く集めることができるようになる。
# 通常は、~0sから数分
[ group_wait: <duration> | default = 30s ]

# 初期通知が既に送信されたグループに追加された新しいアラート
# の通知を送信するまでに待つ時間の長さ
# （通常は、~5mかそれ以上）
[ group_interval: <duration> | default = 5m ]

# 通知が既に送信成功している場合、再送信するまでに待つ時間の長さ
# （通常は、~3hまたはそれ以上）
[ repeat_interval: <duration> | default = 4h ]

# 0以上の子ルート
routes:
  [ - <route> ... ]
```

### 例

```yaml
# 全てのパラメーターを持つrootルート。これらのパラメーターは、子ルートで
# 上書きされてなければ、引き継がれる
route:
  receiver: 'default-receiver'
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  group_by: [cluster, alertname]
  # 以降の子ルートにマッチしないアラートは全て、rootノードに残り、
  # 'default-receiver'に送信される
  routes:
  # service=mysqlまたはservice=cassandraを持つアラートは、
  # database pagerに送信される
  - receiver: 'database-pager'
    group_wait: 10s
    match_re:
      service: mysql|cassandra
  # team=frontendというラベルを持つアラートはこのサブルートにマッチする。
  # clusterとalertnameではなくproductとenvironmentでまとめられる。
  # and alertname.
  - receiver: 'frontend-pager'
    group_by: [product, environment]
    match:
      team: frontend
```

## `<inhibit_rule>`

inhibitionルールは、あるマッチャー集合にマッチするアラート（source）があった場合に、別のマッチャー集合にマッチするアラート（target）をミュートする。
targetとsourceはどちらも、`equal`リストのラベルについて同じラベル値を持っていなければならない。

存在しないラベルと値が空のラベルは、意味的に同じものである。
したがって、`equal`に挙げられたラベル名全てがsourceとtargetのアラートどちらにもない場合、そのinhibitionルールは適用される。

inhibitionルールは、あるアラートが自分自身を抑制しないように、targetとsourceのどちらの側のルールにもマッチするアラートを抑制することはない。
しかしながら、アラートがtargetとsourceのマッチャーの両方にマッチしないようにマッチャーを選ぶようにすることを推奨する。
そう設定した方がはるかに分かりやすいし、この特殊ケースが起きなくなる。

```yaml
# ミュートすべきアラートが満たすべきマッチャー
target_match:
  [ <labelname>: <labelvalue>, ... ]
target_match_re:
  [ <labelname>: <regex>, ... ]

# inhibitionが効果を持つために存在すべきアラートに対するマッチャー
source_match:
  [ <labelname>: <labelvalue>, ... ]
source_match_re:
  [ <labelname>: <regex>, ... ]

# inhibitionが効果を持つために、sourceとtargetのアラート両方で同じ値であるべきラベル
[ equal: '[' <labelname>, ... ']' ]

```

## `<http_config>`

`http_config`によって、レシーバーがHTTPベースのAPIサービスと通信するために使うHTTPクライアントを設定できる。

```yaml
# `basic_auth`、`bearer_token`、`bearer_token_file`は相互排他的であることに注意。

# 設定されたusernameとpasswordでscrapeのリクエストの`Authorization`ヘッダをセットする。
# passwordとpassword_fileは相互排他的である。
basic_auth:
  [ username: <string> ]
  [ password: <secret> ]
  [ password_file: <string> ]

# 設定された署名なしトークン (Bearer Token)でリクエストの`Authorization`ヘッダをセットする。
[ bearer_token: <secret> ]

# 設定ファイルから読み込んだ署名なしトークン (Bearer Token)でリクエストの`Authorization`
# ヘッダをセットする。
[ bearer_token_file: <filepath> ]

# TLSの設定をする
tls_config:
  [ <tls_config> ]

# プロキシURL（オプション）
[ proxy_url: <string> ]
```

## `<tls_config>`

`tls_config`によって、TLS接続の設定ができる。

```yaml
# APIサーバーを証明するためのCA証明書
[ ca_file: <filepath> ]

# クライアント証明書を認証するための証明書と鍵ファイル
[ cert_file: <filepath> ]
[ key_file: <filepath> ]

# Server Name Indicationのためのサーバー名
# https://tools.ietf.org/html/rfc4366#section-3.1
[ server_name: <string> ]

# サーバー証明書の検証の無効化
[ insecure_skip_verify: <boolean> | default = false]
```

## `<receiver>`

receiverは、1つ以上の通知連携の名前付きの設定である。

__新しいレシーバーは積極的に追加されていないので、独自の通知連携を[webhook](/docs/alerting/configuration/#webhook_config)レシーバーで実装することを推奨する。__

```yaml
# ユニークなレシーバーの名前
name: <string>

# 通知連携の設定
email_configs:
  [ - <email_config>, ... ]
hipchat_configs:
  [ - <hipchat_config>, ... ]
pagerduty_configs:
  [ - <pagerduty_config>, ... ]
pushover_configs:
  [ - <pushover_config>, ... ]
slack_configs:
  [ - <slack_config>, ... ]
opsgenie_configs:
  [ - <opsgenie_config>, ... ]
webhook_configs:
  [ - <webhook_config>, ... ]
victorops_configs:
  [ - <victorops_config>, ... ]
wechat_configs:
  [ - <wechat_config>, ... ]
```

## `<email_config>`

```yaml
# Whether or not to notify about resolved alerts.
[ send_resolved: <boolean> | default = false ]

# The email address to send notifications to.
to: <tmpl_string>

# The sender address.
[ from: <tmpl_string> | default = global.smtp_from ]

# The SMTP host through which emails are sent.
[ smarthost: <string> | default = global.smtp_smarthost ]

# The hostname to identify to the SMTP server.
[ hello: <string> | default = global.smtp_hello ]

# SMTP authentication information.
[ auth_username: <string> | default = global.smtp_auth_username ]
[ auth_password: <secret> | default = global.smtp_auth_password ]
[ auth_secret: <secret> | default = global.smtp_auth_secret ]
[ auth_identity: <string> | default = global.smtp_auth_identity ]

# The SMTP TLS requirement.
[ require_tls: <bool> | default = global.smtp_require_tls ]

# TLS configuration.
tls_config:
  [ <tls_config> ]

# The HTML body of the email notification.
[ html: <tmpl_string> | default = '{{ template "email.default.html" . }}' ]
# The text body of the email notification.
[ text: <tmpl_string> ]

# Further headers email header key/value pairs. Overrides any headers
# previously set by the notification implementation.
[ headers: { <string>: <tmpl_string>, ... } ]
```

## `<hipchat_config>`

HipChat notifications use a [Build Your Own](https://confluence.atlassian.com/hc/integrations-with-hipchat-server-683508267.html) integration.

```yaml
# Whether or not to notify about resolved alerts.
[ send_resolved: <boolean> | default = false ]

# The HipChat Room ID.
room_id: <tmpl_string>
# The auth token.
[ auth_token: <secret> | default = global.hipchat_auth_token ]
# The URL to send API requests to.
[ api_url: <string> | default = global.hipchat_api_url ]

# See https://www.hipchat.com/docs/apiv2/method/send_room_notification
# A label to be shown in addition to the sender's name.
[ from:  <tmpl_string> | default = '{{ template "hipchat.default.from" . }}' ]
# The message body.
[ message:  <tmpl_string> | default = '{{ template "hipchat.default.message" . }}' ]
# Whether this message should trigger a user notification.
[ notify:  <boolean> | default = false ]
# Determines how the message is treated by the alertmanager and rendered inside HipChat. Valid values are 'text' and 'html'.
[ message_format:  <string> | default = 'text' ]
# Background color for message.
[ color:  <tmpl_string> | default = '{{ if eq .Status "firing" }}red{{ else }}green{{ end }}' ]

# The HTTP client's configuration.
[ http_config: <http_config> | default = global.http_config ]
```

## `<pagerduty_config>`

PagerDutyへの通知は、[PagerDuty API](https://developer.pagerduty.com/documentation/integration/events)を通して送信される。 PagerDutyは、連携の仕方のドキュメントを[ここ](https://www.pagerduty.com/docs/guides/prometheus-integration-guide/)で提供している。

```yaml
# 解決した(resolved)アラートを通知するかどうか
[ send_resolved: <boolean> | default = true ]

# 次の2つのオプションは、相互排他的である。
# PagerDutyのインテグレーションキー（PagerDutyのインテグレーションタイプ`Events API v2`を利用する場合）
routing_key: <tmpl_secret>
# PagerDutyのインテグレーションキー（PagerDutyのインテグレーションタイプ`Prometheus`を利用する場合）
service_key: <tmpl_secret>

# APIリクエストの送信先のURL
[ url: <string> | default = global.pagerduty_url ]

# Alertmanagerのクライアントを識別するもの
[ client:  <tmpl_string> | default = '{{ template "pagerduty.default.client" . }}' ]
# 通知の送信元へのバックリンク
[ client_url:  <tmpl_string> | default = '{{ template "pagerduty.default.clientURL" . }}' ]

# インシデントの記述
[ description: <tmpl_string> | default = '{{ template "pagerduty.default.description" .}}' ]

# インシデントの深刻度
[ severity: <tmpl_string> | default = 'error' ]

# さらにインシデントの詳細を与えるための任意のキー/バリューのペアの集合
[ details: { <string>: <tmpl_string>, ... } | default = {
  firing:       '{{ template "pagerduty.default.instances" .Alerts.Firing }}'
  resolved:     '{{ template "pagerduty.default.instances" .Alerts.Resolved }}'
  num_firing:   '{{ .Alerts.Firing | len }}'
  num_resolved: '{{ .Alerts.Resolved | len }}'
} ]

# インシデントに添付する画像
images:
  [ <image_config> ... ]

# インシデントに添付するリンク
links:
  [ <link_config> ... ]

# HTTPクライアントの設定
[ http_config: <http_config> | default = global.http_config ]
```

### `<image_config>`

これらのフィールドは、[PagerDuty API documentation](https://v2.developer.pagerduty.com/v2/docs/send-an-event-events-api-v2#section-the-images-property)でドキュメント化されている。

```yaml
source: <tmpl_string>
alt: <tmpl_string>
text: <tmpl_string>
```

### `<link_config>`

これらのフィールドは、[PagerDuty API documentation](https://v2.developer.pagerduty.com/v2/docs/send-an-event-events-api-v2#section-the-links-property)でドキュメント化されている。

```yaml
href: <tmpl_string>
text: <tmpl_string>
```

## `<pushover_config>`

Pushover notifications are sent via the [Pushover API](https://pushover.net/api).

```yaml
# Whether or not to notify about resolved alerts.
[ send_resolved: <boolean> | default = true ]

# The recipient user’s user key.
user_key: <secret>

# Your registered application’s API token, see https://pushover.net/apps
token: <secret>

# Notification title.
[ title: <tmpl_string> | default = '{{ template "pushover.default.title" . }}' ]

# Notification message.
[ message: <tmpl_string> | default = '{{ template "pushover.default.message" . }}' ]

# A supplementary URL shown alongside the message.
[ url: <tmpl_string> | default = '{{ template "pushover.default.url" . }}' ]

# Priority, see https://pushover.net/api#priority
[ priority: <tmpl_string> | default = '{{ if eq .Status "firing" }}2{{ else }}0{{ end }}' ]

# How often the Pushover servers will send the same notification to the user.
# Must be at least 30 seconds.
[ retry: <duration> | default = 1m ]

# How long your notification will continue to be retried for, unless the user
# acknowledges the notification.
[ expire: <duration> | default = 1h ]

# The HTTP client's configuration.
[ http_config: <http_config> | default = global.http_config ]
```

## `<slack_config>`

Slack通知は、Slackの[Webhook](https://api.slack.com/incoming-webhooks)を通して送信される。
この通知には[attachment](https://api.slack.com/docs/message-attachments)が含まれる。

```yaml
# 解消された(resolved)アラートについて通知するかどうか
[ send_resolved: <boolean> | default = false ]

# Slack webhookのURL
[ api_url: <secret> | default = global.slack_api_url ]

# 通知の送信先のチャネルまたはユーザー
channel: <tmpl_string>

# Slack webhook APIで定義されているAPIリクエストデータ
[ icon_emoji: <tmpl_string> ]
[ icon_url: <tmpl_string> ]
[ link_names: <boolean> | default = false ]
[ username: <tmpl_string> | default = '{{ template "slack.default.username" . }}' ]
# The following parameters define the attachment.
actions:
  [ <action_config> ... ]
[ callback_id: <tmpl_string> | default = '{{ template "slack.default.callbackid" . }}' ]
[ color: <tmpl_string> | default = '{{ if eq .Status "firing" }}danger{{ else }}good{{ end }}' ]
[ fallback: <tmpl_string> | default = '{{ template "slack.default.fallback" . }}' ]
fields:
  [ <field_config> ... ]
[ footer: <tmpl_string> | default = '{{ template "slack.default.footer" . }}' ]
[ pretext: <tmpl_string> | default = '{{ template "slack.default.pretext" . }}' ]
[ short_fields: <boolean> | default = false ]
[ text: <tmpl_string> | default = '{{ template "slack.default.text" . }}' ]
[ title: <tmpl_string> | default = '{{ template "slack.default.title" . }}' ]
[ title_link: <tmpl_string> | default = '{{ template "slack.default.titlelink" . }}' ]
[ image_url: <tmpl_string> ]
[ thumb_url: <tmpl_string> ]

# The HTTP client's configuration.
[ http_config: <http_config> | default = global.http_config ]
```

### `<action_config>`

このフィールドは、[Slack APIドキュメント](https://api.slack.com/docs/message-attachments#action_fields)に記載されている。

```yaml
type: <tmpl_string>
text: <tmpl_string>
url: <tmpl_string>
[ style: <tmpl_string> [ default = '' ]
```

### `<field_config>`

このフィールドは、[Slack APIドキュメント](https://api.slack.com/docs/message-attachments#fields)に記載されている。

```yaml
title: <tmpl_string>
value: <tmpl_string>
[ short: <boolean> | default = slack_config.short_fields ]
```

## `<opsgenie_config>`

OpsGenie notifications are sent via the [OpsGenie API](https://docs.opsgenie.com/docs/alert-api).

```yaml
# Whether or not to notify about resolved alerts.
[ send_resolved: <boolean> | default = true ]

# The API key to use when talking to the OpsGenie API.
[ api_key: <secret> | default = global.opsgenie_api_key ]

# The host to send OpsGenie API requests to.
[ api_url: <string> | default = global.opsgenie_api_url ]

# Alert text limited to 130 characters.
[ message: <tmpl_string> ]

# A description of the incident.
[ description: <tmpl_string> | default = '{{ template "opsgenie.default.description" . }}' ]

# A backlink to the sender of the notification.
[ source: <tmpl_string> | default = '{{ template "opsgenie.default.source" . }}' ]

# A set of arbitrary key/value pairs that provide further detail
# about the incident.
[ details: { <string>: <tmpl_string>, ... } ]

# Comma separated list of team responsible for notifications.
[ teams: <tmpl_string> ]

# Comma separated list of tags attached to the notifications.
[ tags: <tmpl_string> ]

# Additional alert note.
[ note: <tmpl_string> ]

# Priority level of alert. Possible values are P1, P2, P3, P4, and P5.
[ priority: <tmpl_string> ]

# The HTTP client's configuration.
[ http_config: <http_config> | default = global.http_config ]
```

## `<victorops_config>`

VictorOps notifications are sent out via the [VictorOps API](https://help.victorops.com/knowledge-base/victorops-restendpoint-integration/)

```yaml
# Whether or not to notify about resolved alerts.
[ send_resolved: <boolean> | default = true ]

# The API key to use when talking to the VictorOps API.
[ api_key: <secret> | default = global.victorops_api_key ]

# The VictorOps API URL.
[ api_url: <string> | default = global.victorops_api_url ]

# A key used to map the alert to a team.
routing_key: <tmpl_string>

# Describes the behavior of the alert (CRITICAL, WARNING, INFO).
[ message_type: <tmpl_string> | default = 'CRITICAL' ]

# Contains summary of the alerted problem.
[ entity_display_name: <tmpl_string> | default = '{{ template "victorops.default.entity_display_name" . }}' ]

# Contains long explanation of the alerted problem.
[ state_message: <tmpl_string> | default = '{{ template "victorops.default.state_message" . }}' ]

# The monitoring tool the state message is from.
[ monitoring_tool: <tmpl_string> | default = '{{ template "victorops.default.monitoring_tool" . }}' ]

# The HTTP client's configuration.
[ http_config: <http_config> | default = global.http_config ]
```

## `<webhook_config>`

webhookレシーバーによって、一般的なレシーバーを設定できる。

```yaml
# 解消された(resolved)アラートについて通知するかどうか
[ send_resolved: <boolean> | default = true ]

# HTTP POSTリクエストを送るエンドポイント
url: <string>

# HTTPクライアントの設定
[ http_config: <http_config> | default = global.http_config ]
```

Alertmanagerは、HTTP POSTリクエストを以下のJSONのフォーマットで、設定されたエンドポイントに送信する。

```
{
  "version": "4",
  "groupKey": <string>,    // アラートのグループを識別するキー（重複削除のためのもの）
  "status": "<resolved|firing>",
  "receiver": <string>,
  "groupLabels": <object>,
  "commonLabels": <object>,
  "commonAnnotations": <object>,
  "externalURL": <string>,  // Alertmanagerへのバックリンク
  "alerts": [
    {
      "status": "<resolved|firing>",
      "labels": <object>,
      "annotations": <object>,
      "startsAt": "<rfc3339>",
      "endsAt": "<rfc3339>",
      "generatorURL": <string> // アラートを起こした実体を特定する
    },
    ...
  ]
}
```

この機能を利用した[連携のリスト](/docs/operating/integrations/#alertmanager-webhook-receiver)がある。

## `<wechat_config>`

WeChat notifications are sent via the [WeChat
API](http://admin.wechat.com/wiki/index.php?title=Customer_Service_Messages).

```yaml
# Whether or not to notify about resolved alerts.
[ send_resolved: <boolean> | default = false ]

# The API key to use when talking to the WeChat API.
[ api_secret: <secret> | default = global.wechat_api_secret ]

# The WeChat API URL.
[ api_url: <string> | default = global.wechat_api_url ]

# The corp id for authentication.
[ corp_id: <string> | default = global.wechat_api_corp_id ]

# API request data as defined by the WeChat API.
[ message: <tmpl_string> | default = '{{ template "wechat.default.message" . }}' ]
[ agent_id: <string> | default = '{{ template "wechat.default.agent_id" . }}' ]
[ to_user: <string> | default = '{{ template "wechat.default.to_user" . }}' ]
[ to_party: <string> | default = '{{ template "wechat.default.to_party" . }}' ]
[ to_tag: <string> | default = '{{ template "wechat.default.to_tag" . }}' ]
```
