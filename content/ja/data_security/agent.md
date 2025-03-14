---
aliases:
- /ja/agent/security/
description: Datadog Agent のセキュリティ対策
further_reading:
- link: /data_security/
  tag: ドキュメント
  text: Datadog に送信されるデータの主要カテゴリを確認する
title: Agent のデータセキュリティ
---

<div class="alert alert-info">このページでは、Datadog に送信されるデータのセキュリティについて説明します。クラウドやアプリケーションのセキュリティ製品や機能をお探しの場合は、<a href="/security/" target="_blank">セキュリティ</a>のセクションをご覧ください。</div>

ローカルにインストールされた [Agent][1] または [HTTP API][2] を使用して Datadog サービスにデータを送信できます。Datadog を使用する上で Datadog Agent を必ず使用しなければならないわけではありませんが、大半のお客様が Agent を利用しています。ここでは、安全な環境を維持するために利用できる主なセキュリティ機能について説明します。

## Agent ディストリビューション

Agent の公式リポジトリおよびバイナリパッケージは署名されています。次の公開鍵のいずれかに対して署名を確認して、ディストリビューションチャンネルを確認します。

- Linux DEB パッケージとリポジトリメタデータ:
  - [D18886567EABAD8B2D2526900D826EB906462314][22]
  - [5F1E256061D813B125E156E8E6266D4AC0962C7D][19]
  - [D75CEA17048B9ACBF186794B32637D44F14F620E][4]
  - [A2923DFF56EDA6E76E55E492D3A80E30382E94DE][3]
- Linux RPM パッケージとリポジトリメタデータ:
  - [2416A37757B1BB0268B3634B52AFC5994F09D16B][21]
  - [7408BFD56BC5BF0C361AAAE85D88EEA3B01082D3][20]
  - [C6559B690CA882F023BDF3F63F4D1729FD4BF915][5]
  - [A4C0B90D7443CF6E4E8AA341F1068E14E09422B3][6]
- MacOS PKG:
  - Apple certificate fingerprint `FDD2ADF623EA75E62C6DC6DBFBA7520CA549AB7314E660D78B0E3DCCF15B2FBA`

Debian および Ubuntu では、`datadog-agent` パッケージに `datadog-signing-keys` パッケージのソフト依存性があるため、上記のキーが APT により信頼されます。パッケージを最新に更新しておくと、最新の署名キーをシステムに保つことができます。

### Windows MSI

Windows 上で Datadog Agent インストーラファイルの署名を検証するには、`Get-AuthenticodeSignature` の出力を `FormatList` (`fl`) にパイプし、次のことを確認します。
- ステータスが有効であること
- 証明書が `Datadog, Inc` によって署名されていること
- 発行者が `DigiCert` であること

例えば、`ddagent-cli-7.49.1.msi` という名前の .msi ファイルを検証する場合:
{{< code-block lang="powershell" >}}
Get-AuthenticodeSignature ddagent-cli-7.49.1.msi | fl
{{< /code-block >}}

コマンドの出力が `A certificate chain could not be built to a trusted root authority` (信頼できる root 認証局への証明書チェーンを構築できなかった) の場合、そのマシンは DigiCert root CA の更新が必要になるかもしれません。

## 情報セキュリティ

Datadog Agent は、デフォルトで、TLS で暗号化された TCP 接続を介して Datadog にデータを送信します。バージョン 6 では、Datadog への接続時に最小の TLS バージョン強制するように Agent を設定できます。たとえば、PCI の要件を満たすために、"強力な暗号化" を使用する必要がある場合は、Agent v6/7 を使用し、Agent の構成ファイルで `min_tls_version: 'tlsv1.2'`、または Agent < 6.39.0/7.39.0 では `force_tls_12: true` を設定をする必要があります。

## ネットワークとプロキシ

