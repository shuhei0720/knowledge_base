# Hub&SpokeのプライベートDNS：ナレッジ

## 概要
Hub（集約DNS基盤）にプライベートDNSゾーンを配置し、Spoke の仮想ネットワークからそのゾーンを参照して名前解決を行うアーキテクチャを対象とします。本記事では具体的な実装手順を示し、PaaS リソースの Private Endpoint 利用時に必要な DNS の扱い（自動登録）までカバーします。

ポイント:
- Hub にゾーンを集約し、VNet リンク／Private DNS Resolverで Spoke から参照するパターンを解説
- Azure Policy による自動レコード登録を前提とした実装


## 前提・用語
- Hub: 集約 DNS ゾーンをホストする管理ネットワーク。
- Spoke: ワークロード用の各 VNet（Hub とピアリングで接続）。
- Private DNS Zone: Azure のプライベート権威 DNS ゾーン（例: `privatelink.blob.core.windows.net`）。
- Private Endpoint: PaaS リソースが割り当てるプライベート IP 経由でアクセスするためのリソース。
- 自動登録: Azure Policyで、Private Endpoint 作成時に Hub の DNS ゾーンへ A レコードを自動作成する仕組み。

前提:
- Hub と Spoke の接続が完了しており、DNS の到達経路（VNet リンク、Private DNS Resolver）が確立されていること。
- すべてのコマンド例では `RESOURCE_GROUP` / `VNET_NAME` / `PE_NAME` / `ZONE_NAME` 等を実環境の値に置換してください。


## 前提: Azure プライベートDNSの仕組み（技術解説）

ここでは実装前に知っておくべき Azure のプライベート DNS に関する主要な仕組みをまとめます。

- プライベート DNS ゾーン
  - Azure 上の「プライベート権威ゾーン」で、例: `privatelink.blob.core.windows.net` のような PaaS 向けのゾーンを Hub に作成して集約します。
  - ゾーンはリソースとして管理され、FQDN と A/CNAME レコードを内部的に保持します。

- Virtual Network Link
  - プライベート DNS ゾーンは Virtual Network Link を使って VNet と関連付けます。
  - `registrationEnabled` を `true` にすると、その VNet にデプロイされた VM の NIC がゾーンへ自動登録される（VM 自動登録）機能が働きますが、Private Endpoint の場合は通常 `registrationEnabled=false` として参照用にリンクします。

- Private Endpoint と Private DNS Zone Group
  - Private Endpoint を作成すると、それを Hub のプライベート DNS ゾーンへ紐づけるためのリソース `Microsoft.Network/privateEndpoints/privateDnsZoneGroups`（ゾーングループ）が使われます。
  - このゾーングループが作られると、対象のプライベート DNS ゾーンに Private Endpoint のプライベート IP を参照するレコードが出現します。

- 自動登録（ポリシー）
  - Azure Policy（多くは `deployIfNotExists`）やマネージドデプロイメントを用いて、Private Endpoint の作成時に自動で対応する `privateDnsZoneGroups` を作成する運用を一般的に取ります。
  - ポリシーは `resourceType`（例: `Microsoft.Storage/storageAccounts`）と `groupId`（例: `blob`）などで条件判定し、該当するゾーン ID をパラメーターで受け取ってデプロイします。

- 名前解決の流れ
1. Spoke のクライアントが FQDN を問い合わせる。
2. VNet の DNS 設定に応じて指定した Private Resolver へ問い合わせが行く。
3. Private DNS Zone が Hub にあり、Spoke がそのゾーンへリンクされていれば、Hub 側のゾーン情報に基づき Private Endpoint のプライベート IP が返る。
4. オンプレ等に転送が必要な場合は、Azure DNS Private Resolver のアウトバウンド転送ルールでオンプレ DNS サーバへフォワードします。

- Azure DNS Private Resolver（Inbound/Outbound）
  - 大規模構成やオンプレ連携時に使う。Inbound を使うとオンプレ等から Azure の Private DNS を解決でき、Outbound を使うと Azure 側のクエリをオンプレの DNS に転送できます。

- 実装時のポイント
  - Hub に作るゾーンは命名規則を揃え、必要なゾーンを事前にすべて用意しておくとポリシー適用が容易です。
  - ポリシーが `deployIfNotExists` で動作するためには、デプロイ用の Managed Identity に対象ゾーンへの書き込み権限（DNS Contributor 等）を付与する必要があります。
  - `az` CLI で確認する主なコマンド:
    - `az network private-dns zone show -g <rg> -n <zone>`（ゾーン確認）
    - `az network private-dns link vnet list -g <rg> --zone-name <zone>`（VNet リンク確認）
    - `az network private-dns record-set a list -g <rg> -z <zone>`（記録済み A レコード確認）
    - `az network private-endpoint show -g <rg> -n <pe>`（Private Endpoint の状態確認）

このセクションを踏まえて実装を進めてください。

### 図: Hub&Spoke とプライベート DNS の動き（簡易図）


```
  +----------------------+                     +-------------------------------+
  |        Spoke         |                     |             Hub               |
  | +------------------+ |    DNS query/      | +---------------------------+ |
  | | Client VM / App  | |------------------->| | Private DNS Zone (e.g.)   | |
  | +------------------+ |                     | | privatelink.blob.core.... | |
  |                      |    Forward/Resolve  | +---------------------------+ |
  | +------------------+ |<-------------------|                             | |
  | |   Spoke VNet     | |                     | +---------------------------+ |
  | +------------------+ |                     | | Private DNS Resolver /    | |
  +----------------------+                     | | Forwarder                 | |
                                               | +---------------------------+ |
                                               +-------------------------------+

  - Private Endpoint (PE) は Spoke 側に配置され、PE の IP は Hub の Private DNS Zone の
    レコードで解決されます。
  - Hub にゾーンを集約し、Spoke を Virtual Network Link で関連付けて参照します。
  - 必要に応じて Resolver でオンプレへ転送（Outbound）やオンプレからの参照（Inbound）を行います。
```


## 実装方針（設計上の決めごと）
1. ゾーン配置戦略
   - Hub に各サービス向けのプライベート DNS ゾーンを作成（例: `privatelink.blob.core.windows.net` を Hub に作る）。
2. DNS 到達方式の選択
   - 単純: Hub の Private DNS Zone に対して Spoke を Virtual Network Link でリンクする。
   - 大規模/オンプレ混在: Hub に Azure DNS Private Resolver（inbound/outbound）を配置し、オンプレや複数リージョンを中継する。
3. 自動化
   - Azure Policy を用いて Private Endpoint 作成時の DNS レコード自動作成を目指す。ポリシーパラメーターはポリシー割当時に Hub のゾーン ID を指定する。
4. 権限設計
   - DNS ゾーン作成・変更は Hub 運用チームが管理。Spoke 側には最小権限を付与し、必要な操作は RBAC で制御する。


## Azure Policy を使った自動化（ポリシー定義とパラメーター）

自動登録を行う際の一般的パターン:
- Condition: Private Endpoint 作成などのリソース作成をトリガーとする（resourceType / resourceProvider 条件）
- Effect: `deployIfNotExists` を使って、指定された Hub の Private DNS Zone に対して `privateDnsZoneGroups` を作成するテンプレートをデプロイ
- Parameters: 対象ゾーン名／ゾーンリソースID、Managed Identity、リージョンなど

