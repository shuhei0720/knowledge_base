# Azure Firewall：ネットワークルール／アプリケーションルールの DNS 挙動と運用上の注意

このドキュメントは Azure Firewall のルール種類ごとの DNS 挙動の違いと、運用上の注意点、ログ確認方法、及び運用で役立つ自動棚卸しスクリプト（PowerShell + az CLI）をまとめた実務向けのメモです。

想定読者: ネットワーク/セキュリティ運用者、クラウド基盤エンジニア

---

## 1. ネットワークルール と アプリケーションルール の DNS 挙動の違い

- ネットワークルール（Network rules）
  - FQDN ターゲットを設定した場合、Firewall は定期的にその FQDN を名前解決して得た IP をキャッシュ（保持）します。
  - クライアントから通信があった際は、保持している IP を用いてパケットを評価／転送します。
  - そのため、FQDN → IP の解決タイミングと実際の接続タイミングの間に IP が変わる（PaaS の背後 IP 変更など）と期待通りの制御にならない可能性があります。

- アプリケーションルール（Application rules）
  - HTTP/HTTPS 等のアプリケーションルールは、クライアントからの通信時に HTTP ヘッダ（Host）や SNI（TLS）に含まれるホスト名を参照します。
  - アプリケーションルールでホスト名が検出されたときに、そのホスト名を名前解決して到達先 IP を取得して評価を行おうとします（オンデマンドで名前解決が走る）。
  - 重要な注意点: クライアント OS 側で DNS サフィックスが設定されている環境で、短縮されたホスト名（末尾が省略された FQDN）をアプリケーションルール側に登録すると、名前解決の挙動により期待どおりに制御できない場合があります。つまり、OS 側の DNS サフィックス補完によりヘッダで渡される値とポリシーの FQDN が一致しないことがあるため、アプリケーションルールへは完全修飾ドメイン名（FQDN）を登録することを推奨します。

### 実務的な示唆
- 可変 IP をもつ PaaS を対象にする場合は、可能ならアプリケーションルール（ホストベース）で制御するか、ネットワークルールのキャッシュ更新ポリシーと運用手順を整備する。
- クライアント側で DNS サフィックス管理がある組織環境では、アプリケーションルールは FQDN で運用すること。

---

## 2. ネットワークルールでブラックリスト運用する場合の注意点

- Azure Firewall のルール評価は順序とルール種別に依存します。ネットワークルールで特定の宛先を「すべて拒否（deny）」すると、その後のアプリケーションルールは評価されず、該当トラフィックはネットワークルールの評価で止められます。
- つまり「ネットワークルールで広く拒否 → 下位でアプリケーションルールで例外許可」を期待する構成は期待どおりに動作しない可能性があります。

対策:
- ブラックリスト運用をする場合はルールの順序とスコープを慎重に設計する。
- 可能な限りアプリケーションルールでホワイトリストを設計し、ネットワークルールは必要最小限の通過/遮断に留めることを検討する。

---

## 3. ネットワークルールの ICMP（と Protocol=Any の注意）

- ネットワークルールは TCP、UDP、ICMP、または任意の IP プロトコル（Any）で構成できます。任意 IP は IANA のプロトコル番号で定義される全ての IP プロトコルを指します。
- 宛先ポートが明示的に構成されている場合、ルールは TCP/UDP ルールへ変換されます。
- 歴史的経緯: 2020-11-09 より前は `Any` が TCP/UDP/ICMP を意味していました。そのため、過去に作成されたルールで `Protocol = Any` かつ `destination ports = '*'` の設定が残っている場合があります。現在の意味（任意 IP プロトコル）に従ってすべての IP プロトコルを許可する意図が無いなら、そのルールを修正して必要なプロトコル（TCP、UDP、または ICMP）を明示することを推奨します。

参考: https://learn.microsoft.com/ja-jp/azure/firewall/rule-processing

---

## 4. Firewall のログ確認方法（簡易）

- Azure Firewall の診断ログは Azure Diagnostics / Log Analytics に送信して KQL（Kusto Query Language）で検索できます。
- 代表的な KQL の出し方（例: 特定ルールのログ抽出）:

