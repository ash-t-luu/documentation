---
aliases:
- /ja/integrations/awsbeanstalk/
- /ja/developers/faq/i-want-my-application-deployed-in-a-container-through-elasticbeanstalk-to-talk-to-dogstatsd/
categories:
- aws
- cloud
- configuration & deployment
- log collection
- network
- provisioning
custom_kind: インテグレーション
dependencies: []
description: AWS Elastic Beanstalk のキーメトリクスを追跡します。
doc_link: https://docs.datadoghq.com/integrations/amazon_elasticbeanstalk/
draft: false
further_reading:
- link: https://www.datadoghq.com/blog/deploy-datadog-aws-elastic-beanstalk
  tag: ブログ
  text: AWS Elastic Beanstalk への Datadog のデプロイ
git_integration_title: amazon_elasticbeanstalk
has_logo: true
integration_id: ''
integration_title: AWS Elastic Beanstalk
integration_version: ''
is_public: true
manifest_version: '1.0'
name: amazon_elasticbeanstalk
public_title: Datadog-AWS Elastic Beanstalk インテグレーション
short_description: AWS Elastic Beanstalk のキーメトリクスを追跡します。
version: '1.0'
---

<!--  SOURCED FROM https://github.com/DataDog/dogweb -->
## 概要

AWS Elastic Beanstalk は、Apache、Nginx、Passenger、IIS などの使い慣れたサーバーで、Java、.NET、PHP、Node.js、Python、Ruby、Go、および Docker を使用して開発された Web アプリケーションやサービスをデプロイおよびスケーリングするための使いやすいサービスです。

## セットアップ

### インストール

まだ行っていない場合は、まず [Amazon Web Services インテグレーション][1]をセットアップします。Elastic Beanstalk メトリクスを受信するには、ご使用の環境で[拡張ヘルスレポート機能を有効][2]にし、[拡張ヘルスメトリクスを CloudWatch に公開][3]するように環境を構成する必要があります。

**注**: これらの設定により、CloudWatch カスタムメトリクス料金が加算されます。

## 収集データ

### メトリクス
{{< get-metrics-from-git "amazon_elasticbeanstalk" >}}


AWS から取得される各メトリクスには、ホスト名やセキュリティ グループなど、AWS コンソールに表示されるのと同じタグが割り当てられます。

### イベント

AWS Elastic Beanstalk インテグレーションには、イベントは含まれません。

### サービスチェック

AWS Elastic Beanstalk インテグレーションには、サービスのチェック機能は含まれません。

## Datadog Agent 構成

次のステップでは、Elastic Beanstalk VM に Datadog Agent をデプロイし、AWS インテグレーションによってクロールされるメトリクスに加えて、ホストメトリクスもレポートするようにします。詳しくは、[クラウドインスタンスに Datadog Agent をインストールするメリットは何ですか？][4]をお読みください。

インストール方法を選択して、Elastic Beanstalk 環境に Agent を構成します。

{{< tabs >}}

{{% tab "コンテナなし (Linux)" %}}

コンテナなしのセットアップの場合、[コンフィギュレーションファイル (.ebextensions) による高度な環境のカスタマイズ][1]を使用して、Datadog Agent を Elastic Beanstalk にインストールします。

1. [アプリケーションソースバンドル][2]のルートに `.ebextensions` という名前のフォルダーを作成します。
2. [99datadog.config][3] をダウンロードし、`.ebextensions` フォルダに入れます。
3. `/etc/datadog-agent/datadog.yaml` のファイルテンプレート内の `api_key` の値を、お使いの [Datadog API キー][4]で変更します。
4. Agent が正しい Datadog ロケーションにデータを送信するために、`/etc/datadog-agent/datadog.yaml` の `site` の値を Datadog リージョンに変更します (例: {{&lt; region-param key="dd_site" code="true" &gt;}})。
5. すべてのホストが同じバージョンの Agent を実行するように、`option_settings` の下に `DD_AGENT_VERSION` を設定して特定の Agent バージョンを固定します。
6. [Elastic Beanstalk コンソール][5]、[EB CLI][6]、または [AWS CLI][7] でアプリケーションをデプロイします。

Agent の設定は `/etc/datadog-agent/datadog.yaml` に追加することができます。

例えば、ライブプロセスモニタリングを有効にするには

```text
process_config:
  enabled: "true"
```

#### トレースの収集

アプリケーションがコンテナ化されておらず、Datadog Agent が `99datadog.config` で構成されているとき、アプリケーションが[トレーシングライブラリセットアップ][8]でインスツルメントされている場合は、追加のコンフィギュレーションなしでトレーシングが有効になります。