ポリシー適用の注意点:
- `deployIfNotExists` のデプロイは非同期的に行われるため、作成直後にレコードが見えないことがある。
- デプロイ用の Managed Identity に対する RBAC が不足しているとデプロイが失敗する。
- ポリシーで扱うパラメーターは一元管理し、ゾーン名とゾーンIDを混同しないこと。

（ここには代表的な `deployIfNotExists` ポリシー JSON を挿入して運用者に渡す想定です。必要ならこのドキュメント内にフル JSON を追加します。）

```
Policy 自動化の流れ (簡易)

 Client/Spoke --(Private Endpoint作成)--> Policy (deployIfNotExists 条件マッチ)
             |
             v
           Managed Identity
             |
             v
        Deployment (privateDnsZoneGroups)
             |
             v
         Hub の Private DNS Zone へレコード追加
```


## 実装手順（共通）

1. Hub に必要な Private DNS Zone を作成

```bash
# 例: プライベートゾーン作成
az network private-dns zone create -g <HUB_RG> -n privatelink.blob.core.windows.net
```

2. Spoke をゾーンにリンク（参照専用）

```bash
az network private-dns link vnet create -g <HUB_RG> -n link-to-spoke1 --zone-name privatelink.blob.core.windows.net --virtual-network /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Network/virtualNetworks/<spoke-vnet> --registration-enabled false
```

3. Private Endpoint を作成（通常は各サービスのリソース側で作成）

```bash
az network private-endpoint create -g <SPOKE_RG> -n <pe-name> --vnet-name <spoke-vnet> --subnet <subnet> --private-connection-resource-id <resource-id> --group-ids <groupId> --connection-name <conn-name>
```

4. ポリシーで自動登録（ポリシーを適用していればここで自動的に `privateDnsZoneGroups` が作られる）

5. 手動確認:
- Private DNS ゾーン内に A/CNAME が作成されているか
- Spoke から名前解決できるか（`nslookup`/`dig`）

```
実装手順の概観

 [1] Hub にゾーン作成 -> [2] Spoke をリンク -> [3] Private Endpoint 作成 -> [4] Policy が自動登録 -> [5] 動作確認

 (1) ---> (2) ---> (3) ---> (4) ---> (5)
```


## サービス別実装ガイド（代表例）

以下は代表的な PaaS の扱い方（具体的には環境ごとに差分あり）。

```
ゾーンとPEの関係（簡易）

  [PaaS Service] --(Private Endpoint)--> [Spoke PE IP]
                                    |
                                    v
                          [Hub Private DNS Zone]

PE により Hub 側ゾーンの A/CNAME が参照され、Spoke から名前解決されます。
```

- Azure Storage (Blob, File, Queue, Table)
  - Private Endpoint を作成すると `privatelink.blob.core.windows.net` などのゾーンにレコードが必要。
  - 通常はポリシーで自動登録する。

- Azure SQL
  - `privatelink.database.windows.net` 等、CNAME を利用するケースがあり、CNAME の取り扱いに注意する。

- Cognitive Services / Azure AI
  - サービスにより `privatelink.cognitiveservices.azure.com` のような独自のゾーン名を持つ。

- AKS (Private Cluster)
  - AKS の一部の機能は外部 DNS 名を参照するため、必要に応じてカスタム DNS レコードを追加する。

（詳細は実際にデプロイするサービス毎のドキュメントを参照し、必要に応じてこの節を展開してください。）


## サービス一覧: プライベートDNSゾーン参照表

以下に「主要サービスの要約」と「完全参照表（折りたたみ）」を用意しました。普段参照する主要サービスは上の要約を見てください。詳細は展開して確認できます。

### 主要サービス要約

| サービスカテゴリ | 代表リソース | 代表的なプライベートDNSゾーン |
|---|---:|---|
| 記憶域 | Storage (Blob) | `privatelink.blob.core.windows.net` |
| データベース | Azure SQL | `privatelink.database.windows.net` |
| セキュリティ | Key Vault | `privatelink.vaultcore.azure.net` |
| Web | App Service | `privatelink.azurewebsites.net` |
| コンテナ | AKS (API) | `privatelink.{regionName}.azmk8s.io` |

<details>
<summary>完全参照表（展開して全項目を表示）</summary>

以下は対象サービスごとのプライベートDNSゾーン名と、ドキュメント内で想定している自動登録ポリシーの種別、使用可能リージョン等の一覧です。