```kql
AzureDiagnostics
| where Category == 'AzureFirewallNetworkRule' or Category == 'AzureFirewallApplicationRule'
| where TimeGenerated > ago(7d)
| project TimeGenerated, ResourceId, Category, OperationName, msg_s, srcIp_s, destIp_s, destPort_s, Rule_s
| order by TimeGenerated desc
```

- 運用では、Log Analytics に送ったログをダッシュボード化してアラート（例: 特定拒否の急増）を設定すると効率的です。

---

## 5. よくある運用上の誤解・注意点

- `*.xxx.com` で穴あけ（ワイルドカード許可）しても `xxx.com`（裸ドメイン）は自動で許可されない点に注意。
- FQDN にアンダースコア（`_`）は使えない（FQDN の仕様に準拠しないため、Azure Firewall の FQDN マッチに使えない）。
- Azure Firewall の穴あけは基本的に「送信元 → 送信先」方向の通信だけで問題ない。つまり HTTPS、RDP、SMB、NTP、ICMP 等のアウトバウンド／インバウンドを用途に応じて片方向で穴あけすれば十分であることが多い。

---

## 6. Azure Firewall の自動棚卸しスクリプト（PowerShell + az CLI）

以下は運用で役に立つ「未使用ルール検出」スクリプトの例（PowerShell）。Log Analytics（Azure Diagnostics）に送った Firewall ログを参照して、過去 N 日で使われていないルールを CSV 出力します。実行前に `az login` 済みであること。Cloud Shell ではセッション古くなることがあるので注意。

```powershell
# PowerShell + AZ CLI: 未使用の Azure Firewall ルール（ネットワーク／アプリケーションを別処理、エラー行抑制＋トークン明示）
$ErrorActionPreference = 'Stop'

# ===== 変数定義 （ここから）=====
# Log Analytics ワークスペースのリソースID
$LA_WS_RESOURCE_ID = "/subscriptions/7f043232-66a7-4d79-a660-fcb22d1644d0/resourcegroups/rg-hub-prod-resource-01/providers/microsoft.operationalinsights/workspaces/log-hub-prod-collection-01"

# 2つの Azure Firewall Policy のリソースID（2つ目は任意、空ならスキップ）
$FirewallPolicyResourceId1 = "/subscriptions/04fa21cc-4c6b-47cc-83f3-2c2ef7e3c8c8/resourceGroups/rg-hub-prod-network-01/providers/Microsoft.Network/firewallPolicies/afwp-hub-prod-outbound-01"
$FirewallPolicyResourceId2 = ""  # 空ならスキップ

# 参照期間（日）
$LookbackDays = 365
# Category（表記揺れ対応）
$NetworkCategories = @('AZFWNetworkRule','AzureFirewallNetworkRule')
$AppCategories     = @('AZFWApplicationRule','AzureFirewallApplicationRule')

# CSV出力設定
$ExportCsv = $true
$OutNetworkCsv = './unused_network_rules.csv'
$OutAppCsv     = './unused_app_rules.csv'

# ===== 変数定義 （ここまで）=====

# （中略）
# スクリプト全文は運用環境に合わせて変数を調整してご利用ください。

# ===== メイン処理（ここまで） =====
Ensure-AzCli
$armToken = Get-ArmToken -Resource 'https://management.azure.com/'
# ...（スクリプトの残りは上記提供スニペットを参照）
```

> 注意: スクリプト中のリソース ID やサブスクリプションはダミー/例です。実行前に環境に合わせて書き換えてください。

---

## 7. まとめ・運用チェックリスト（簡易）

- アプリケーションルールはホスト名（FQDN）ベースで評価されることを理解する。
- クライアント側の DNS サフィックスや短縮名により意図しない挙動が発生するため、FQDN での登録を徹底する。
- ネットワークルールの `Any` 設定や歴史的に作成されたルールを見直し、意図したプロトコルのみを明示する。
- ブラックリスト運用はルール評価順序に注意。必要ならホワイトリスト運用へ移行検討。
- ログは Log Analytics へ集約し、KQL で可視化／アラート設計を行う。
- 定期的に未使用ルールの棚卸しを行い、ルール肥大化を防ぐ。

---

参照:
- Azure Firewall rule processing: https://learn.microsoft.com/ja-jp/azure/firewall/rule-processing


作成済みファイル: `/workspaces/knowledge_base/AzureFirewall/【Azure】AzureFirewall_DNS挙動と運用上の注意.md`