[1]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ebextensions.html
[2]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/applications-sourcebundle.html
[3]: https://docs.datadoghq.com/ja/config/99datadog.config
[4]: https://app.datadoghq.com/organization-settings/api-keys
[5]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environment-configuration-methods-during.html#configuration-options-during-console-ebextensions
[6]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environment-configuration-methods-during.html#configuration-options-during-ebcli
[7]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environment-configuration-methods-during.html#configuration-options-during-awscli
[8]: https://docs.datadoghq.com/ja/tracing/setup/
{{% /tab %}}

{{% tab "コンテナなし (Windows)" %}}

コンテナなしのセットアップの場合、[コンフィギュレーションファイル (.ebextensions) による高度な環境のカスタマイズ][1]を使用して、Datadog Agent を Elastic Beanstalk にインストールします。

1. [アプリケーションソースバンドル][2]のルートに `.ebextensions` という名前のフォルダーを作成します。
2. [99datadog-windows.config][3] をダウンロードし、`.ebextensions` フォルダに移動します。サンプル構成を確認し、必要に応じて変更を加えます。
3. `99datadog-windows.config` で、`APIKEY` の値を [Datadog API キー][4]に置き換えます。
4. (オプション) 環境変数を追加する必要がある場合は、`99datadog-windows.config` の `00_setup-env1` セクションで環境変数を設定します。環境変数を設定する必要がない場合は、このセクションを削除してもかまいません。
5. (オプション) 環境で APM を有効にしない場合は、`packages.msi.DotnetAPM` セクション、`02_setup-APM1` セクション、および `03_setup-APM2` セクションを削除してください。
7. **Trace Collection with .NET APM** の場合:
    1. `packages.msi.DotnetAPM` リンクを、[dd-trace-dotnet リリースノート][5]に記載されている希望のバージョンの MSI ファイル (Windows Installer) に置き換えてください。
    2. (オプション) .NET APM の環境変数を追加する必要がある場合は、`99datadog-windows.config` の `00_setup-env1` セクションで環境変数を設定します。
8. [Elastic Beanstalk コンソール][6]、[EB CLI][7]、または [AWS CLI][8] でアプリケーションをデプロイします。


#### トレースの収集

アプリケーションがコンテナ化されておらず、Datadog Agent が `99datadog-windows.config` で構成されている場合、前のセクションで説明されている手順以上の追加構成を行うことなくトレーシングが有効になります。トレーシングのインスツルメンテーションについては、[Datadog APM のセットアップ][8]を参照してください。



[1]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ebextensions.html
[2]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/applications-sourcebundle.html
[3]: https://docs.datadoghq.com/ja/config/99datadog-windows.config
[4]: https://app.datadoghq.com/organization-settings/api-keys
[5]: https://github.com/DataDog/dd-trace-dotnet/releases
[6]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environment-configuration-methods-during.html#configuration-options-during-console-ebextensions
[7]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environment-configuration-methods-during.html#configuration-options-during-ebcli
[8]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environment-configuration-methods-during.html#configuration-options-during-awscli
[9]: https://docs.datadoghq.com/ja/tracing/setup/
{{% /tab %}}

{{% tab "単一のコンテナ" %}}

単一の Docker コンテナのセットアップの場合、[コンフィギュレーションファイル (.ebextensions) による高度な環境のカスタマイズ][1]を使用して、Datadog Agent を Elastic Beanstalk にインストールします。

**注**: このセットアップでは、API キーをソースコードの一部である .ebextensions ディレクトリに配置する必要があります。[AWS Secret Manager][2] などのシークレット管理ツールを用いて、API キーを保護してください。

1. [アプリケーションソースバンドル][3]のルートに `.ebextensions` という名前のフォルダーを作成します。
2. [99datadog.config][4] をダウンロードし、`.ebextensions` フォルダに入れます。
3. `/etc/datadog-agent/datadog.yaml` のファイルテンプレート内の `api_key` の値を、お使いの [Datadog API キー][5]で変更します。
4. Agent が正しい Datadog ロケーションにデータを送信するために、`/etc/datadog-agent/datadog.yaml` の `site` の値を Datadog リージョンに変更します (例: {{&lt; region-param key="dd_site" code="true" &gt;}})。
5. すべてのホストが同じバージョンの Agent を実行するように、`option_settings` の下に `DD_AGENT_VERSION` を設定して特定の Agent バージョンを固定します。
6. [Elastic Beanstalk コンソール][6]、[EB CLI][7]、または [AWS CLI][8] でアプリケーションをデプロイします。

Agent の設定は `/etc/datadog-agent/datadog.yaml` に追加することができます。

例えば、ライブプロセスモニタリングを有効にするには

```text
process_config:
  enabled: "true"
```