| 分類 | リソースの種類 | サブリソース | プライベートDNSゾーンの名前 | 自動登録Azureポリシー | 使用可能リージョン | 備考 |
|---|---|---|---|---|---|---|
| AI + 機械学習 | Azure Machine Learning (Microsoft.MachineLearningServices/workspaces) | amlworkspace | privatelink.api.azureml.ms | Configure Azure Machine Learning workspace to use private DNS zones | 制限なし | |
| AI + 機械学習 | Azure Machine Learning (Microsoft.MachineLearningServices/workspaces) | amlworkspace | privatelink.notebooks.azure.net | Configure Azure Machine Learning workspace to use private DNS zones | 制限なし | |
| AI + 機械学習 | Azure AI サービス (Microsoft.CognitiveServices/accounts) | account | privatelink.cognitiveservices.azure.com | カスタムポリシー(Configure a private DNS Zone ID for Cognitive Services account groupID) | 制限なし | |
| AI + 機械学習 | Azure AI サービス (Microsoft.CognitiveServices/accounts) | account | privatelink.openai.azure.com | カスタムポリシー(Configure a private DNS Zone ID for Cognitive Services account groupID) | 制限なし | |
| AI + 機械学習 | Azure Bot Service (Microsoft.BotService/botServices) | Bot | privatelink.directline.botframework.com | Configure BotService resources to use private DNS zones | 制限なし | |
| AI + 機械学習 | Azure Bot Service (Microsoft.BotService/botServices) | Token | privatelink.token.botframework.com | Configure BotService resources to use private DNS zones | 制限なし | |
| 分析 | Azure Synapse Analytics (Microsoft.Synapse/workspaces) | Sql | privatelink.sql.azuresynapse.net | Configure Azure Synapse workspaces to use private DNS zones | 制限なし | |
| 分析 | Azure Synapse Analytics (Microsoft.Synapse/workspaces) | SqlOnDemand | privatelink.sql.azuresynapse.net | Configure Azure Synapse workspaces to use private DNS zones | 制限なし | |
| 分析 | Azure Synapse Analytics (Microsoft.Synapse/workspaces) | Dev | privatelink.dev.azuresynapse.net | Configure Azure Synapse workspaces to use private DNS zones | 制限なし | |
| 分析 | Azure Synapse Studio (Microsoft.Synapse/privateLinkHubs) | Web | privatelink.azuresynapse.net | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |
| 分析 | Azure Event Hubs (Microsoft.EventHub/namespaces) | namespace | privatelink.servicebus.windows.net | Configure Event Hub namespaces to use private DNS zones | 制限なし | |
| 分析 | Azure Data Factory (Microsoft.DataFactory/factories) | dataFactory | privatelink.datafactory.azure.net | Configure private DNS zones for private endpoints that connect to Azure Data Factory | 制限なし | |
| 分析 | Azure Data Factory (Microsoft.DataFactory/factories) | portal | privatelink.adf.azure.com | Configure private DNS zones for private endpoints that connect to Azure Data Factory | 制限なし | |
| 分析 | Azure HDInsight (Microsoft.HDInsight/clusters) | gateway | privatelink.azurehdinsight.net | Configure Azure HDInsight clusters to use private DNS zones | 制限なし | |
| 分析 | Azure HDInsight (Microsoft.HDInsight/clusters) | headnode | privatelink.azurehdinsight.net | Configure Azure HDInsight clusters to use private DNS zones | 制限なし | |
| 分析 | Azure Data Explorer (Microsoft.Kusto/Clusters) | cluster | privatelink.{regionName}.kusto.windows.net | カスタムポリシー(プライベート DNS ゾーンを使用するように Azure Data Explorerを構成する) | japaneast | 他リージョンを使用したい場合は基盤運用者に相談 |
| 分析 | Azure Data Explorer (Microsoft.Kusto/Clusters) | cluster | privatelink.blob.core.windows.net | カスタムポリシー(プライベート DNS ゾーンを使用するように Azure Data Explorerを構成する) | japaneast | 他リージョンを使用したい場合は基盤運用者に相談 |
| 分析 | Azure Data Explorer (Microsoft.Kusto/Clusters) | cluster | privatelink.queue.core.windows.net | カスタムポリシー(プライベート DNS ゾーンを使用するように Azure Data Explorerを構成する) | japaneast | 他リージョンを使用したい場合は基盤運用者に相談 |
| 分析 | Azure Data Explorer (Microsoft.Kusto/Clusters) | cluster | privatelink.table.core.windows.net | カスタムポリシー(プライベート DNS ゾーンを使用するように Azure Data Explorerを構成する) | japaneast | 他リージョンを使用したい場合は基盤運用者に相談 |
| 分析 | Microsoft Power BI (Microsoft.PowerBI/privateLinkServicesForPowerBI) | tenant | privatelink.analysis.windows.net | カスタムポリシー | 制限なし | |
| 分析 | Microsoft Power BI (Microsoft.PowerBI/privateLinkServicesForPowerBI) | tenant | privatelink.pbidedicated.windows.net | カスタムポリシー | 制限なし | |
| 分析 | Microsoft Power BI (Microsoft.PowerBI/privateLinkServicesForPowerBI) | tenant | privatelink.tip1.powerquery.microsoft.com | カスタムポリシー | 制限なし | |
| 分析 | Azure Databricks (Microsoft.Databricks/workspaces) | databricks_ui_api | privatelink.azuredatabricks.net | Configure Azure Databricks workspace to use private DNS zones | 制限なし | |
| 分析 | Azure Databricks (Microsoft.Databricks/workspaces) | browser_authentication | privatelink.azuredatabricks.net | Configure Azure Databricks workspace to use private DNS zones | 制限なし | |
| Compute | Azure Batch (Microsoft.Batch/batchAccounts) | batchAccount | privatelink.batch.azure.com | Deploy - Configure private DNS zones for private endpoints that connect to Batch accounts | 制限なし | |
| Compute | Azure Batch (Microsoft.Batch/batchAccounts) | nodeManagement | privatelink.batch.azure.com | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |
| Compute | Azure Virtual Desktop (Microsoft.DesktopVirtualization/workspaces) | global | privatelink-global.wvd.microsoft.com | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |
| Compute | Azure Virtual Desktop (Microsoft.DesktopVirtualization/workspaces) | feed | privatelink.wvd.microsoft.com | Configure Azure Virtual Desktop workspace resources to use private DNS zones | 制限なし | |
| Compute | Azure Virtual Desktop (Microsoft.DesktopVirtualization/hostpools) | connection | privatelink.wvd.microsoft.com | Configure Azure Virtual Desktop hostpool resources to use private DNS zones | 制限なし | |
| Containers | Azure Kubernetes Service - Kubernetes API (Microsoft.ContainerService/managedClusters) | management | privatelink.{regionName}.azmk8s.io | カスタムポリシーを使用して・AKSで作成されるインフラストラクチャリソースグループのみプライベートDNSゾーンの作成を許可(プライベートDNSゾーンの作成を禁止する(AKSでの自動作成を除く))・プライベートDNSゾーンを自動的にHubネットワークにリンク(SpokeのプライベートDNSゾーンをHubに紐づける) | 制限なし | |
| Containers | Azure Container Apps (Microsoft.App/ManagedEnvironments) | managedEnvironment | privatelink.{regionName}.azurecontainerapps.io | カスタムポリシー(Deploy-Private-DNS-Generic) | japaneast | 他リージョンを使用したい場合は基盤運用者に相談 |
| Containers | Azure Container Registry (Microsoft.ContainerRegistry/registries) | registry | privatelink.azurecr.io | Configure Container registries to use private DNS zones | 制限なし | |
| データベース | Azure SQL Database (Microsoft.Sql/servers) | sqlServer | privatelink.database.windows.net | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |
| データベース | Azure SQL Managed Instance (Microsoft.Sql/managedInstances) | managedInstance | privatelink.{dnsPrefix}.database.windows.net | プライベートDNSゾーン名に可変の値が入るため対応不可 | 制限なし | 使用したい場合は基盤運用者に相談 |
| データベース | Azure Cosmos DB (Microsoft.DocumentDB/databaseAccounts) | Sql | privatelink.documents.azure.com | Configure CosmosDB accounts to use private DNS zones | 制限なし | |
| データベース | Azure Cosmos DB (Microsoft.DocumentDB/databaseAccounts) | MongoDB | privatelink.mongo.cosmos.azure.com | Configure CosmosDB accounts to use private DNS zones | 制限なし | |
| データベース | Azure Cosmos DB (Microsoft.DocumentDB/databaseAccounts) | Cassandra | privatelink.cassandra.cosmos.azure.com | Configure CosmosDB accounts to use private DNS zones | 制限なし | |
| データベース | Azure Cosmos DB (Microsoft.DocumentDB/databaseAccounts) | Gremlin | privatelink.gremlin.cosmos.azure.com | Configure CosmosDB accounts to use private DNS zones | 制限なし | |
| データベース | Azure Cosmos DB (Microsoft.DocumentDB/databaseAccounts) | Table | privatelink.table.cosmos.azure.com | Configure CosmosDB accounts to use private DNS zones | 制限なし | |
| データベース | Azure Cosmos DB (Microsoft.DocumentDB/databaseAccounts) | Analytical | privatelink.analytics.cosmos.azure.com | Configure CosmosDB accounts to use private DNS zones | 制限なし | |
| データベース | Microsoft.DBforPostgreSQL/serverGroupsv2 | coordinator | privatelink.postgres.cosmos.azure.com | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |
| データベース | Azure Database for PostgreSQL - フレキシブル サーバー (Microsoft.DBforPostgreSQL/flexibleServers) | postgresqlServer | privatelink.postgres.database.azure.com | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |
| データベース | Azure Database for MySQL - フレキシブル サーバー (Microsoft.DBforMySQL/flexibleServers) | mysqlServer | privatelink.mysql.database.azure.com | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |
| データベース | Azure Cache for Redis (Microsoft.Cache/Redis) | redisCache | privatelink.redis.cache.windows.net | Configure Azure Cache for Redis to use private DNS zones | 制限なし | |
| データベース | Azure Cache for Redis Enterprise (Microsoft.Cache/RedisEnterprise) | redisEnterprise | privatelink.redisenterprise.cache.azure.net | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |
| ハイブリッドとマルチクラウド | Azure Arc (Microsoft.HybridCompute/privateLinkScopes) | hybridcompute | privatelink.his.arc.azure.com | Configure Azure Arc Private Link Scopes to use private DNS zones | 制限なし | |
| ハイブリッドとマルチクラウド | Azure Arc (Microsoft.HybridCompute/privateLinkScopes) | hybridcompute | privatelink.guestconfiguration.azure.com | Configure Azure Arc Private Link Scopes to use private DNS zones | 制限なし | |
| ハイブリッドとマルチクラウド | Azure Arc (Microsoft.HybridCompute/privateLinkScopes) | hybridcompute | privatelink.dp.kubernetesconfiguration.azure.com | Configure Azure Arc Private Link Scopes to use private DNS zones | 制限なし | |
| 統合 | Azure Service Bus (Microsoft.ServiceBus/namespaces) | namespace | privatelink.servicebus.windows.net | Configure Service Bus namespaces to use private DNS zones | 制限なし | |
| 統合 | Azure Event Grid (Microsoft.EventGrid/topics) | topic | privatelink.eventgrid.azure.net | Deploy - Configure Azure Event Grid topics to use private DNS zones | 制限なし | |
| 統合 | Azure Event Grid (Microsoft.EventGrid/domains) | domain | privatelink.eventgrid.azure.net | Deploy - Configure Azure Event Grid domains to use private DNS zones | 制限なし | |
| 統合 | Azure Event Grid (Microsoft.EventGrid/namespaces) | topic | privatelink.eventgrid.azure.net | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |
| 統合 | Azure Event Grid (Microsoft.EventGrid/namespaces/topicSpace) | topicSpace | privatelink.ts.eventgrid.azure.net | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |
| 統合 | Azure Event Grid (Microsoft.EventGrid/partnerNamespaces) | partnernamespace | privatelink.eventgrid.azure.net | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |
| 統合 | Azure API Management (Microsoft.ApiManagement/service) | Gateway | privatelink.azure-api.net | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |
| 統合 | Azure Health Data Services (Microsoft.HealthcareApis/workspaces) | healthcareworkspace | privatelink.workspace.azurehealthcareapis.com | カスタムポリシー | 制限なし | |
| 統合 | Azure Health Data Services (Microsoft.HealthcareApis/workspaces) | healthcareworkspace | privatelink.fhir.azurehealthcareapis.com | カスタムポリシー | 制限なし | |
| 統合 | Azure Health Data Services (Microsoft.HealthcareApis/workspaces) | healthcareworkspace | privatelink.dicom.azurehealthcareapis.com | カスタムポリシー | 制限なし | |
| モノのインターネット (IoT) | Azure IoT Hub (Microsoft.Devices/IotHubs) | iotHub | privatelink.azure-devices.net | カスタムポリシー(プライベート DNS ゾーンを使用するようにIoT Hubsを構成する) | 制限なし | |
| モノのインターネット (IoT) | Azure IoT Hub (Microsoft.Devices/IotHubs) | iotHub | privatelink.servicebus.windows.net | カスタムポリシー(プライベート DNS ゾーンを使用するようにIoT Hubsを構成する) | 制限なし | |
| モノのインターネット (IoT) | Azure IoT Hub Device Provisioning Service (Microsoft.Devices/ProvisioningServices) | iotDps | privatelink.azure-devices-provisioning.net | Configure IoT Hub device provisioning instances to use private DNS zones | 制限なし | |
| モノのインターネット (IoT) | Device Update for IoT Hubs (Microsoft.DeviceUpdate/accounts) | DeviceUpdate | privatelink.api.adu.microsoft.com | Configure Azure Device Update for IoT Hub accounts to use private DNS zones | 制限なし | |
| モノのインターネット (IoT) | Azure IoT Central (Microsoft.IoTCentral/IoTApps) | iotApp | privatelink.azureiotcentral.com | Deploy - Configure IoT Central to use private DNS zones | 制限なし | |
| モノのインターネット (IoT) | Azure Digital Twins (Microsoft.DigitalTwins/digitalTwinsInstances) | API | privatelink.digitaltwins.azure.net | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |
| Media | Azure Media Services (Microsoft.Media/mediaservices) | keydelivery | privatelink.media.azure.net | Configure Azure Media Services to use private DNS zones | 制限なし | |
| Media | Azure Media Services (Microsoft.Media/mediaservices) | liveevent | privatelink.media.azure.net | Configure Azure Media Services to use private DNS zones | 制限なし | |
| Media | Azure Media Services (Microsoft.Media/mediaservices) | streamingendpoint | privatelink.media.azure.net | Configure Azure Media Services to use private DNS zones | 制限なし | |
| 管理とガバナンス | Azure Automation (Microsoft.Automation/automationAccounts) | Webhook | privatelink.azure-automation.net | Configure Azure Automation accounts with private DNS zones | 制限なし | |
| 管理とガバナンス | Azure Automation (Microsoft.Automation/automationAccounts) | DSCAndHybridWorker | privatelink.azure-automation.net | Configure Azure Automation accounts with private DNS zones | 制限なし | |
| 管理とガバナンス | Azure Backup (Microsoft.RecoveryServices/vaults) | AzureBackup | privatelink.{regionCode}.backup.windowsazure.com | [Preview]: Configure Recovery Services vaults to use private DNS zones for backup | japaneast | |
| 管理とガバナンス | Azure Site Recovery (Microsoft.RecoveryServices/vaults) | AzureSiteRecovery | privatelink.siterecovery.windowsazure.com | [Preview]: Configure Azure Recovery Services vaults to use private DNS zones | 制限なし | |
| 管理とガバナンス | Azure Monitor (Microsoft.Insights/privateLinkScopes) | azuremonitor | privatelink.monitor.azure.com | Configure Azure Monitor Private Link Scope to use private DNS zones | 制限なし | |
| 管理とガバナンス | Azure Monitor (Microsoft.Insights/privateLinkScopes) | azuremonitor | privatelink.oms.opinsights.azure.com | Configure Azure Monitor Private Link Scope to use private DNS zones | 制限なし | |
| 管理とガバナンス | Azure Monitor (Microsoft.Insights/privateLinkScopes) | azuremonitor | privatelink.ods.opinsights.azure.com | Configure Azure Monitor Private Link Scope to use private DNS zones | 制限なし | |
| 管理とガバナンス | Azure Monitor (Microsoft.Insights/privateLinkScopes) | azuremonitor | privatelink.agentsvc.azure-automation.net | Configure Azure Monitor Private Link Scope to use private DNS zones | 制限なし | |
| 管理とガバナンス | Azure Monitor (Microsoft.Insights/privateLinkScopes) | azuremonitor | privatelink.blob.core.windows.net | Configure Azure Monitor Private Link Scope to use private DNS zones | 制限なし | |
| 管理とガバナンス | Microsoft Purview (Microsoft.Purview/accounts) | account | privatelink.purview.azure.com | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |
| 管理とガバナンス | Microsoft Purview (Microsoft.Purview/accounts) | portal | privatelink.purviewstudio.azure.com | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |
| 管理とガバナンス | Azure Migrate (Microsoft.Migrate/migrateProjects) | Default | privatelink.prod.migration.windowsazure.com | Configure Azure Migrate resources to use private DNS zones | 制限なし | 使用したい場合は基盤運用者に相談 |
| 管理とガバナンス | Azure Migrate (Microsoft.Migrate/assessmentProjects) | Default | privatelink.prod.migration.windowsazure.com | Configure Azure Migrate resources to use private DNS zones | 制限なし | 使用したい場合は基盤運用者に相談 |
| 管理とガバナンス | Azure Resource Manager (Microsoft.Authorization/resourceManagementPrivateLinks) | ResourceManagement | privatelink.azure.com | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |
| 管理とガバナンス | Azure Managed Grafana (Microsoft.Dashboard/grafana) | grafana | privatelink.grafana.azure.com | Configure Azure Managed Grafana workspaces to use private DNS zones | 制限なし | |
| セキュリティ | Azure Key Vault (Microsoft.KeyVault/vaults) | vault | privatelink.vaultcore.azure.net | Configure Azure Key Vaults to use private DNS zones | 制限なし | |
| セキュリティ | Azure Key Vault (Microsoft.KeyVault/managedHSMs) | managedhsm | privatelink.managedhsm.azure.net | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |
| セキュリティ | Azure App Configuration (Microsoft.AppConfiguration/configurationStores) | configurationStores | privatelink.azconfig.io | Configure private DNS zones for private endpoints connected to App Configuration | 制限なし | |
| セキュリティ | Azure Attestation (Microsoft.Attestation/attestationProviders) | standard | privatelink.attest.azure.net | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |
| 記憶域 | Storage アカウント (Microsoft.Storage/storageAccounts) | blob | privatelink.blob.core.windows.net | Configure a private DNS Zone ID for blob groupID | 制限なし | |
| 記憶域 | Storage アカウント (Microsoft.Storage/storageAccounts) | blob_secondary | privatelink.blob.core.windows.net | Configure a private DNS Zone ID for blob_secondary groupID | 制限なし | |
| 記憶域 | Storage アカウント (Microsoft.Storage/storageAccounts) | table | privatelink.table.core.windows.net | Configure a private DNS Zone ID for table groupID | 制限なし | |
| 記憶域 | Storage アカウント (Microsoft.Storage/storageAccounts) | table_secondary | privatelink.table.core.windows.net | Configure a private DNS Zone ID for table_secondary groupID | 制限なし | |
| 記憶域 | Storage アカウント (Microsoft.Storage/storageAccounts) | queue | privatelink.queue.core.windows.net | Configure a private DNS Zone ID for queue groupID | 制限なし | |
| 記憶域 | Storage アカウント (Microsoft.Storage/storageAccounts) | queue_secondary | privatelink.queue.core.windows.net | Configure a private DNS Zone ID for queue_secondary groupID | 制限なし | |
| 記憶域 | Storage アカウント (Microsoft.Storage/storageAccounts) | file | privatelink.file.core.windows.net | Configure a private DNS Zone ID for file groupID | 制限なし | |
| 記憶域 | Storage アカウント (Microsoft.Storage/storageAccounts) | web | privatelink.web.core.windows.net | Configure a private DNS Zone ID for web groupID | 制限なし | |
| 記憶域 | Storage アカウント (Microsoft.Storage/storageAccounts) | web_secondary | privatelink.web.core.windows.net | Configure a private DNS Zone ID for web_secondary groupID | 制限なし | |
| 記憶域 | Azure Data Lake ファイル システム Gen2 (Microsoft.Storage/storageAccounts) | dfs | privatelink.dfs.core.windows.net | Configure a private DNS Zone ID for dfs groupID | 制限なし | |
| 記憶域 | Azure Data Lake ファイル システム Gen2 (Microsoft.Storage/storageAccounts) | dfs_secondary | privatelink.dfs.core.windows.net | Configure a private DNS Zone ID for dfs_secondary groupID | 制限なし | |
| 記憶域 | Azure File Sync (Microsoft.StorageSync/storageSyncServices) | afs | privatelink.afs.azure.net | Configure Azure File Sync to use private DNS zones | 制限なし | |
| 記憶域 | Azure Managed Disks (Microsoft.Compute/diskAccesses) | disks | privatelink.blob.core.windows.net | Configure disk access resources to use private DNS zones | 制限なし | |
| Web | Azure Search (Microsoft.Search/searchServices) | searchService | privatelink.search.windows.net | Configure Azure AI Search services to use private DNS zones | 制限なし | |
| Web | Azure Relay (Microsoft.Relay/namespaces) | namespace | privatelink.servicebus.windows.net | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |
| Web | SignalR (Microsoft.SignalRService/SignalR) | signalr | privatelink.service.signalr.net | Deploy - Configure private DNS zones for private endpoints connect to Azure SignalR Service | 制限なし | |
| Web | Azure Static Web Apps (Microsoft.Web/staticSites) | staticSites | privatelink.{partitionId}.azurestaticapps.net | プライベートDNSゾーン名に可変の値が入るため対応不可 | 制限なし | 使用したい場合は基盤運用者に相談 |
| Web | Azure App Service(Microsoft.Web/hostingEnvironments) | sites | privatelink.appserviceenvironment.net | カスタムポリシー(Configure App Service apps to use private DNS zones with corresponding domain suffix) | 制限なし | |
| Web | Azure App Service(Microsoft.Web/sites) | sites | privatelink.azurewebsites.net | カスタムポリシー(Configure App Service apps to use private DNS zones with corresponding domain suffix) | 制限なし | |
| Web | Azure App Service(Microsoft.Web/sites) | sites | scm.privatelink.azurewebsites.net | カスタムポリシー(Configure App Service apps to use private DNS zones with corresponding domain suffix) | 制限なし | |
| Web | Azure Web PubSub サービス (Microsoft.SignalRService/WebPubSub) | webpubsub | privatelink.webpubsub.azure.com | カスタムポリシー(Deploy-Private-DNS-Generic-AllLocation) | 制限なし | |

