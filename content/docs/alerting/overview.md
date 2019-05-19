---
title: アラート概要
sort_rank: 1
nav_icon: sliders
---

# アラート概要

Prometheusのアラートは、二つの部分に分けられている。 Prometheusサーバーのアラートルールは、Alertmanagerにアラートを送る。 [Alertmanager](../alertmanager)は、アラートを一時停止したり、アラートの依存関係を元に通知を抑制したり、集約したり、eメールやPagerDuty、HipChatなどの方法での通知の送信などを含むそれらアラートの管理を行う。

アラートと通知をセットアップする主なステップは以下の通り。

* Alertmanagerのセットアップと[設定](../configuration)をする
* Alertmanagerと通信するように[Prometheusを設定](../../prometheus/latest/configuration/configuration/#<alertmanager_config>)する
* Prometheusの[アラートルール](../../prometheus/latest/configuration/alerting_rules/)を作成する
