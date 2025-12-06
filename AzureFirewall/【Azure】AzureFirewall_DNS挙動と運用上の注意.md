# Azure Firewall：ナレッジ達

AzureFirewallの現場で問題になった様々なナレッジを記録します。


## 1. ネットワークルール と アプリケーションルール の DNS 挙動の違い

- ネットワークルール（Network rules）
  - FQDN ターゲットを設定した場合、Firewall は定期的にその FQDN を名前解決して得た IP をキャッシュ（保持）します。
  - クライアントから通信があった際は、保持している IP を用いてパケットを評価／転送します。

- アプリケーションルール（Application rules）
  - HTTP/HTTPS 等のアプリケーションルールは、クライアントからの通信時に HTTP ヘッダ（Host）や SNI（TLS）に含まれるホスト名を参照します。
  - アプリケーションルールでホスト名が検出されたときに、そのホスト名を名前解決して到達先 IP を取得して評価を行おうとします（オンデマンドで名前解決が走る）。
  - 重要な注意点: クライアント OS 側で DNS サフィックスが設定されている環境で、短縮されたホスト名（末尾が省略された FQDN）をアプリケーションルール側に登録すると、名前解決の挙動により期待どおりに制御できない場合があります。つまり、OS 側の DNS サフィックス補完によりヘッダで渡される値とポリシーの FQDN が一致しないことがあるため、アプリケーションルールへは完全修飾ドメイン名（FQDN）を登録することを推奨します。

### まとめ
- クライアント側で DNS サフィックス管理がある組織環境では、アプリケーションルールは FQDN で運用すること。


## 2. ネットワークルールでブラックリスト運用する場合の注意点

- Azure Firewall のルール評価は順序とルール種別に依存します。ネットワークルールで特定の宛先を「すべて拒否（deny）」すると、その後のアプリケーションルールは評価されず、該当トラフィックはネットワークルールの評価で止められます。
- つまり「ネットワークルールで広く拒否 → 下位でアプリケーションルールで例外許可」を期待する構成は期待どおりに動作しない可能性があります。

対策:
- ブラックリスト運用をする場合はルールの順序とスコープを慎重に設計する。
- 可能な限りアプリケーションルールでホワイトリストを設計し、ネットワークルールは必要最小限の通過/遮断に留めることを検討する。


## 3. ネットワークルールの ICMP の注意

- ネットワークルールは TCP、UDP、ICMP、または任意の IP プロトコル（Any）で構成できます。任意 IP は IANA のプロトコル番号で定義される全ての IP プロトコルを指します。
- 宛先ポートが明示的に構成されている場合、ルールは TCP/UDP ルールへ変換されます。
- 歴史的経緯: 2020-11-09 より前は `Any` が TCP/UDP/ICMP を意味していました。そのため、過去に作成されたルールで `Protocol = Any` かつ `destination ports = '*'` の設定が残っている場合があります。現在の意味（任意 IP プロトコル）に従ってすべての IP プロトコルを許可する意図が無いなら、そのルールを修正して必要なプロトコル（TCP、UDP、または ICMP）を明示することを推奨します。

参考: https://learn.microsoft.com/ja-jp/azure/firewall/rule-processing


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


## 5. 穴あけの誤解・注意点

- `*.xxx.com` で穴あけ（ワイルドカード許可）しても `xxx.com`（裸ドメイン）は自動で許可されない点に注意。
- FQDN にアンダースコア（`_`）は使えない（FQDN の仕様に準拠しないため、Azure Firewall の FQDN マッチに使えない）。
- Azure Firewall の穴あけは基本的に「送信元 → 送信先」方向の通信だけで問題ない。つまり HTTPS、RDP、SMB、NTP、ICMP 等のアウトバウンド／インバウンドを用途に応じて片方向で穴あけすれば十分であることが多い。

---

## 6. Azure Firewall の自動棚卸しスクリプト（PowerShell + az CLI）

以下は運用で役に立つ「未使用ルール検出」スクリプトの例（PowerShell）。Log Analytics（Azure Diagnostics）に送った Firewall ログを参照して、過去 N 日で使われていないルールを CSV 出力します。実行前に `az login` 済みであること。Cloud Shell ではセッション古くなることがあるので注意。