</details>

## テンプレート：Azure CLI / Bicep サンプル

- Private DNS Zone 作成（Bicep）

```bicep
resource privateDnsZone 'Microsoft.Network/privateDnsZones@2018-09-01' = {
  name: 'privatelink.blob.core.windows.net'
  location: 'global'
}
```

- privateDnsZoneGroups をデプロイする ARM/Bicep を用いたサンプルは、ポリシーの `deployment` 部分で使います。


## トラブルシュートと検証方法

- 名前解決できない場合:
  - Spoke のネットワーク設定で DNS が正しく向いているか確認（カスタム DNS を使っている場合はフォワーダ設定を見直す）。
  - `az network private-dns record-set a list -g <HUB_RG> -z <zone>` でレコードが存在するか確認。
  - Private Endpoint の状態 (`az network private-endpoint show`) と `privateDnsZoneGroups` の状態を確認。

- ポリシーがデプロイを行わない場合:
  - デプロイ用の Managed Identity に対する RBAC を確認（DNS Contributor 等）。
  - ポリシーの条件式が対象リソースにマッチしているかログで確認。


## 付録: 参考資料
- Azure Private Endpoint と Private DNS の公式ドキュメント
- Azure DNS Private Resolver の公式ドキュメント


## 付録: Azure Policy 定義（代表的な deployIfNotExists テンプレート集）