#### トレースの収集

単一の Docker コンテナのトレースを有効にするには

1. `99datadog.config` ファイルの `/etc/datadog-agent/datadog.yaml` セクションを `apm_non_local_traffic` で更新し、次のようなフォーマットにします。

    ```
    apm_config:
      enabled: "true"
      apm_non_local_traffic: "true"
    ```

2. トレーシングライブラリをセットアップして、トレースが [ブリッジネットワークの Gateway IP][9] に送られるようにします。アプリケーションコンテナ内からのデフォルトが `172.17.0.1` になります (これが Gateway IP かどうかわからない場合は、`docker inspect <container id>` を実行して確認します)。

すべての言語で、環境変数 `DD_AGENT_HOST` をゲートウェイ IP に設定します。または、以下の言語の場合、次を使用してプログラムでホスト名を設定します。

##### Python

```python
from ddtrace import tracer

tracer.configure(hostname="172.17.0.1")
```

##### Node.js

```javascript
const tracer = require('dd-trace');

tracer.init({ hostname: "172.17.0.1" });
```

##### Ruby

```ruby
require 'ddtrace'

Datadog.configure do |c|
  c.tracer hostname: "172.17.0.1")
end
```

##### Go

```go
package main

import (
    "gopkg.in/DataDog/dd-trace-go.v1/ddtrace/tracer"
)

func main() {
  tracer.Start(tracer.WithAgentAddr("172.17.0.1"))
  defer tracer.Stop()

  // ...
}
```



[1]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ebextensions.html
[2]: https://aws.amazon.com/secrets-manager/
[3]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/applications-sourcebundle.html
[4]: https://docs.datadoghq.com/ja/config/99datadog.config
[5]: https://app.datadoghq.com/organization-settings/api-keys
[6]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environment-configuration-methods-during.html#configuration-options-during-console-ebextensions
[7]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environment-configuration-methods-during.html#configuration-options-during-ebcli
[8]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environment-configuration-methods-during.html#configuration-options-during-awscli
[9]: https://docs.docker.com/network/network-tutorial-standalone/
{{% /tab %}}

{{% tab "複数のコンテナ" %}}

複数の Docker コンテナの場合、コンテナ化された Datadog Agent を使用して、`Dockerrun.aws.json` という名前のファイルで Docker の使用状況を監視します。

`Dockerrun.aws.json` ファイルは Elastic Beanstalk 固有の JSON ファイルで、Docker コンテナセットを Elastic Beanstalk アプリケーションとしてデプロイする方法を記述します。このファイルをマルチコンテナ Docker 環境に使用できます。`Dockerrun.aws.json` は、環境内の各コンテナインスタンスにデプロイされるコンテナと、マウントするコンテナのホストインスタンス上に作成されるデータボリュームを記述します。

`Dockerrun.aws.json` ファイルは、単独で使用することも、他のソースコードと共にアーカイブに圧縮して使用することもできます。`Dockerrun.aws.json` と共にアーカイブされるソースコードは、コンテナインスタンスにデプロイされ、`/var/app/current/` ディレクトリでアクセスできます。構成の `volumes` セクションを使用して、インスタンスで実行されるコンテナのマウントポイントを提供します。また、埋め込みコンテナ定義の `mountPoints` セクションを使用して、コンテナからマウントポイントをマウントします。

以下のコードサンプルは、Datadog Agent を宣言する `Dockerrun.aws.json` を示しています。`containerDefinitions` セクションを、ご使用の [Datadog API キー][1]、タグ (オプション)、および追加のコンテナ定義で更新してください。必要に応じて、このファイルを上述の追加コンテンツと共に圧縮できます。このファイルの構文の詳細については、[マルチコンテナ Docker コンフィギュレーション][2]を参照してください。

**注**:

- 多くのリソースを使用する場合は、メモリの上限を上げる必要があります。
- すべてのホストが同じ Agent バージョンを実行するようにするには、`agent:7` を [Docker イメージ][3]の特定のマイナーバージョンに変更することをお勧めします。
{{< site-region region="us3,eu,gov" >}}
- `DD_SITE` を {{< region-param key="dd_site" code="true" >}} に設定して、Agent が正しい Datadog の場所にデータを送信するようにします。
{{< /site-region >}}

