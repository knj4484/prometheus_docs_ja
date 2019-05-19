---
title: cAdvisorを用いたDockerコンテナメトリクスの監視
---

# cAdvisorを用いたDockerコンテナメトリクスの監視

[cAdvisor](https://github.com/google/cadvisor)(**c**ontainer **Advisor**の略)は、稼働中のコンテナのリソース消費とパフォーマンスのデータを分析しexposeする。
cAdvisorは、初期状態で、Prometheusのメトリクスを出力する。
このガイドでは、

* Prometheus、cAdvisor、[Redis](https://redis.io/)サーバーのそれぞれのコンテナを含む一つの[Docker Compose](https://docs.docker.com/compose/)の設定を作成する
* Redisコンテナが生成、cAdvisorが収集、Prometheusがscrapeするいくつかのコンテナメトリクスを調査する

## Prometheusの設定

まず、cAdvisorからメトリクスをscrapeするように[Prometheusを設定する](/docs/prometheus/latest/configuration/configuration)必要がある。`prometheus.yml`ファイルを作成し、この設定のようにする。

```yaml
scrape_configs:
- job_name: cadvisor
  scrape_interval: 5s
  static_configs:
  - targets:
    - cadvisor:8080
```

## Docker Composeの設定

どのポートをexposeするか、どのボリュームを使うかなどに加えて、どのコンテナがインストールの一部かを指定するDocker Compose [configuration](https://docs.docker.com/compose/compose-file/)が必要となる。

[`prometheus.yml`](#prometheus-configuration)を作成したのと同じフォルダで、`docker-compose.yml`を作成し、内容をこのDocker Compose の設定のようにする。

```yaml
version: '3.2'
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
    - 9090:9090
    command:
    - --config.file=/etc/prometheus/prometheus.yml
    volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
    depends_on:
    - cadvisor
  cadvisor:
    image: google/cadvisor:latest
    container_name: cadvisor
    ports:
    - 8080:8080
    volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:rw
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
    depends_on:
    - redis
  redis:
    image: redis:latest
    container_name: redis
    ports:
    - 6379:6379
```

この設定は、Docker Composeに三つのサービスを実行するように指示する。それぞれの以下の[Docker](https://docker.com)コンテナに対応する。

1. `prometheus`サービスは、ローカルの設定ファイル`prometheus.yml`を利用する（`volumes`パラメーターによってコンテナにインポートされる）
2. `cadvisor`サービスは、8080ポート（cAdvisorメトリクスのデフォルトポート）をexposeし、ローカルのボリューム（`/`や`/var/run`など）に依存している
3. `redis`サービスは、標準的なRedisサーバーで、cAdvisorがこのコンテナからコンテナのメトリクスを自動的に（つまり、追加の設定をしなくても）取得する

この設定でインストールをするには下記のコマンドを実行する

```bash
docker-compose up
```

Docker Composeが三つのコンテナ全ての軌道に成功すると、以下のように出力されるはずである。

```
prometheus  | level=info ts=2018-07-12T22:02:40.5195272Z caller=main.go:500 msg="Server is ready to receive web requests."
```

[`ps`](https://docs.docker.com/compose/reference/ps/)コマンドを使って、三つのコンテナ全てが動いていることが確認できる。

```bash
docker-compose ps
```

出力は、以下のようになるだろう。

```
   Name                 Command               State           Ports
----------------------------------------------------------------------------
cadvisor     /usr/bin/cadvisor -logtostderr   Up      8080/tcp
prometheus   /bin/prometheus --config.f ...   Up      0.0.0.0:9090->9090/tcp
redis        docker-entrypoint.sh redis ...   Up      0.0.0.0:6379->6379/tcp
```

## cAdvisorのWeb UIの調査

cAdvisorのweb UIは、`http://localhost:8080`で見ることが出来る。特定のコンテナの統計やグラフは、`http://localhost:8080/docker/<container>`で調べることが出来る。
例えば、Redisコンテナのメトリクスは`http://localhost:8080/docker/redis`で、Prometheusのメトリクスは`http://localhost:8080/docker/prometheus`で見ることが出来る。

## expressionブラウザでのメトリクスの調査

cAdvisorのweb UIは、cAdvisorが監視しているものを調査するには便利だが、コンテナのメトリクスの調査のためのインターフェースを提供していない。
そのためには、`http://localhost:9090/graph`で見られるPrometheusの[expressionブラウザ](/docs/visualization/browser)が必要である。
以下のようなPrometheusのexpressionバーにexpressionを入力することが出来る。

![Prometheus expression bar](/assets/prometheus-expression-bar.png)

まず、`container_start_time_seconds`というメトリクス（コンテナの開始時間で単位は秒）から始める。
`name="<container_name>"`というexpressionで名前を指定してコンテナを選択できる。コンテナ名は、Docker Composeの設定の`container_name`と対応している。
したがって、例えば、[`container_start_time_seconds{name="redis"}`](http://localhost:9090/graph?g0.range_input=1h&g0.expr=container_start_time_seconds%7Bname%3D%22redis%22%7D&g0.tab=1)というexpressionで、`redis`コンテナの開始時間を表示することが出来る。

NOTE: cAdvisorが収集するコンテナのメトリクスの完全な一覧は、[cAdvisorのドキュメント](https://github.com/google/cadvisor/blob/master/docs/storage/prometheus.md)で見ることが出来る。

## その他のexpression

以下の表にその他のexpressionの例をいくつか示す

Expression | Description | For
:----------|:------------|:---
[`rate(container_cpu_usage_seconds_total{name="redis"}[1m])`](http://localhost:9090/graph?g0.range_input=1h&g0.expr=rate(container_cpu_usage_seconds_total%7Bname%3D%22redis%22%7D%5B1m%5D)&g0.tab=1) | [cgroup](https://en.wikipedia.org/wiki/Cgroups)の最後の1分のコア単位のCPU使用率 | `redis`コンテナ
[`container_memory_usage_bytes{name="redis"}`](http://localhost:9090/graph?g0.range_input=1h&g0.expr=container_memory_usage_bytes%7Bname%3D%22redis%22%7D&g0.tab=1) | cgroupの合計メモリ利用量（バイト） | `redis`コンテナ
[`rate(container_network_transmit_bytes_total[1m])`](http://localhost:9090/graph?g0.range_input=1h&g0.expr=rate(container_network_transmit_bytes_total%5B1m%5D)&g0.tab=1) | 最後の1分でコンテナがネットワークに送信したバイト/秒 | 全てのコンテナ
[`rate(container_network_receive_bytes_total[1m])`](http://localhost:9090/graph?g0.range_input=1h&g0.expr=rate(container_network_receive_bytes_total%5B1m%5D)&g0.tab=1) | 最後の1分でコンテナがネットワークから受信したバイト/秒 | 全てのコンテナ

## まとめ

このガイドでは、Docker Composeを使って、PrometheusコンテナがcAdvisorからメトリクスをscrapeし、cAdvisorはRedisコンテナが生成したメトリクスを収集するように、一つの設定で三つの別々のコンテナを動かした。
また、Prometheusのexpressionブラウザを使って、cAdvisorコンテナのいくつかのメトリクスを調査した。