注意: 以下は実運用で使えるようにパラメーター化したテンプレート例です。各項目（`parameters` や `deployment` 内のテンプレート）は実環境のゾーンリソースID、Managed Identity、割当スコープに合わせて必ず調整してください。これらは動作例であり、そのまま適用すると権限不足や名前衝突で失敗する可能性があります。

1) 汎用: Deploy-Private-DNS-Generic

```json
{
  "properties": {
    "displayName": "Deploy Private DNS Zone Group for Private Endpoint (Generic)",
    "policyType": "Custom",
    "mode": "Indexed",
    "parameters": {
      "privateDnsZoneId": {
        "type": "String",
        "metadata": {"displayName": "Private DNS Zone ResourceId"}
      },
      "dnsZoneConfigName": {
        "type": "String",
        "defaultValue": "default",
        "metadata": {"displayName": "DNS Zone Config Name"}
      }
    },
    "policyRule": {
      "if": {
        "allOf": [
          {"field": "type", "equals": "Microsoft.Network/privateEndpoints"}
        ]
      },
      "then": {
        "effect": "deployIfNotExists",
        "details": {
          "type": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups",
          "existenceCondition": {
            "allOf": [
              {"field": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId", "equals": "[parameters('privateDnsZoneId')]"}
            ]
          },
          "roleDefinitionIds": [],
          "deployment": {
            "properties": {
              "mode": "incremental",
              "parameters": {
                "privateDnsZoneId": {"value": "[parameters('privateDnsZoneId')]"},
                "dnsZoneConfigName": {"value": "[parameters('dnsZoneConfigName')]"}
              },
              "template": {
                "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
                "contentVersion": "1.0.0.0",
                "parameters": {
                  "privateDnsZoneId": {"type": "string"},
                  "dnsZoneConfigName": {"type": "string"}
                },
                "resources": [
                  {
                    "type": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups",
                    "apiVersion": "2021-08-01",
                    "name": "[concat(last(split(deployment().name, '/')), '/', parameters('dnsZoneConfigName'))]",
                    "properties": {
                      "privateDnsZoneConfigs": [
                        {
                          "name": "[parameters('dnsZoneConfigName')]",
                          "properties": {
                            "privateDnsZoneId": "[parameters('privateDnsZoneId')]"
                          }
                        }
                      ]
                    }
                  }
                ]
              }
            }
          }
        }
      }
    }
  }
}
```

