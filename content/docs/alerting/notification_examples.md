---
title: 通知テンプレートの例
sort_rank: 8
---
# 通知テンプレートの例

このドキュメントでは、様々なアラートの設定と対応するAlertmanagerの設定ファイル（alertmanager.yml）の例を示す。各設定で、[Go templating](http://golang.org/pkg/text/template/)を利用している。

## Slackへの通知のカスタマイズ

この例では、送信されたアラートの対処方法が書かれた組織のWikiへのURLをSlackへの通知が送るようにカスタマイズされている。

```
global:
  slack_api_url: '<slack_webhook_url>'

route:
  receiver: 'slack-notifications'
  group_by: [alertname, datacenter, app]

receivers:
- name: 'slack-notifications'
  slack_configs:
  - channel: '#alerts'
    text: 'https://internal.myorg.net/wiki/alerts/{{ .GroupLabels.app }}/{{ .GroupLabels.alertname }}'
```

## CommonAnnotationsのアノテーションの取得

この例では、Slackレシーバーに送られるテキストを、Alertmanagerが送るデータ`CommonAnnotations`に入っている`summary`と`description`を取得してカスタマイズする。

アラート

```
groups:
- name: Instances
  rules:
  - alert: InstanceDown
    expr: up == 0
    for: 5m
    labels:
      severity: page
    # Prometheus templates apply here in the annotation and label fields of the alert.
    annotations:
      description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes.'
      summary: 'Instance {{ $labels.instance }} down'
```

レシーバー

```
- name: 'team-x'
  slack_configs:
  - channel: '#alerts'
    # Alertmanager templates apply here.
    text: "<!channel> \nsummary: {{ .CommonAnnotations.summary }}\ndescription: {{ .CommonAnnotations.description }}"
```

## Ranging over all received Alerts

直前の例と同じアラートを仮定して、Alertmanagerから受け取る全てのアラートを列挙するようにカスタマイズする。

レシーバー

```
- name: 'default-receiver'
  slack_configs:
  - channel: '#alerts'
    title: "{{ range .Alerts }}{{ .Annotations.summary }}\n{{ end }}"
    text: "{{ range .Alerts }}{{ .Annotations.description }}\n{{ end }}"
```

## 再利用可能なテンプレートの定義

1つ目の例に戻ると、複数行に渡る複雑なテンプレートを避けるために、名前付きのテンプレートを含み、Alertmanagerに読み込まれるテンプレートを含むファイルを提供することが出来る。
`/alertmanager/template/myorg.tmpl`というファイルを作り、"slack.myorg.text"という名前のテンプレートを作成する。

```
{{ define "slack.myorg.text" }}https://internal.myorg.net/wiki/alerts/{{ .GroupLabels.app }}/{{ .GroupLabels.alertname }}{{ end}}
```

設定ファイルでは、カスタムテンプレートファイルのパスを指定し、"text"フィールドのために上記のテンプレートを読み込む。

```
global:
  slack_api_url: '<slack_webhook_url>'

route:
  receiver: 'slack-notifications'
  group_by: [alertname, datacenter, app]

receivers:
- name: 'slack-notifications'
  slack_configs:
  - channel: '#alerts'
    text: '{{ template "slack.myorg.text" . }}'

templates:
- '/etc/alertmanager/templates/myorg.tmpl'
```

この例は、この[ブログポスト](https://prometheus.io/blog/2016/03/03/custom-alertmanager-templates/)でさらに詳細に説明されている。