Datadog は SaaS 製品です。監視データを送信するには、お客様のネットワークから公共のインターネットへのアウトバウンド接続を確立する必要があります。トラフィックは常に Agent から Datadog へと開始され、デフォルトでは、TLS で暗号化された TCP 接続で行われます。Datadog から Agent へのセッションが開始されることはありません。必要な Datadog のドメインとポートをリストアップする方法については、Agent の[ネットワーク][7]のページを参照してください。さらに、公共のインターネットに直接接続していないホスト、またはアウトバウンドトラフィックが制限されているホストを監視する場合は、監視データを[プロキシ][8]から送信することを検討してください。

## Agent ログの難読化

Datadog Agent は、[Agent のトラブルシューティング][9]をサポートするため、必要に応じてローカルにログを生成します。安全対策として、これらのローカルログは、何らかの資格情報 (API キー、パスワード、トークンキーワードなど) を表している可能性があるいくつかのキーワードとパターンを使用してフィルターされた後、難読化されてからディスクに書き込まれます。

## ローカル HTTPS サーバー

Agent v6/7 は、実行中の Agent と Agent ツール (たとえば、`datadog-agent` コマンド) の間の通信を容易にするために、ローカル HTTPS API を公開しています。API サーバーは、ローカルネットワークインターフェイス (`localhost/127.0.0.1`) からのみアクセスすることができ、Agent を実行しているユーザーだけが読み取ることができるトークンを使用して認証が強制されます。ローカル HTTPS API への通信は、`localhost` 上での傍受から保護するため、転送時に暗号化されます。

## Agent GUI

Agent v6/7 には、デフォルトでグラフィカルユーザーインターフェイス (GUI) が付属しており、デフォルトの Web ブラウザで起動します。この GUI は、GUI を起動するユーザーが、Agent の構成ファイルを開く権限も含めて、正しいユーザーアクセス許可を持つ場合にのみ起動されます。GUI には、ローカルネットワークインターフェイス (`localhost/127.0.0.1`) からのみアクセスできます。最後に、GUI は、GUI サーバーとのすべての通信を認証し、クロスサイトリクエストフォージェリ (CSRF) 攻撃から防御するために使用するトークンを生成して保存するため、ユーザーの cookie を有効にする必要があります。必要に応じて、GUI を完全に無効にすることもできます。

## Agent のセキュリティスキャン

Datadog の脆弱性管理プログラムには、サポート用インフラストラクチャー/アプリケーションコンポーネントの定期評価が含まれています。たとえば、サポート用コアサービスのアクティブスキャンを行います。Datadog のセキュリティチームは、Datadog の脆弱性管理ポリシーに従って月 1 回のスキャンを実施して、構成やソフトウェアの脆弱性を特定し、見つかった脆弱性の修正状況を追跡します。

特に Container Agent に関しては、Datadog は [CoreOS の Clair][10] と [snyk.io][11] を使用して定期的に脆弱性統計情報の分析を実施しています。さらに、Container Agent を [Docker Trusted Registry][12] や [Red Hat Container Catalog][13] にリリースする際にセキュリティスキャンを利用しています。Datadog の社内脆弱性管理プログラムに加えて、Datadog はコンテナセキュリティベンダーとも連携しています。

Detadog のセキュリティにバグを発見された場合は、[security@datadoghq.com][14] にご連絡ください。24 時間以内に対応いたします。Datadog との通信を暗号化する必要がある場合は、Datadog の [PGP キー][15]をダウンロードできます。発見した問題が解決されるまで、問題を公表されないようにお願いいたします。

## 非特権ユーザーとして実行

デフォルトでは、Agent は Linux では `dd-agent` ユーザーとして、[Windows][16] では `ddagentuser` アカウントとして実行されます。例外は以下のとおりです。

- `system-probe` が Linux で `root` として、Windows で `LOCAL_SYSTEM` として実行。
- `process-agent` が Windows で `LOCAL_SYSTEM` として実行。
- `security-agent` が Linux で `root` として実行。

## 機密情報管理