2) サービス例: Storage (Blob) 向けポリシー

```json
{
  "properties": {
    "displayName": "Deploy Private DNS for Storage Private Endpoints",
    "policyType": "Custom",
    "mode": "Indexed",
    "parameters": {
      "privateDnsZoneId": {"type": "String"}
    },
    "policyRule": {
      "if": {
        "allOf": [
          {"field": "type", "equals": "Microsoft.Storage/storageAccounts"},
          {"field": "Microsoft.Storage/storageAccounts/privateEndpointConnections[*].name", "exists": "true"}
        ]
      },
      "then": {
        "effect": "deployIfNotExists",
        "details": {
          "type": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups",
          "existenceCondition": {
            "field": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId",
            "equals": "[parameters('privateDnsZoneId')]"
          },
          "deployment": {
            "properties": {
              "mode": "incremental",
              "template": { /* 同様のテンプレートをここに置く（省略可） */ }
            }
          }
        }
      }
    }
  }
}
```

3) App Service 向けテンプレート（代表例）

```json
{
  "properties": {
    "displayName": "Deploy Private DNS for App Service Private Endpoints",
    "policyType": "Custom",
    "mode": "Indexed",
    "parameters": {"privateDnsZoneId": {"type": "String"}},
    "policyRule": {
      "if": {"field": "type", "equals": "Microsoft.Web/sites"},
      "then": {"effect": "deployIfNotExists", "details": {"type": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups", "existenceCondition": {"field": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals": "[parameters('privateDnsZoneId')]"}, "deployment": {"properties": {"mode": "incremental","template": {}}}}}
  }
}
```

4) Cognitive Services / AI 向けテンプレート

```json
{
  "properties": {
    "displayName": "Deploy Private DNS for Cognitive Services Private Endpoints",
    "policyType": "Custom",
    "mode": "Indexed",
    "parameters": {"privateDnsZoneId": {"type": "String"}},
    "policyRule": {"if": {"field": "type", "equals": "Microsoft.CognitiveServices/accounts"},"then": {"effect": "deployIfNotExists","details": {"type": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition": {"field": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals": "[parameters('privateDnsZoneId')]"},"deployment": {"properties": {"mode": "incremental","template": {}}}}}}
  }
}
```

5) IoT Hub 向けテンプレート（代表例）

```json
{
  "properties": {
    "displayName": "Deploy Private DNS for IoT Hub Private Endpoints",
    "policyType": "Custom",
    "mode": "Indexed",
    "parameters": {"privateDnsZoneId": {"type": "String"}},
    "policyRule": {"if": {"field": "type", "equals": "Microsoft.Devices/IotHubs"},"then": {"effect": "deployIfNotExists","details": {"type": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition": {"field": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals": "[parameters('privateDnsZoneId')]"},"deployment": {"properties": {"mode": "incremental","template": {}}}}}}
  }
}
```

6) Data Explorer（Kusto）向けテンプレート（代表例）

```json
{
  "properties": {
    "displayName": "Deploy Private DNS for Data Explorer Private Endpoints",
    "policyType": "Custom",
    "mode": "Indexed",
    "parameters": {"privateDnsZoneId": {"type": "String"}},
    "policyRule": {
      "if": {"field": "type", "equals": "Microsoft.Kusto/clusters"},
      "then": {"effect": "deployIfNotExists","details": {"type": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition": {"field": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals": "[parameters('privateDnsZoneId')]"},"deployment": {"properties": {"mode": "incremental","template": {}}}}}
    }
  }
}
```

7) SQL / CosmosDB / EventHubs / ServiceBus / KeyVault / ContainerRegistry / MachineLearning などの代表テンプレート

（下記はパターン化したテンプレートの例です。`if.field` の `equals` 値を対象リソースタイプに変更しています。実運用では `deployment.properties.template` を具体的に記述してください。）