```json
{
    "AWSEBDockerrunVersion": 2,
    "volumes": [
        {
            "name": "docker_sock",
            "host": {
                "sourcePath": "/var/run/docker.sock"
            }
        },
        {
            "name": "proc",
            "host": {
                "sourcePath": "/proc/"
            }
        },
        {
            "name": "cgroup",
            "host": {
                "sourcePath": "/cgroup/"
            }
        }
    ],
    "containerDefinitions": [
        {
            "name": "dd-agent",
            "image": "gcr.io/datadoghq/agent:7",
            "environment": [
                {
                    "name": "DD_API_KEY",
                    "value": "<YOUR_DD_API_KEY>"
                },
                {
                    "name": "DD_SITE",
                    "value": "<YOUR_DD_SITE>"
                },
                {
                    "name": "DD_TAGS",
                    "value": "<SIMPLE_TAG>, <KEY:VALUE_TAG>"
                }
            ],
            "memory": 256,
            "mountPoints": [
                {
                    "sourceVolume": "docker_sock",
                    "containerPath": "/var/run/docker.sock",
                    "readOnly": false
                },
                {
                    "sourceVolume": "proc",
                    "containerPath": "/host/proc",
                    "readOnly": true
                },
                {
                    "sourceVolume": "cgroup",
                    "containerPath": "/host/sys/fs/cgroup",
                    "readOnly": true
                }
            ]
        }
    ]
}
```

#### 環境の作成

コンテナ定義が完了したら、それを Elastic Beanstalk に送信します。具体的な手順については、AWS Elastic Beanstalk ドキュメント内の [マルチコンテナ Docker 環境][4]を参照してください。

#### DogStatsD

[マルチコンテナ Docker 環境][4]で DogStatsD を使用してアプリケーションコンテナからカスタムメトリクスを収集するには、`Dockerrun.aws.json` に以下の追加を行います。

1. `dd-agent` コンテナの下に環境変数 `DD_DOGSTATSD_NON_LOCAL_TRAFFIC` を追加します。

    ```json
    {
      "name": "DD_DOGSTATSD_NON_LOCAL_TRAFFIC",
      "value": "true"
    }
    ```

2. アプリケーションコンテナの下に `dd-agent` コンテナへのリンクを追加します。

    ```text
    "links": [ "dd-agent:dd-agent"]
    ```

詳細については、[DogStatsD と Docker][5] を参照してください。



[1]: https://app.datadoghq.com/organization-settings/api-keys
[2]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_v2config.html
[3]: https://gcr.io/datadoghq/agent
[4]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_ecstutorial.html
[5]: https://docs.datadoghq.com/ja/integrations/faq/dogstatsd-and-docker/
{{% /tab %}}

{{< /tabs >}}

#### 複数 Docker コンテナ

1. アプリケーションと同じ `Dockerrun.aws.json` で、`datadog/agent` イメージを使用して Datadog Agent コンテナを追加します。以下を追加します。
    - `portMappings` セクションで、`containerPort` 8126 と `hostPort` 8126 を追加します。
    - `environment` セクションで、`DD_APM_ENABLED` と `DD_APM_NON_LOCAL_TRAFFIC` を `true` に設定します。
2. [トレーシングライブラリのセットアップ][14]でインスツルメントされたアプリケーションコンテナで、以下を追加します。
    - `environment` セクションで、`DD_AGENT_HOST` と呼ばれる環境変数を Datadog Agent コンテナの名前に追加します。
    - `links` セクションで、Agent コンテナを環境変数として使用されるように設定します。

以下の例を参照してください。

```text
 "containerDefinitions": [    {
      "name": "dd-agent",
      "image": "datadog/agent:latest",
      "environment": [
          {
              "name": "DD_API_KEY",
              "value": "<api key>"
          },
          {
              "name": "DD_APM_ENABLED",
              "value": "true"
          },
          {
             "name": "DD_APM_NON_LOCAL_TRAFFIC",
             "value": "true"
          },
         # その他必要な環境変数
      ],
      "portMappings": [
        {
          "hostPort": 8126,
          "containerPort": 8126
        }
      ],
      "memory": 256,
      "mountPoints": [
          # 必要なマウントポイント
         }
      ]
    },
    {
      "name": "application-container",
      "image": "<application image name>",
      "environment": [
        {
          "name": "DD_AGENT_HOST",
          "value": "dd-agent",
          # その他必要な環境変数
        }
      ],
      "links": [
        "dd-agent:dd-agent"
      ],

```

## トラブルシューティング

ご不明な点は、[Datadog のサポートチーム][5]までお問い合わせください。

## その他の参考資料

{{< partial name="whats-next/whats-next.html" >}}


[1]: https://docs.datadoghq.com/ja/integrations/amazon_web_services/
[2]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/health-enhanced.html
[3]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/health-enhanced-cloudwatch.html#health-enhanced-cloudwatch-console
[4]: https://docs.datadoghq.com/ja/agent/guide/why-should-i-install-the-agent-on-my-cloud-instances/
[5]: https://docs.datadoghq.com/ja/help/