Agent の構成ファイルに機密情報がプレーンテキストで格納されることを避ける必要がある場合は、[機密情報管理][17]パッケージを利用できます。このパッケージを使用すると、Agent は、ユーザー提供の実行プログラムを呼び出して機密情報の取得や解読を処理してから、処理した機密情報をメモリにロードします。任意のキー管理サービス、認証方法、継続的インテグレーションワークフローを使用して、実行プログラムを設計できます。

詳細については、[秘密情報管理][18]のドキュメントを参照してください。

## テレメトリーの収集

{{< site-region region="gov" >}}

非政府サイト上の Agent は、Datadog Agent に関する環境情報、パフォーマンス情報、および機能使用状況の情報を収集します。Agent が政府サイトを検出した場合、または [Datadog Agent FIPS Proxy][1] が使用されている場合、Agent はこのテレメトリー収集を自動的に無効化します。そのような検出が不可能な場合 (例えば、プロキシを使用している場合)、Agent のテレメトリーは送信されますが、Datadog のインテークで即座に破棄されます。そもそもこのデータが送信されること自体を避けるために、Datadog は Agent コンフィギュレーションファイルで `agent_telemetry` 設定を更新し、明示的に Agent のテレメトリーを無効化することを推奨しています。以下はその例です。

{{< tabs >}}
{{% tab "datadog.yaml" %}}

```yaml
agent_telemetry:
  enabled: false
```
{{% /tab %}}
{{% tab "環境変数" %}}

```bash
DD_AGENT_TELEMETRY_ENABLED=false
```
{{% /tab %}}
{{< /tabs >}}
[1]: https://docs.datadoghq.com/ja/agent/configuration/agent-fips-proxy/?tab=hostorvm&site=gov
{{< /site-region >}}
{{< site-region region="us,us3,us5,eu,ap1" >}}
Datadog は、Datadog Agent に関する環境情報、パフォーマンス情報、および機能使用状況の情報を収集する場合があります。これには、Datadog Agent の診断ログや、難読化されたスタックトレースを含むクラッシュダンプが含まれ、Datadog Agent のサポートおよびさらなる改善に役立ちます。

このテレメトリー収集は、以下の例のように Agent コンフィギュレーションファイルで `agent_telemetry` 設定を更新することで無効化できます。
{{< tabs >}}
{{% tab "datadog.yaml" %}}

```yaml
agent_telemetry:
  enabled: false
```
{{% /tab %}}
{{% tab "環境変数" %}}

```bash
DD_AGENT_TELEMETRY_ENABLED=false
```
{{% /tab %}}
{{< /tabs >}}

**テレメトリー内容:**
| メタデータ ([source][1]) |
| ------------------------ |
| マシン ID |
| マシン名 |
| OS |
| OS バージョン |
| Agent バージョン |