```json
{
  "templates": [
    {
      "name": "SQL",
      "policy": {
        "properties": {
          "displayName": "Deploy Private DNS for SQL Private Endpoints",
          "policyType": "Custom",
          "mode": "Indexed",
          "parameters": {"privateDnsZoneId": {"type": "String"}},
          "policyRule": {"if": {"field": "type","equals": "Microsoft.Sql/servers"},"then": {"effect": "deployIfNotExists","details": {"type": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition": {"field": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals": "[parameters('privateDnsZoneId')]"},"deployment": {"properties": {"mode": "incremental","template": {}}}}}}
        }
      }
    },
    {
      "name": "CosmosDB",
      "policy": {
        "properties": {
          "displayName": "Deploy Private DNS for CosmosDB Private Endpoints",
          "policyType": "Custom",
          "mode": "Indexed",
          "parameters": {"privateDnsZoneId": {"type": "String"}},
          "policyRule": {"if": {"field": "type","equals": "Microsoft.DocumentDB/databaseAccounts"},"then": {"effect": "deployIfNotExists","details": {"type": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition": {"field": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals": "[parameters('privateDnsZoneId')]"},"deployment": {"properties": {"mode": "incremental","template": {}}}}}}
        }
      }
    },
    {
      "name": "EventHubs",
      "policy": {
        "properties": {
          "displayName": "Deploy Private DNS for Event Hubs Private Endpoints",
          "policyType": "Custom",
          "mode": "Indexed",
          "parameters": {"privateDnsZoneId": {"type": "String"}},
          "policyRule": {"if": {"field": "type","equals": "Microsoft.EventHub/namespaces"},"then": {"effect": "deployIfNotExists","details": {"type": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition": {"field": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals": "[parameters('privateDnsZoneId')]"},"deployment": {"properties": {"mode": "incremental","template": {}}}}}}
        }
      }
    },
    {
      "name": "ServiceBus",
      "policy": {
        "properties": {
          "displayName": "Deploy Private DNS for Service Bus Private Endpoints",
          "policyType": "Custom",
          "mode": "Indexed",
          "parameters": {"privateDnsZoneId": {"type": "String"}},
          "policyRule": {"if": {"field": "type","equals": "Microsoft.ServiceBus/namespaces"},"then": {"effect": "deployIfNotExists","details": {"type": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition": {"field": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals": "[parameters('privateDnsZoneId')]"},"deployment": {"properties": {"mode": "incremental","template": {}}}}}}
        }
      }
    },
    {
      "name": "KeyVault",
      "policy": {
        "properties": {
          "displayName": "Deploy Private DNS for Key Vault Private Endpoints",
          "policyType": "Custom",
          "mode": "Indexed",
          "parameters": {"privateDnsZoneId": {"type": "String"}},
          "policyRule": {"if": {"field": "type","equals": "Microsoft.KeyVault/vaults"},"then": {"effect": "deployIfNotExists","details": {"type": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition": {"field": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals": "[parameters('privateDnsZoneId')]"},"deployment": {"properties": {"mode": "incremental","template": {}}}}}}
        }
      }
    },
    {
      "name": "ContainerRegistry",
      "policy": {
        "properties": {
          "displayName": "Deploy Private DNS for ACR Private Endpoints",
          "policyType": "Custom",
          "mode": "Indexed",
          "parameters": {"privateDnsZoneId": {"type": "String"}},
          "policyRule": {"if": {"field": "type","equals": "Microsoft.ContainerRegistry/registries"},"then": {"effect": "deployIfNotExists","details": {"type": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition": {"field": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals": "[parameters('privateDnsZoneId')]"},"deployment": {"properties": {"mode": "incremental","template": {}}}}}}
        }
      }
    },
    {
      "name": "MachineLearning",
      "policy": {
        "properties": {
          "displayName": "Deploy Private DNS for Machine Learning Private Endpoints",
          "policyType": "Custom",
          "mode": "Indexed",
          "parameters": {"privateDnsZoneId": {"type": "String"}},
          "policyRule": {"if": {"field": "type","equals": "Microsoft.MachineLearningServices/workspaces"},"then": {"effect": "deployIfNotExists","details": {"type": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition": {"field": "Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals": "[parameters('privateDnsZoneId')]"},"deployment": {"properties": {"mode": "incremental","template": {}}}}}}
        }
      }
    }
  ]
}
```

8) AKS（Private Cluster）向け注意事項

- AKS はクラスターの構成やアドオンによって DNS 要件が変わるため、単一テンプレートで完結しない場合があります。実装手順のヒント:
  - クラスター内部から必要な FQDN（マニフェストやアドオンが参照する DNS 名）を洗い出す
  - CoreDNS のフォワード設定や `stubDomains` を確認・必要なら調整する
  - 必要に応じて、ポリシーで `Microsoft.Network/privateDnsZones/A` のレコードを作るテンプレートを用意する


各テンプレートは「対象リソースの特定」「デプロイ用 Managed Identity の権限」「パラメーターの正確な紐付け」の3点が揃って初めて安全に運用できます。必要であれば、実際の運用環境に合わせたフルポリシー（パラメーター定義・ロール定義・サンプルパラメーター値）を作成して、このファイルへ挿入します。


<!-- 付録B: 全サービス向けポリシー定義のインライン挿入 -->

# 付録 B: 全サービス向け Azure Policy 定義（deployIfNotExists）

以下は、本文の参照表にある各サービス／リソースタイプ向けの `deployIfNotExists` 型ポリシー定義テンプレート集です。各テンプレートはパラメーター化されています。割当時に `privateDnsZoneId` に Hub のゾーンリソース ID を渡して利用してください。

## 使用例（ポリシー割当時のパラメーター）

```json
{
  "parameters": {
    "privateDnsZoneId": {
      "value": "/subscriptions/<sub>/resourceGroups/<HubRG>/providers/Microsoft.Network/privateDnsZones/privatelink.blob.core.windows.net"
    }
  }
}
```

> 注: 実運用ではデプロイ用 Managed Identity に `DNS Contributor` 権限を付与してください。`deployment.properties.template` 部分には Hub の `privateDnsZoneGroups` を作成する ARM/Bicep を入れてください。


以下はサービス別テンプレート（代表例）。必要に応じて `if` 条件や `existenceCondition` を調整してください。

### Storage (Microsoft.Storage/storageAccounts)

```json
{ "properties": { "displayName": "Deploy Private DNS for Storage Private Endpoints", "policyType": "Custom", "mode": "Indexed", "parameters": {"privateDnsZoneId": {"type": "String"}}, "policyRule": { "if": {"allOf": [{"field": "type","equals": "Microsoft.Storage/storageAccounts"},{"field":"Microsoft.Storage/storageAccounts/privateEndpointConnections[*].name","exists":"true"}]}, "then": {"effect": "deployIfNotExists","details": {"type":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition":{"field":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals":"[parameters('privateDnsZoneId')]"},"deployment":{"properties":{"mode":"incremental","template":{}}}} } } } }
```

### SQL (Microsoft.Sql/servers)

```json
{ "properties": { "displayName": "Deploy Private DNS for SQL Private Endpoints", "policyType": "Custom", "mode": "Indexed", "parameters": {"privateDnsZoneId": {"type": "String"}}, "policyRule": { "if": {"allOf": [{"field": "type","equals": "Microsoft.Sql/servers"},{"field":"Microsoft.Sql/servers/privateEndpointConnections[*].name","exists":"true"}]}, "then": {"effect": "deployIfNotExists","details": {"type":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition":{"field":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals":"[parameters('privateDnsZoneId')]"},"deployment":{"properties":{"mode":"incremental","template":{}}}} } } } }
```