```powershell
# PowerShell + AZ CLI: 未使用の Azure Firewall ルール
# 実行前に az login 済みであること
$ErrorActionPreference = 'Stop'

# ===== 変数定義 （ここから）=====
# Log Analytics ワークスペースのリソースID
$LA_WS_RESOURCE_ID = "/subscriptions/xxxx/resourcegroups/xxxx/providers/microsoft.operationalinsights/workspaces/xxxx"

# 2つの Azure Firewall Policy のリソースID（2つ目は任意、空ならスキップ）
$FirewallPolicyResourceId1 = "/subscriptions/xxxx/resourceGroups/xxxx/providers/Microsoft.Network/firewallPolicies/xxxx"
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



# ===== 関数定義 （ここから）=====
function Ensure-AzCli {
  if (-not (Get-Command az -ErrorAction SilentlyContinue)) { throw 'Azure CLI (az) が見つかりません。インストールしてください。' }
  try { az account show --only-show-errors | Out-Null } catch { throw 'az にログインしてから実行してください (az login)。' }
}

# ARMトークン取得（必要時に再ログイン）
function Get-ArmToken {
  param([string]$Resource = 'https://management.azure.com/')
  try {
    $token = az account get-access-token --resource $Resource --query accessToken -o tsv --only-show-errors
    if ([string]::IsNullOrWhiteSpace($token)) { throw 'Empty token' }
    return $token.Trim()
  } catch {
    Write-Host "ARM トークン取得に失敗: $($_.Exception.Message)。再ログインを試みます..." -ForegroundColor Yellow
    try { az logout --only-show-errors | Out-Null } catch {}
    az login --scope "$Resource/.default" --only-show-errors | Out-Null
    $token = az account get-access-token --resource $Resource --query accessToken -o tsv --only-show-errors
    if ([string]::IsNullOrWhiteSpace($token)) { throw 'ARM トークンが取得できませんでした。Cloud Shellの再起動やローカル環境での実行を検討してください。' }
    return $token.Trim()
  }
}

function Parse-WorkspaceId {
  param([string]$WorkspaceResourceId)
  $parts = $WorkspaceResourceId -split '/'
  $subId = $parts[2]
  $rg    = $parts[4]
  $wsIdx = [Array]::IndexOf($parts, 'workspaces')
  if ($wsIdx -lt 0) { throw "ワークスペース名の解析に失敗しました: $WorkspaceResourceId" }
  $wsName = $parts[$wsIdx + 1]
  [pscustomobject]@{ SubscriptionId=$subId; ResourceGroup=$rg; WorkspaceName=$wsName }
}

function Ensure-OperationalInsightsProvider {
  param([string]$SubscriptionId)
  try { $state = (az provider show -n Microsoft.OperationalInsights --subscription $SubscriptionId --query "registrationState" -o tsv --only-show-errors) } catch { $state = $null }
  if (-not $state -or $state -ne 'Registered') {
    Write-Host "Registering resource provider Microsoft.OperationalInsights in subscription $SubscriptionId..." -ForegroundColor Yellow
    az provider register -n Microsoft.OperationalInsights --subscription $SubscriptionId --only-show-errors | Out-Null
    $attempts=0
    do {
      Start-Sleep -Seconds 5
      try { $state = (az provider show -n Microsoft.OperationalInsights --subscription $SubscriptionId --query "registrationState" -o tsv --only-show-errors) } catch { $state = $null }
      $attempts++
    } while ($state -ne 'Registered' -and $attempts -lt 12)
    Write-Host "Provider registration state: $state"
  }
}

# Policyメタ: リソースIDから直接パース（az resource show 不要）
function Parse-PolicyResourceId {
  param([string]$PolicyResourceId)
  if ([string]::IsNullOrWhiteSpace($PolicyResourceId)) { return $null }
  $m = [regex]::Match($PolicyResourceId, '^/subscriptions/([^/]+)/resourceGroups/([^/]+)/providers/Microsoft\.Network/firewallPolicies/([^/]+)$', 'IgnoreCase')
  if (-not $m.Success) { throw "Firewall Policy リソースIDの形式が不正です: $PolicyResourceId" }
  $subId = $m.Groups[1].Value
  $rg    = $m.Groups[2].Value
  $name  = $m.Groups[3].Value
  [pscustomobject]@{ Id=$PolicyResourceId; SubscriptionId=$subId; ResourceGroup=$rg; Name=$name; NameLower=$name.ToLower() }
}

# Policyに紐づく Firewall ResourceId を取得（ARM API + トークン明示）
function Get-FirewallIdsForPolicy {
  param([object]$PolicyMeta,[string]$ArmToken)
  if (-not $PolicyMeta) { return @() }
  $api = '2023-11-01'
  $uri = "https://management.azure.com/subscriptions/$($PolicyMeta.SubscriptionId)/providers/Microsoft.Network/azureFirewalls?api-version=$api"
  $resp = az rest --only-show-errors --method get --uri $uri --headers "Authorization=Bearer $ArmToken" -o json
  $payload = $resp | ConvertFrom-Json
  $items = @(); if ($payload.value) { $items = $payload.value } else { $items = $payload }
  $items | Where-Object { $_.properties.firewallPolicy.id -eq $PolicyMeta.Id } | ForEach-Object { $_.id.ToLower() }
}

# 定義済みルール（タイプ別: 'NetworkRule' or 'ApplicationRule'）（ARM API + トークン明示）
function Get-DefinedRulesForPolicyByType {
  param([object]$PolicyMeta,[ValidateSet('NetworkRule','ApplicationRule')] [string]$RuleType,[string]$ArmToken)
  if (-not $PolicyMeta) { return @() }
  $api = '2023-11-01'
  $uri = "https://management.azure.com/subscriptions/$($PolicyMeta.SubscriptionId)/resourceGroups/$($PolicyMeta.ResourceGroup)/providers/Microsoft.Network/firewallPolicies/$($PolicyMeta.Name)/ruleCollectionGroups?api-version=$api"
  $resp = az rest --only-show-errors --method get --uri $uri --headers "Authorization=Bearer $ArmToken" -o json
  $payload = $resp | ConvertFrom-Json
  $rcgs = @(); if ($payload.value) { $rcgs = $payload.value } else { $rcgs = $payload }
  $out = @()
  foreach ($rcg in ($rcgs | Where-Object { $_ })) {
    $rcs = $rcg.properties.ruleCollections
    if (-not $rcs) { continue }
    foreach ($rc in $rcs) {
      if ($rc.ruleCollectionType -ne 'FirewallPolicyFilterRuleCollection') { continue }
      $rules = $rc.rules; if (-not $rules) { continue }
      foreach ($rule in $rules) {
        if ($rule.ruleType -ne $RuleType) { continue }
        $out += [pscustomobject]@{
          PolicyKey          = $PolicyMeta.NameLower
          RuleCollectionKey  = ([string]$rc.name).ToLower()
          RuleKey            = ([string]$rule.name).ToLower()
          Policy             = $PolicyMeta.Name
          RuleCollection     = [string]$rc.name
          Rule               = [string]$rule.name
        }
      }
    }
  }
  $out | Group-Object PolicyKey, RuleCollectionKey, RuleKey | ForEach-Object { $_.Group[0] }
}

# OperationalInsights Query API（ARM 2017-10-01 のみ使用）
function Invoke-LogAnalyticsQueryArm2017 {
  param([string]$WorkspaceResourceId,[string]$Kql,[string]$Timespan,[string]$ArmToken)
  $ws = Parse-WorkspaceId -WorkspaceResourceId $WorkspaceResourceId
  Ensure-OperationalInsightsProvider -SubscriptionId $ws.SubscriptionId
  $api = '2017-10-01'
  $uri = "https://management.azure.com/subscriptions/$($ws.SubscriptionId)/resourceGroups/$($ws.ResourceGroup)/providers/Microsoft.OperationalInsights/workspaces/$($ws.WorkspaceName)/query?api-version=$api"
  $body = @{ query = $Kql; timespan = $Timespan } | ConvertTo-Json -Compress
  $resp = az rest --only-show-errors --method post --uri $uri --headers "Content-Type=application/json" --headers "Authorization=Bearer $ArmToken" --body $body -o json
  return ($resp | ConvertFrom-Json)
}

# 使用済みルール（Categoryとフィルタを指定）
function Get-UsedRulesArm {
  param([string]$WorkspaceResourceId,[string[]]$PolicyNamesLower,[string[]]$FirewallIdsLower,[int]$LookbackDays,[string[]]$Categories,[string]$ArmToken)
  $PolicyNamesLower = $PolicyNamesLower | Where-Object { -not [string]::IsNullOrWhiteSpace($_) }
  $FirewallIdsLower = $FirewallIdsLower | Where-Object { -not [string]::IsNullOrWhiteSpace($_) }
  $policyList   = ($PolicyNamesLower | ForEach-Object { '"' + $_ + '"' }) -join ', '
  $fwList       = ($FirewallIdsLower | ForEach-Object { '"' + $_ + '"' }) -join ', '
  $categoryList = ($Categories        | ForEach-Object { '"' + $_ + '"' }) -join ', '
  $kql = @"
AzureDiagnostics
| where isnotempty(RuleCollection_s) and isnotempty(Rule_s)
| where Category in ($categoryList)
| where TimeGenerated > ago(${LookbackDays}d)
| where tolower(Policy_s) in ($policyList) or tolower(ResourceId) in ($fwList)
| project PolicyKey = tolower(Policy_s), RuleCollectionKey = tolower(tostring(RuleCollection_s)), RuleKey = tolower(tostring(Rule_s))
| distinct PolicyKey, RuleCollectionKey, RuleKey
"@
  $timespan = "P${LookbackDays}D"
  try {
    $obj = Invoke-LogAnalyticsQueryArm2017 -WorkspaceResourceId $WorkspaceResourceId -Kql $kql -Timespan $timespan -ArmToken $ArmToken
  } catch {
    Write-Host "Log Analytics ARM Query 呼び出しに失敗しました: $($_.Exception.Message)" -ForegroundColor Yellow
    return @()
  }
  if (-not $obj.tables -or $obj.tables.Count -eq 0) { return @() }
  $rows = $obj.tables[0].rows
  $out = @()
  foreach ($r in $rows) { $out += [pscustomobject]@{ PolicyKey=[string]$r[0]; RuleCollectionKey=[string]$r[1]; RuleKey=[string]$r[2] } }
  $out | Group-Object PolicyKey, RuleCollectionKey, RuleKey | ForEach-Object { $_.Group[0] }
}

# ===== 関数定義 （ここまで）=====



# ===== メイン処理（ここから） =====
Ensure-AzCli
# ARMトークンを一度だけ取得し、全API呼び出しに使用（エラー抑制）
$armToken = Get-ArmToken -Resource 'https://management.azure.com/'

$policy1 = Parse-PolicyResourceId -PolicyResourceId $FirewallPolicyResourceId1
$policy2 = Parse-PolicyResourceId -PolicyResourceId $FirewallPolicyResourceId2

if ($policy1) { Write-Host "Policy1: $($policy1.Name) [RG: $($policy1.ResourceGroup), Sub: $($policy1.SubscriptionId)]" }
if ($policy2) { Write-Host "Policy2: $($policy2.Name) [RG: $($policy2.ResourceGroup), Sub: $($policy2.SubscriptionId)]" } else { Write-Host "Policy2: (未指定)" }

# Policyに紐づく Firewall ResourceId を取得（UsedRules フィルタで使用）
$fwIds1 = Get-FirewallIdsForPolicy -PolicyMeta $policy1 -ArmToken $armToken
$fwIds2 = Get-FirewallIdsForPolicy -PolicyMeta $policy2 -ArmToken $armToken
$fwIds  = @($fwIds1 + $fwIds2) | Sort-Object -Unique
Write-Host ("Firewalls found for policy: {0}" -f ($fwIds.Count))

# --- ネットワークルール ---
$usedNet = Get-UsedRulesArm -WorkspaceResourceId $LA_WS_RESOURCE_ID -PolicyNamesLower @($policy1?.NameLower, $policy2?.NameLower) -FirewallIdsLower $fwIds -LookbackDays $LookbackDays -Categories $NetworkCategories -ArmToken $armToken
$usedNet = $usedNet ?? @()
Write-Host ("Used network rules: {0}" -f ($usedNet.Count))

$definedNet1 = Get-DefinedRulesForPolicyByType -PolicyMeta $policy1 -RuleType 'NetworkRule' -ArmToken $armToken
$definedNet2 = Get-DefinedRulesForPolicyByType -PolicyMeta $policy2 -RuleType 'NetworkRule' -ArmToken $armToken
$definedNet  = @($definedNet1 + $definedNet2)
Write-Host ("Defined network rules: {0}" -f ($definedNet.Count))

$unusedNet = Compare-Object -ReferenceObject $definedNet -DifferenceObject $usedNet -Property PolicyKey, RuleCollectionKey, RuleKey -PassThru | Where-Object { $_.SideIndicator -eq '<=' }
$reportNet = $unusedNet | Select-Object @{n='Policy';e={$_.Policy}}, @{n='RuleCollection';e={$_.RuleCollection}}, @{n='RuleName';e={$_.Rule}}

Write-Host "\n===== Unused Network Rules (last $LookbackDays days) =====" -ForegroundColor Cyan
if ($reportNet.Count -gt 0) {
  $reportNet | Sort-Object Policy, RuleCollection, RuleName | Format-Table -AutoSize
  if ($ExportCsv) { $reportNet | Sort-Object Policy, RuleCollection, RuleName | Export-Csv -Path $OutNetworkCsv -NoTypeInformation -Encoding UTF8; Write-Host "CSV exported: $OutNetworkCsv" }
} else { Write-Host "未使用のネットワークルールは見つかりませんでした。" -ForegroundColor Green }

# --- アプリケーションルール ---
$usedApp = Get-UsedRulesArm -WorkspaceResourceId $LA_WS_RESOURCE_ID -PolicyNamesLower @($policy1?.NameLower, $policy2?.NameLower) -FirewallIdsLower $fwIds -LookbackDays $LookbackDays -Categories $AppCategories -ArmToken $armToken
$usedApp = $usedApp ?? @()
Write-Host ("Used application rules: {0}" -f ($usedApp.Count))

$definedApp1 = Get-DefinedRulesForPolicyByType -PolicyMeta $policy1 -RuleType 'ApplicationRule' -ArmToken $armToken
$definedApp2 = Get-DefinedRulesForPolicyByType -PolicyMeta $policy2 -RuleType 'ApplicationRule' -ArmToken $armToken
$definedApp  = @($definedApp1 + $definedApp2)
Write-Host ("Defined application rules: {0}" -f ($definedApp.Count))

$unusedApp = Compare-Object -ReferenceObject $definedApp -DifferenceObject $usedApp -Property PolicyKey, RuleCollectionKey, RuleKey -PassThru | Where-Object { $_.SideIndicator -eq '<=' }
$reportApp = $unusedApp | Select-Object @{n='Policy';e={$_.Policy}}, @{n='RuleCollection';e={$_.RuleCollection}}, @{n='RuleName';e={$_.Rule}}

Write-Host "\n===== Unused Application Rules (last $LookbackDays days) =====" -ForegroundColor Cyan
if ($reportApp.Count -gt 0) {
  $reportApp | Sort-Object Policy, RuleCollection, RuleName | Format-Table -AutoSize
  if ($ExportCsv) { $reportApp | Sort-Object Policy, RuleCollection, RuleName | Export-Csv -Path $OutAppCsv -NoTypeInformation -Encoding UTF8; Write-Host "CSV exported: $OutAppCsv" }
} else { Write-Host "未使用のアプリケーションルールは見つかりませんでした。" -ForegroundColor Green }

# ===== メイン処理（ここまで） =====
```

> 注意: スクリプト中のリソース ID やサブスクリプションはダミーです。実行前に環境に合わせて書き換えてください。


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

