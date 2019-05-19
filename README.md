# Prometheusドキュメント和訳

このリポジトリは、[Prometheusドキュメント](https://github.com/prometheus/docs)の和訳で、コンテンツと静的なサイトを生成するコードの両方が含まれている。

## Contributing Changes

新しくこのリポジトリに貢献したい場合の一般的な手順については[`CONTRIBUTING.md`](CONTRIBUTING.md)を参照すること。

[`content/docs`](content/docs)に主なドキュメントの内容が置かれている。

Prometheusサーバーに関するドキュメントの和訳は、[別のリポジトリ](https://github.com/knj4484/prometheus/tree/release-ja-2.8/docs)で管理されている。

ガイドラインとして、このドキュメントを一般的に適用できるように保ち、ユースケースに特有の変更は避けること。

## Prerequisites

[bundler](https://bundler.io/)を含むRuby環境を準備した上で、以下のコマンドで必要なgemをインストールする。

```bash
cd prometheus_docs_ja
make bundle
```

## Building

静的なサイトを生成するために以下のコマンドを実行する。

```bash
make build
```

生成された静的なサイトは`output`ディレクトリに保存される。

APIのレート制限を避けるためにAPIトークンを利用することも出来る。APIトークンは https://github.com/settings/tokens/new から取得できる。
```bash
export GITHUB_AUTHENTICATION='-u user:token'
```

## Development Server

生成されたサイトをローカルのサーバーで表示するには、以下のコマンドを実行する。

```bash
# 関連ファイルが変更されたらサイトがリビルドされるようにする
make guard
# 別のシェルでローカル開発サーバーを起動する
make serve
```

これで生成されたサイトが[http://localhost:3000/docs/introduction/overview/](http://localhost:3000/docs/introduction/overview/)で閲覧できるはずである。

## Automatic Deployment

This site is automatically deployed using [Netlify](https://www.netlify.com/).

If you have the prerequisite access rights, you can view the Netlify settings here:

* GitHub webhook notifying Netlify of branch changes: https://github.com/prometheus/docs/settings/hooks
* Netlify project: https://app.netlify.com/sites/prometheus-docs

Changes to the `master` branch are deployed to the main site at https://prometheus.io.

Netlify also creates preview deploys for every pull request. To view these for a PR where all checks have passed:

1. In the CI section of the PR, click on "Show all checks".
2. On the "deploy/netlify" entry, click on "Details" to view the preview site for the PR.

You may have to wait a while for the "deploy/netlify" check to appear after creating or updating the PR, even if the other checks have already passed.

## License

Apache License 2.0, see [LICENSE](LICENSE).
