---
title: クライアント
sort_rank: 6
nav_icon: sliders
---

# アラートの送信

**免責事項**: Prometheusは、アラートルールで生成されたアラートを自動的に送信する。 直接クライアントを実装するのではなく、時系列データに基づいた[アラートルール](../../prometheus/latest/configuration/alerting_rules/)をPrometheusに設定することを強くお勧めする。

Alertmanagerは、エンドポイント`/api/v1/alerts`でアラートをリッスンしている。 クライアントは、アラートがactiveである限りアラートを再送信し続ける（普通は大体30秒から3分）と想定されている。 クライアントは、このエンドポイントに以下の形式のPOSTリクエストでアラートのリストをプッシュすることができる。

```
[
  {
    "labels": {
      "alertname": "<requiredAlertName>",
      "<labelname>": "<labelvalue>",
      ...
    },
    "annotations": {
      "<labelname>": "<labelvalue>",
    },
    "startsAt": "<rfc3339>",
    "endsAt": "<rfc3339>",
    "generatorURL": "<generator_url>"
  },
  ...
]
```
ラベルは、同一のアラートを特定し、重複削除を行うために利用される。 アノテーションは、アラートを特定することはなく、常に一番新しく受信した値にセットされる。

タイムスタンプはどちらも必須ではない。 `startsAt`が省略されると、Alertmanagerによって現在時刻が割り振られる。 `endsAt`はアラートの終了時刻がわかっている場合のみセットされる。そうでなければ、アラートが最後に受信された時間からのタイムアウト期間に設定される。

`generatorURL`は、クライアントの中でこのアラートを起こした実体を特定するユニークなバックリンクである。
