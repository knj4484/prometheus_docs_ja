# Blackbox exporterの設定

設定ファイルは、以下に記すスキームで定義された[YAML形式](https://ja.wikipedia.org/wiki/YAML)で書かれる。 
ブラケットは、パラメーターがオプションであることを表す。
リストでないパラメーターは、指定されたデフォルト値にセットされる。

一般的なプレースホルダーは以下の通り。

* `<boolean>`: 値として`true`または`false`を取ることが出来るブーリアン
* `<int>`: a regular integer
* `<duration>`: 正規表現`[0-9]+(ms|[smhdwy])`にマッチする時間幅
* `<filename>`: a valid path in the current working directory
* `<string>`: 文字列
* `<secret>`: パスワードのような秘密の文字列
* `<regex>`: 正規表現

そのほかのプレースホルダーは、個別に定義される。

### Module
```yml

  # プローブで使うプロトコル（http、tcp、dns、icmp）
  prober: <prober_string>

  # プローブを諦めるまでの時間
  [ timeout: <duration> ]

  # The specific probe configuration - at most one of these should be specified.
  # 個別のプローブの設定。
  [ http: <http_probe> ]
  [ tcp: <tcp_probe> ]
  [ dns: <dns_probe> ]
  [ icmp: <icmp_probe> ]

```

### <http_probe>
```yml

  # このプローブで受容するステータスコード。デフォルトは2xx
  [ valid_status_codes: <int>, ... | default = 2xx ]

  このプローブで受容するHTTPバージョン
  [ valid_http_versions: <string>, ... ]

  # プローブが使うHTTPメソッド
  [ method: <string> | default = "GET" ]

  # プローブのためのHTTPヘッダー
  headers:
    [ <string>: <string> ... ]

  # プローブがリダイレクトに従うかどうか
  [ no_follow_redirects: <boolean> | default = false ]

  # SSLが提示されたらプローブを失敗させる
  [ fail_if_ssl: <boolean> | default = false ]

  # SSLが提示されなかったら失敗させる
  [ fail_if_not_ssl: <boolean> | default = false ]

  # レスポンスボディが正規表現にマッチしたらプローブを失敗させる
  fail_if_body_matches_regexp:
    [ - <regex>, ... ]

  # レスポンスボディが正規表現にマッチしなかったらプローブを失敗させる
  fail_if_body_not_matches_regexp:
    [ - <regex>, ... ]

  # レスポンスヘッダーが正規表現にマッチしたらプローブを失敗させる。複数の値があるヘッダーに対しては、少なくとも1つマッチしたら失敗させる
  fail_if_header_matches:
    [ - <http_header_match_spec>, ... ]

  # レスポンスヘッダーが正規表現にマッチしなかったらプローブを失敗させる。複数の値があるヘッダーに対しては、1つもマッチしなかったら失敗させる
  fail_if_header_not_matches:
    [ - <http_header_match_spec>, ... ]

  # HTTPプローブのTLSプロトコルの設定
  tls_config:
    [ <tls_config> ]

  # 監視対象のHTTP Basic認証のクレデンシャル
  basic_auth:
    [ username: <string> ]
    [ password: <secret> ]

  # 監視対象の署名なしトークン (Bearer Token)
  [ bearer_token: <secret> ]

  # 監視対象の署名なしトークン (Bearer Token)のファイル
  [ bearer_token_file: <filename> ]

  # 監視対象に接続するために利用するプロキシサーバー
  [ proxy_url: <string> ]

  HTTPプローブのIPプロトコル（ip4、ip6）
  [ preferred_ip_protocol: <string> | default = "ip6" ]
  [ ip_protocol_fallback: <boolean> | default = true ]

  # プローブで使うHTTPリクエストボディ
  body: [ <string> ]


```

#### <http_header_match_spec>

```yml
header: <string>,
regexp: <regex>,
[ allow_missing: <boolean> | default = false ]
```

### <tcp_probe>

```yml

# TCPプローブのIPプロトコル（ip4、ip6）
[ preferred_ip_protocol: <string> | default = "ip6" ]
[ ip_protocol_fallback: <boolean | default = true> ]

# ソースIPアドレス
[ source_ip_address: <string> ]

# TCPプローブで送られるクエリと対応する期待されるレスポンス。
# starttlsはTCP接続をTLSにアップグレードする。
query_response:
  [ - [ [ expect: <string> ],
        [ send: <string> ],
        [ starttls: <boolean | default = false> ]
      ], ...
  ]

# 接続が開始された時にTLSが使われるかどうか
[ tls: <boolean | default = false> ]

# TCPプローブのTLSプロトコルの設定
tls_config:
  [ <tls_config> ]

```

### <dns_probe>

```yml

# DNSプローブのIPプロトコル（ip4、ip6）
[ preferred_ip_protocol: <string> | default = "ip6" ]
[ ip_protocol_fallback: <boolean | default = true> ]

# ソースIPアドレス
[ source_ip_address: <string> ]

[ transport_protocol: <string> | default = "udp" ] # udp, tcp

query_name: <string>

[ query_type: <string> | default = "ANY" ]

# 有効なレスポンスコードのリスト
valid_rcodes:
  [ - <string> ... | default = "NOERROR" ]

validate_answer_rrs:

  fail_if_matches_regexp:
    [ - <regex>, ... ]

  fail_if_not_matches_regexp:
    [ - <regex>, ... ]

validate_authority_rrs:

  fail_if_matches_regexp:
    [ - <regex>, ... ]

  fail_if_not_matches_regexp:
    [ - <regex>, ... ]

validate_additional_rrs:

  fail_if_matches_regexp:
    [ - <regex>, ... ]

  fail_if_not_matches_regexp:
    [ - <regex>, ... ]

```

### <icmp_probe>

```yml

# ICMPプローブのIPプロトコル（ip4、ip6）
[ preferred_ip_protocol: <string> | default = "ip6" ]
[ ip_protocol_fallback: <boolean | default = true> ]

# ソースIPアドレス
[ source_ip_address: <string> ]

# IPヘッダーのDFビットをセットする。ip4かつ*nixシステム上でのみ機能する
[ dont_fragment: <boolean> | default = false ]

# ペイロードのサイズ
[ payload_size: <int> ]

```

### <tls_config>

```yml

# ターゲットの証明書の検証を無効にする
[ insecure_skip_verify: <boolean> | default = false ]

# ターゲットの使われるCA証明書
[ ca_file: <filename> ]

# ターゲットのクライアント証明書ファイル
[ cert_file: <filename> ]

# ターゲットのクライアントキーファイル
[ key_file: <filename> ]

# ターゲットのホスト名を検証するために使われる
[ server_name: <string> ]

```