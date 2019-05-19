---
title: レコーディングルール
sort_rank: 6
---

# レコーディングルール

[レコーディングルール](/docs/prometheus/latest/configuration/recording_rules/)の一貫した命名体系によって、ルールの意味を一目で理解することが簡単になる。
また、不正確だったり意味のない計算が目立つようになり、間違いを避けることができる。

このページは、集約を正しく行うにはどうすればいいかを記し、また命名規約を提示する。

## 命名と集約

レコーディングルールは、一般的な形式`level:metric:operations`になっているべきである。
`level`は、集約レベルおよび出力されるラベルを表す。
`metric`は、メトリック名で、`rate()`や`irate()`を利用した時に`_total`を削除する以外は、変更するべきではない。
`operations`は、メトリックに適用された演算のリストで、もっとも新しい演算が一番先頭に来る。

メトリック名を変更せずにしておくことで、メトリックが何か知ることが簡単になり、コードベースでの検索も簡単になる。

operationsを綺麗に保つために、`sum()`以外の演算がある場合は`_sum`を省略する。
結合性のある演算はまとめて良い（例えば、`min_min`は`min`と同じである）。

利用すべき明らかな演算がない場合、`sum`を使う。
割り算をして比を取る場合、メトリクス名を`_per_`で区切り、演算は`ratio`とする。

比を集約する場合、分母と分子を別々に集約した後、割り算をすること。
統計的に妥当ではないので、比の平均を取ったり、平均の平均をとったりしないように。

観測値の平均を計算するために、サマリーの`_count`や`_sum`を集約し、割り算をする際に、それを比として扱うのは不恰好であろう。
代わりに、`_count`や`_sum`という接尾辞をなくして、`rate`を`mean`で置き換えること。
これで、その時間幅の平均的な観測値を表している。

集約後に消したいラベルを`without`で常に指定すること。
これは、`job`などの他の全てのラベルを保持することになり、コンフリクトを回避し、より有益なメトリクスやアラートが得られる。

## 例

ラベル`path`を持つ秒間リクエストを集約する

```
- record: instance_path:requests:rate5m
  expr: rate(requests_total{job="myjob"}[5m])

- record: path:requests:rate5m
  expr: sum without (instance)(instance_path:requests:rate5m{job="myjob"})
```

リクエストと失敗の比を計算し、jobレベルの失敗の比に集約する。

```
- record: instance_path:request_failures:rate5m
  expr: rate(request_failures_total{job="myjob"}[5m])

- record: instance_path:request_failures_per_requests:ratio_rate5m
  expr: |
    (
        instance_path:request_failures:rate5m{job="myjob"}
      /
        instance_path:requests:rate5m{job="myjob"}
    )

# 分母と分子を集約し、pathレベルの比を得るために割り算する
- record: path:request_failures_per_requests:ratio_rate5m
  expr: |
    (
        sum without (instance)(instance_path:request_failures:rate5m{job="myjob"})
      /
        sum without (instance)(instance_path:requests:rate5m{job="myjob"})
    )

# メトリクス実装によって付与されたりインスタンス識別のためのラベルは残っていないので、
# レベルとして`job`を使う
- record: job:request_failures_per_requests:ratio_rate5m
  expr: |
    (
        sum without (instance, path)(instance_path:request_failures:rate5m{job="myjob"})
      /
        sum without (instance, path)(instance_path:requests:rate5m{job="myjob"})
    )
```

ある時間間隔の平均レイテンシーをサマリーから計算する

```
- record: instance_path:request_latency_seconds_count:rate5m
  expr: rate(request_latency_seconds_count{job="myjob"}[5m])

- record: instance_path:request_latency_seconds_sum:rate5m
  expr: rate(request_latency_seconds_sum{job="myjob"}[5m])

- record: instance_path:request_latency_seconds:mean5m
  expr: |
    (
        instance_path:request_latency_seconds_sum:rate5m{job="myjob"}
      /
        instance_path:request_latency_seconds_count:rate5m{job="myjob"}
    )

# 分母と分子を集約した後、割り算する
- record: path:request_latency_seconds:mean5m
  expr: |
    (
        sum without (instance)(instance_path:request_latency_seconds_sum:rate5m{job="myjob"})
      /
        sum without (instance)(instance_path:request_latency_seconds_count:rate5m{job="myjob"})
    )
```

関数`avg()`を使って、instanceとpathにまたがって平均クエリーレートが計算される

```
- record: job:request_latency_seconds_count:avg_rate5m
  expr: avg without (instance, path)(instance:request_latency_seconds_count:rate5m{job="myjob"})
```

入力されるメトリック名と比べると、`without`で指定されたラベルが出力されるメトリック名のレベルから消えていることに注意。
集約がない場合は、レベルが必ず同じになる。
そうでない場合は、ルールの中に間違いがある可能性が高い。