### Key Vault (Microsoft.KeyVault/vaults)

```json
{ "properties": { "displayName": "Deploy Private DNS for Key Vault Private Endpoints", "policyType": "Custom", "mode": "Indexed", "parameters": {"privateDnsZoneId": {"type": "String"}}, "policyRule": { "if": {"field": "type","equals": "Microsoft.KeyVault/vaults"}, "then": {"effect": "deployIfNotExists","details": {"type":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition":{"field":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals":"[parameters('privateDnsZoneId')]"},"deployment":{"properties":{"mode":"incremental","template":{}}}} } } } }
```

### App Service (Microsoft.Web/sites)

```json
{ "properties": { "displayName": "Deploy Private DNS for App Service Private Endpoints", "policyType": "Custom", "mode": "Indexed", "parameters": {"privateDnsZoneId": {"type": "String"}}, "policyRule": { "if": {"field": "type","equals": "Microsoft.Web/sites"}, "then": {"effect": "deployIfNotExists","details": {"type":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition":{"field":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals":"[parameters('privateDnsZoneId')]"},"deployment":{"properties":{"mode":"incremental","template":{}}}} } } } }
```

### Cognitive Services / OpenAI (Microsoft.CognitiveServices/accounts)

```json
{ "properties": { "displayName": "Deploy Private DNS for Cognitive Services Private Endpoints", "policyType": "Custom", "mode": "Indexed", "parameters": {"privateDnsZoneId": {"type": "String"}}, "policyRule": { "if": {"field": "type","equals": "Microsoft.CognitiveServices/accounts"}, "then": {"effect": "deployIfNotExists","details": {"type":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition":{"field":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals":"[parameters('privateDnsZoneId')]"},"deployment":{"properties":{"mode":"incremental","template":{}}}} } } } }
```

### Container Registry (Microsoft.ContainerRegistry/registries)

```json
{ "properties": { "displayName": "Deploy Private DNS for ACR Private Endpoints", "policyType": "Custom", "mode": "Indexed", "parameters": {"privateDnsZoneId": {"type": "String"}}, "policyRule": { "if": {"field": "type","equals": "Microsoft.ContainerRegistry/registries"}, "then": {"effect": "deployIfNotExists","details": {"type":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition":{"field":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals":"[parameters('privateDnsZoneId')]"},"deployment":{"properties":{"mode":"incremental","template":{}}}} } } } }
```

### Cosmos DB (Microsoft.DocumentDB/databaseAccounts)

```json
{ "properties": { "displayName": "Deploy Private DNS for CosmosDB Private Endpoints", "policyType": "Custom", "mode": "Indexed", "parameters": {"privateDnsZoneId": {"type": "String"}}, "policyRule": { "if": {"field": "type","equals": "Microsoft.DocumentDB/databaseAccounts"}, "then": {"effect": "deployIfNotExists","details": {"type":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition":{"field":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals":"[parameters('privateDnsZoneId')]"},"deployment":{"properties":{"mode":"incremental","template":{}}}} } } } }
```

### Event Hubs / Service Bus (Microsoft.EventHub/namespaces / Microsoft.ServiceBus/namespaces)

```json
{ "properties": { "displayName": "Deploy Private DNS for EventHubs/ServiceBus Private Endpoints", "policyType": "Custom", "mode": "Indexed", "parameters": {"privateDnsZoneId": {"type": "String"}}, "policyRule": { "if": {"anyOf": [{"field":"type","equals":"Microsoft.EventHub/namespaces"},{"field":"type","equals":"Microsoft.ServiceBus/namespaces"}]}, "then": {"effect":"deployIfNotExists","details": {"type":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition":{"field":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals":"[parameters('privateDnsZoneId')]"},"deployment":{"properties":{"mode":"incremental","template":{}}}} } } } }
```

### Databricks (Microsoft.Databricks/workspaces)

```json
{ "properties": { "displayName": "Deploy Private DNS for Databricks Private Endpoints", "policyType": "Custom", "mode": "Indexed", "parameters": {"privateDnsZoneId": {"type": "String"}}, "policyRule": { "if": {"field": "type","equals": "Microsoft.Databricks/workspaces"}, "then": {"effect": "deployIfNotExists","details": {"type":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition":{"field":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals":"[parameters('privateDnsZoneId')]"},"deployment":{"properties":{"mode":"incremental","template":{}}}} } } } }
```

### AKS (Microsoft.ContainerService/managedClusters)

```json
{ "properties": { "displayName": "Deploy Private DNS for AKS Private Endpoints", "policyType": "Custom", "mode": "Indexed", "parameters": {"privateDnsZoneId": {"type": "String"}}, "policyRule": { "if": {"field":"type","equals":"Microsoft.ContainerService/managedClusters"}, "then": {"effect":"deployIfNotExists","details": {"type":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition":{"field":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals":"[parameters('privateDnsZoneId')]"},"deployment":{"properties":{"mode":"incremental","template":{}}}} } } } }
```

### IoT (Microsoft.Devices/IotHubs / Microsoft.Devices/ProvisioningServices / Microsoft.IoTCentral/IoTApps)

```json
{ "properties": { "displayName": "Deploy Private DNS for IoT Private Endpoints", "policyType": "Custom", "mode": "Indexed", "parameters": {"privateDnsZoneId": {"type": "String"}}, "policyRule": { "if": {"anyOf": [{"field":"type","equals":"Microsoft.Devices/IotHubs"},{"field":"type","equals":"Microsoft.Devices/ProvisioningServices"},{"field":"type","equals":"Microsoft.IoTCentral/IoTApps"}]}, "then": {"effect":"deployIfNotExists","details": {"type":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition":{"field":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals":"[parameters('privateDnsZoneId')]"},"deployment":{"properties":{"mode":"incremental","template":{}}}} } } } }
```

### 汎用テンプレート（未記載のリソースタイプ用）

```json
{ "properties": { "displayName": "Deploy Private DNS (Generic) - replace resource type", "policyType": "Custom", "mode": "Indexed", "parameters": {"privateDnsZoneId": {"type": "String"}, "targetResourceType": {"type":"String"}}, "policyRule": { "if": {"field":"type","equals":"[parameters('targetResourceType')]"}, "then": {"effect":"deployIfNotExists","details": {"type":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups","existenceCondition":{"field":"Microsoft.Network/privateEndpoints/privateDnsZoneGroups[*].privateDnsZoneConfigs[*].properties.privateDnsZoneId","equals":"[parameters('privateDnsZoneId')]"},"deployment":{"properties":{"mode":"incremental","template":{}}}} } } } }
```


上記テンプレート群は本文の完全表に登場するほとんどのリソースタイプをカバーする汎用/代表パターンです。必要なら優先サービス（例: `Storage`, `SQL`, `KeyVault`, `AppService`, `AKS`）ごとに `deployment.properties.template` をフルに埋めた "そのまま割当可能" な JSON です。