| メトリクス ([source][2])                       | 説明                                                                                       |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| checks.execution_time                       | チェックの実行時間 (ミリ秒単位)                                                            |
| pymem.inuse                                 | Python インタープリタによって割り当てられているバイト数                                               |
| pymem.alloc                                 | Agent 起動以来、Python インタープリタによって割り当てられた累計バイト数            |
| api_server.request_duration_seconds         | CLI コマンド実行性能 (実行された場合)                                                  |
| logs.decoded                                | デコードされたログの総数                                                                      |
| logs.sender_latency                         | HTTP sender latency in millisecondsHTTP 送信者の遅延時間 (ミリ秒単位)                                                               |
| logs.bytes_missed                           | Agent が消費される前に失われたバイト数の総計 (ログローテーション後など) |
| logs.dropped                                | ドロップされたログの総数                                                                      |
| logs.bytes_sent                             | エンコード前に送信されたバイト数の総計 (該当する場合)                                                |
| logs.encoded_bytes_sent                     | エンコード後に送信されたバイト数の総計 (該当する場合)                                                 |
| dogstatsd.udp_packets                       | DogStatsD UDP パケットのバイト数                                                                       |
| dogstatsd.uds_packets                       | DogStatsD UDS パケットのバイト数                                                                       |
| transactions.input_count                    | 受信トランザクション数                                                                        |
| transactions.requeued                       | 再キューイングされたトランザクション数                                                                         |
| transactions.retries                        | 再試行されたトランザクション数                                                                           |
| point.sent                                  | 送信されたメトリクスの総数                                                                      |
| point.dropped                               | ドロップされたメトリクスの総数                                                                   |
| oracle.activity_samples_count               | クエリアクティビティを計測する際に取得した行数 (収集されたアクティビティサンプル数)         |
| oracle.activity_latency                     | Time to retrieve query activity in millisecondsクエリアクティビティ取得時間 (ミリ秒単位)                                                   |
| oracle.statement_metrics                    | データベースメトリクス取得時間 (ミリ秒単位)                                                 |
| oracle.statement_plan_errors                | 実行計画取得時のエラー数                                                    |
| postgres.collect_relations_autodiscovery_ms | リレーションのオートディスカバリー収集時間 (ミリ秒単位)                                          |
| postgres.collect_stat_autodiscovery_ms      | オートディスカバリー統計の収集時間 (ミリ秒単位)                                               |
| postgres.get_new_pg_stat_activity_ms        | `pg_stat_activity` 取得時間 (ミリ秒単位)                                                    |
| postgres.get_new_pg_stat_activity_count     | `pg_stat_activity` 収集のために取得された総行数                                                  |
| postgres.get_active_connections_ms          | アクティブな接続取得時間 (ミリ秒単位)                                                    |
| postgres.get_active_connections_count       | アクティブな接続取得のために取得された総行数                                                      |
| postgres.collect_activity_snapshot_ms       | アクティビティスナップショット取得時間 (ミリ秒単位)                                                     |
| postgres.collect_statement_samples_ms       | ステートメントサンプル取得時間 (ミリ秒単位)                                                     |
| postgres.collect_statement_samples_count    | ステートメントサンプル収集のために取得された総行数                                                   |

適用可能なメトリクスのみが送信されます。例えば、DBM が有効でない場合、データベース関連のメトリクスは一切送信されません。


[1]: https://github.com/DataDog/datadog-agent/blob/4dc6ed6eb069bdea7e93f2d267ac5086a98c968c/comp/core/agenttelemetry/impl/sender.go#L218-L221
[2]: https://github.com/DataDog/datadog-agent/blob/4dc6ed6eb069bdea7e93f2d267ac5086a98c968c/comp/core/agenttelemetry/impl/config.go#L156

{{< /site-region >}}

### その他の参考資料

{{< partial name="whats-next/whats-next.html" >}}

[1]: /ja/agent/
[2]: /ja/api/
[3]: https://keys.datadoghq.com/DATADOG_APT_KEY_382E94DE.public
[4]: https://keys.datadoghq.com/DATADOG_APT_KEY_F14F620E.public
[5]: https://keys.datadoghq.com/DATADOG_RPM_KEY_FD4BF915.public
[6]: https://keys.datadoghq.com/DATADOG_RPM_KEY_E09422B3.public
[7]: /ja/agent/faq/network/
[8]: /ja/agent/configuration/proxy/
[9]: /ja/agent/troubleshooting/
[10]: https://coreos.com/clair
[11]: https://snyk.io
[12]: https://docs.docker.com/v17.09/datacenter/dtr/2.4/guides
[13]: https://access.redhat.com/containers
[14]: mailto:security@datadoghq.com
[15]: https://www.datadoghq.com/8869756E.asc.txt
[16]: /ja/agent/faq/windows-agent-ddagent-user/
[17]: https://github.com/DataDog/datadog-agent/blob/master/docs/agent/secrets.md
[18]: /ja/agent/configuration/secrets-management/
[19]: https://keys.datadoghq.com/DATADOG_APT_KEY_C0962C7D.public
[20]: https://keys.datadoghq.com/DATADOG_RPM_KEY_B01082D3.public
[21]: https://keys.datadoghq.com/DATADOG_RPM_KEY_4F09D16B.public
[22]: https://keys.datadoghq.com/DATADOG_APT_KEY_06462314.public