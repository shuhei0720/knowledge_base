# はじめに
Azureで自動化する場合の一つの方法として、アラートルール＋Azure Automationという構成を使用できます。
今回はその構成の構成方法をご説明します。
ここでは、以下の知識を学ぶことが目的とします。
- Azure Automationの使い方
- PowerShell
- KQLでのログ検索
- アラートルールの使い方

# 今回の構成図
今回デモで作成する構成を以下に示します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e411cade-0b83-438d-953e-7c5ecb39ee6e.png)

アクティビティログから仮想ネットワークが作成されたログをアラートルールで検知し、Automationを起動して、「Protect」と名前に含まれるサブネットのIP範囲をIPグループリソースに記録します。

# デモ環境の作成

## 1. サブスクリプションのアクティビティログを設定
アラート発砲のログ収集のための設定を行います。
- AzureポータルからLog Analyticsワークスペースを作成
- Azureポータルのサブスクリプションに移動→「アクティビティログ」→「診断設定」から先ほど作成したLog Analyticsワークスペースにログを送信する
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/1e9376b3-16e2-4dd3-a415-40c71f077106.png)

## 2. Azure Automationの作成
Protectという名前のサブネットを走査して、IP範囲を収集し、IPグループに設定するPowerShellスクリプトを使用して、Azure Automationを作成します。
- AzureポータルからAutomation アカウントを作成
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/2ea72faf-c161-483b-b473-d335cb0df0b2.png)

- Automationアカウントのシステム割り当てIDにTenant Rootの「ネットワーク共同作成者」と「管理グループ閲覧者」を付与
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/7905f30a-1fd9-4e76-aeab-2a85b33dc288.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/3e54262f-0371-4b40-a1aa-73f2b375cd15.png)

- Automation変数を設定

| 変数名 | タイプ | 役割 | 設定例 | 注意事項 |
| ---- | ---- | ---- | ---- | ---- |
|MgmtGroupId|文字列|走査対象の管理グループID（再帰で配下のサブスクリプションを取得）|Production|管理グループの「ID（Name）」を指定。表示名ではない点に注意。|
|IpGroupSubscriptionId|文字列|IPグループを作成・更新するサブスクリプションID|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|Runbookの更新先。|
|IpGroupRg|文字列|IPグループのリソースグループ名|test-auto|事前にRGが存在していること。|
|IpGroupName|文字列|IPグループのリソース名|ipg-protected|既存があれば更新、なければ新規作成。|
|Location|文字列|IPグループのリージョン|japaneast|既存があれば既存locationを使用。新規時のみ必須。|
|ProtectPattern|文字列|保護サブネット判定の正規表現|protect|大文字小文字は区別しない。|
|IncludeIPv6|文字列|IPv6 CIDRも同期対象に含めるか|false|falseでIPv6除外。|

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/9692f045-5859-4496-9328-be54bf147175.png)

- Runbookの作成
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/26865c73-0439-4b09-a34b-4c347ceca217.png)

- Runbookに以下のコードを貼り付け→「保存」→「公開」
※コードの構文はコードの内のコメント解説を参照

```powershell
<#
概要（ランブックの目的）
- 管理グループ配下のサブスクリプションを列挙し、各VNetの「Protect」を含む名前のサブネットCIDR（IPv4/IPv6）を収集して、指定のIPグループへ同期（PUT）します。

PowerShell構文の観点（本スクリプトでよく出るポイント）
- paramブロック: 実行引数の型指定と既定値設定が可能。
- [string]/[bool]などの型リテラル: PowerShellでは柔軟に型変換されるが、意図しない変換を避けるため型指定は有効。
- -match（正規表現） vs -like（ワイルドカード）: -matchはRegex、-likeは*や?のワイルドカード。
- -not 演算子: 真偽を反転。空文字列/ $null は偽になるため存在チェックに利用されることが多い。
- try/catch と $ErrorActionPreference: エラーを例外化し、catchで制御（止める/握りつぶす）できる。
- ハッシュテーブル @{ ... } と ConvertTo-Json: PowerShellのハッシュをJSONへ直列化。ネストが深い場合は -Depth を調整。
- .NET型の併用: System.Net.Http.HttpClient や System.UriBuilder を直接使ってREST呼び出し。
#>

# 1) 実行引数（param）: 型指定と既定値を設定
# PowerShellではparamブロックで受け取る引数の型を指定できます。既定値がある場合は「= 値」で指定。
param(
  [string] $MgmtGroupId,           # 管理グループID（例: mg-xxxxxxxx）。未指定時はAutomation変数から取得（後段）
  [string] $IpGroupSubscriptionId, # IPグループが存在/作成されるサブスクリプションID（GUID）
  [string] $IpGroupRg,             # IPグループのリソースグループ名
  [string] $IpGroupName,           # IPグループ名
  [string] $Location = "japaneast", # 既存IPグループがない場合の作成リージョン。既定は "japaneast"
  [string] $ProtectRegex = "(?i)protect", # サブネット名に対する正規表現。(?i) は大文字小文字無視
  [bool]   $IncludeIPv6 = $false   # IPv6CIDRを含めるか。bool型、既定false
)

# 2) エラー動作設定
# Stopにすることで、非終端エラーも例外として扱われ、try/catch に入ります。
$ErrorActionPreference = 'Stop'

# 3) ユーティリティ関数定義
# Get-VarOrDefault: Automation変数を読み取り、未設定や空の場合はFallbackを返す
function Get-VarOrDefault {
  param([string]$Name, [object]$Fallback)
  try {
    # Get-AutomationVariable は変数未存在時にエラーになる場合があるため、try/catchで保護
    $v = Get-AutomationVariable -Name $Name -ErrorAction Stop
    # 取得値がnullでも空文字でもない場合はそのまま返す。ToString().Trim()で空白も除外
    if ($null -ne $v -and $v.ToString().Trim() -ne "") { return $v } else { return $Fallback }
  } catch {
    # 変数が存在しない等のエラー時はFallbackを返す
    return $Fallback
  }
}

# Build-Uri: ベースURL＋パス＋api-versionから完全なURL文字列を生成
# .NETのSystem.UriBuilderを使い、クエリに api-version を付与します。
function Build-Uri {
  param([string]$baseUrl,[string]$path,[string]$ver)
  $u = [System.UriBuilder]::new("$baseUrl$path")
  $u.Query = "api-version=$ver"
  return $u.Uri.AbsoluteUri
}

# 4) 入力パラメータのフォールバック（Automation変数優先）
# -not は存在チェックに便利（null/空は偽）。未指定なら同名のAutomation変数を参照。
if (-not $MgmtGroupId)           { $MgmtGroupId           = Get-VarOrDefault -Name 'MgmtGroupId'           -Fallback $MgmtGroupId }
if (-not $IpGroupSubscriptionId) { $IpGroupSubscriptionId = Get-VarOrDefault -Name 'IpGroupSubscriptionId' -Fallback $IpGroupSubscriptionId }
if (-not $IpGroupRg)             { $IpGroupRg             = Get-VarOrDefault -Name 'IpGroupRg'             -Fallback $IpGroupRg }
if (-not $IpGroupName)           { $IpGroupName           = Get-VarOrDefault -Name 'IpGroupName'           -Fallback $IpGroupName }
if (-not $Location)              { $Location              = Get-VarOrDefault -Name 'Location'              -Fallback $Location }

# ProtectRegexは既定値のままのときだけ、別名変数 ProtectPattern を参照
# 文字列比較(-eq)は完全一致。正規表現ではありません。
if ($ProtectRegex -eq "(?i)protect") { $ProtectRegex = Get-VarOrDefault -Name 'ProtectPattern' -Fallback $ProtectRegex }

# IncludeIPv6は文字列/真偽両対応のため、型で分岐
# -is 演算子で型チェック、-matchで正規表現一致。^(?i:true|1|yes)$ は全体一致・大小無視。
$incV6Var = Get-VarOrDefault -Name 'IncludeIPv6' -Fallback $IncludeIPv6
if ($incV6Var -is [string]) { $IncludeIPv6 = ($incV6Var -match '^(?i:true|1|yes)$') } else { $IncludeIPv6 = [bool]$incV6Var }

# 5) 認証（マネージドID）
# Connect-AzAccount -Identity はAutomationアカウントのManaged Identityでログインするためのコマンドです。
Connect-AzAccount -Identity | Out-Null  # Out-Nullは出力を捨てる（表示しない）

# 6) 管理API呼び出し準備（Bearerトークン＋HttpClient）
# ResourceUrlに管理エンドポイントを指定し、トークン文字列を取得
$token = (Get-AzAccessToken -ResourceUrl "https://management.azure.com/").Token
$baseUrl = "https://management.azure.com"         # 文字列は二重引用符で展開可能（$変数の埋め込み）。単引用符は展開しない。
$apiMgmtVer = '2021-04-01'                        # 単引用符はリテラル文字列（展開なし）。バージョン固定。

# .NETのHttpClientを生成し、Authorization/Acceptヘッダーを設定
$client = [System.Net.Http.HttpClient]::new()
$client.DefaultRequestHeaders.Authorization = New-Object System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", $token)
$client.DefaultRequestHeaders.Accept.Clear()
$client.DefaultRequestHeaders.Accept.Add([System.Net.Http.Headers.MediaTypeWithQualityHeaderValue]::new("application/json"))

# 7) 管理グループ配下のサブスクリプションIDを列挙
# 配列初期化: @() は空の配列。+= で要素追加。PowerShellの配列は可変です。
$subIds = @()

# descendants APIのURLを構築し、ページング（nextLink）を辿る
$mgDescUri = Build-Uri -baseUrl $baseUrl -path "/providers/Microsoft.Management/managementGroups/$MgmtGroupId/descendants" -ver $apiMgmtVer
$nextUri = $mgDescUri
while ($nextUri) {  # whileループ: 条件が真の間繰り返し
  $resp = $client.GetAsync($nextUri).GetAwaiter().GetResult()  # 非同期Taskを同期的に待機
  $body = $resp.Content.ReadAsStringAsync().GetAwaiter().GetResult()
  if (-not $resp.IsSuccessStatusCode -or -not $body) { break } # 成功/ボディ有りを確認。失敗ならbreakで脱出
  $obj = $body | ConvertFrom-Json  # JSON文字列をPowerShellオブジェクトへ変換（パイプ | は前段の出力を次に渡す）
  foreach ($ent in $obj.value) {   # foreachループ: コレクションの各要素を走査
    # id文字列から /subscriptions/{GUID} を正規表現で抽出
    # [regex]::Match は .NETのRegex。([0-9a-fA-F-]{36}) はGUID長（ハイフン含む36文字）に概ね対応
    $m = [regex]::Match([string]$ent.id, '/subscriptions/([0-9a-fA-F-]{36})')
    if ($m.Success) { $subIds += $m.Groups[1].Value }  # マッチ成功なら1番目のキャプチャグループを配列に追加
  }
  $nextUri = $obj.nextLink  # ページング用の次リンク。存在しなければ$nullとなり、whileを抜ける
}

# descendantsで取得ゼロのとき、subscriptions APIでフォールバック
if ($subIds.Count -eq 0) {  # .Countは要素数。-eq は等価比較。
  $mgSubsUri = Build-Uri -baseUrl $baseUrl -path "/providers/Microsoft.Management/managementGroups/$MgmtGroupId/subscriptions" -ver $apiMgmtVer
  $resp2 = $client.GetAsync($mgSubsUri).GetAwaiter().GetResult()
  $body2 = $resp2.Content.ReadAsStringAsync().GetAwaiter().GetResult()
  if ($resp2.IsSuccessStatusCode -and $body2) {  # -and は論理AND
    $obj2 = $body2 | ConvertFrom-Json
    foreach ($v in $obj2.value) {
      $m2 = [regex]::Match([string]$v.id, '/subscriptions/([0-9a-fA-F-]{36})')
      if ($m2.Success) { $subIds += $m2.Groups[1].Value }
    }
  }
}

# null除外（Where-Object）、ユニーク化（Sort-Object -Unique）
# Where-Object { $_ } は真偽判定（$_ は現在の要素）。Sort-Object -Unique は重複排除とソートを同時に行う。
$subIds = $subIds | Where-Object { $_ } | Sort-Object -Unique
if ($subIds.Count -eq 0) { $client.Dispose(); return }  # 0件ならHttpClientを破棄して早期終了

# 8) ProtectサブネットCIDRの収集
# HashSet[string] は高速な重複排除に適した .NET コレクション。Addの戻り値はbool（追加されたか）。
$protectedPrefixes = New-Object System.Collections.Generic.HashSet[string]
foreach ($sid in $subIds) {
  try {
    # Set-AzContext: 以降のAzコマンドの対象サブスクリプションを切り替える
    Set-AzContext -Subscription $sid | Out-Null

    # VNet一覧を取得。-ErrorAction SilentlyContinue はエラーを表示せず続行（例外にしない）
    $vnets = Get-AzVirtualNetwork -ErrorAction SilentlyContinue
    foreach ($vnet in $vnets) {
      foreach ($subnet in $vnet.Subnets) {
        # -match は正規表現一致。既定の $ProtectRegex は (?i)protect（大小無視の「protect」）
        if ($subnet.Name -match $ProtectRegex) {
          # AddressPrefixes（複数）か AddressPrefix（単一）かは環境/バージョンで異なる場合があるため分岐
          if ($subnet.AddressPrefixes) {
            foreach ($p in $subnet.AddressPrefixes) {
              $pt = $p.Trim()  # Trim()は前後空白を除去
              # IPv6はコロン(:)を含むことが多いため、-like '*:*' で簡易判定。IncludeIPv6=falseなら除外
              if (-not $IncludeIPv6 -and $pt -like '*:*') { continue }
              [void]$protectedPrefixes.Add($pt)  # [void]で戻り値（true/false）を無視
            }
          } elseif ($subnet.AddressPrefix) {  # 単一プレフィックス
            $pt = $subnet.AddressPrefix.Trim()
            if (-not $IncludeIPv6 -and $pt -like '*:*') {  # IPv6除外条件
              # 何もしない（除外）
            } else {
              [void]$protectedPrefixes.Add($pt)
            }
          }
        }
      }
    }
  } catch {
    # サブスクリプション単位の権限不足やAPIエラーを握りつぶして続行
    # 運用方針として、全体の収集を優先する設計（詳細ログは必要に応じて追加）
  }
}

# 収集ゼロなら終了。DisposeでHttpClientのリソースを解放。
if ($protectedPrefixes.Count -eq 0) { $client.Dispose(); return }

# HashSet → 配列へ展開し、null除外＋ソート。@(...) は配列化。
$sortedNew = [string[]](@($protectedPrefixes | Where-Object { $_ } | Sort-Object))

# 9) IPグループの作成/更新（PUT）
# 反映先サブスクリプションへコンテキストを切り替え
Set-AzContext -Subscription $IpGroupSubscriptionId | Out-Null

# 管理APIトークンを再取得（トークンはスコープや寿命の都合で取り直すのが安全）
$token = (Get-AzAccessToken -ResourceUrl "https://management.azure.com/").Token
$client.DefaultRequestHeaders.Authorization = New-Object System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", $token)

# ipGroups リソースのAPIバージョンとパスを定義
$apiIpgVer = '2023-06-01'
$ipgPath = "/subscriptions/$IpGroupSubscriptionId/resourceGroups/$IpGroupRg/providers/Microsoft.Network/ipGroups/$IpGroupName"
$ipgUri  = Build-Uri -baseUrl $baseUrl -path $ipgPath -ver $apiIpgVer

# 既存IPグループをGETしてlocationを確認（既存がある場合はそれを使用）
$respGet = $client.GetAsync($ipgUri).GetAwaiter().GetResult()
$bodyGet = $respGet.Content.ReadAsStringAsync().GetAwaiter().GetResult()
$existingLocation = $null
if ($respGet.IsSuccessStatusCode -and $bodyGet) { 
  try { 
    $existingLocation = ($bodyGet | ConvertFrom-Json).location 
  } catch { 
    # locationの取得に失敗しても致命的ではないため続行
  } 
}

# 使用するロケーションを決定（既存locationが優先、無ければ入力Location）
$useLocation = if ($existingLocation) { $existingLocation } else { $Location }  # if式は値を返す（PowerShellの特徴）
if (-not $useLocation) { $client.Dispose(); throw "Locationが特定できません。" }  # throwで例外送出

# PUTペイロードをハッシュテーブルで作成 → ConvertTo-Jsonで直列化
# -Depth はネストが深い場合に必要。ここでは properties 内の配列まで十分に含めるため 6 を指定。
$payloadObject = @{ location = $useLocation; properties = @{ ipAddresses = $sortedNew } }
$payloadJson   = $payloadObject | ConvertTo-Json -Depth 6

# StringContentでHTTPボディを生成。エンコーディングUTF8、Content-Type: application/json
$content = New-Object System.Net.Http.StringContent($payloadJson, [System.Text.Encoding]::UTF8, 'application/json')

# PUT実行（作成/更新）。IPグループは存在しなければ作成、あれば上書き（完全入れ替え）
$respPut = $client.PutAsync($ipgUri, $content).GetAwaiter().GetResult()
$bodyPut = $respPut.Content.ReadAsStringAsync().GetAwaiter().GetResult()
if (-not $respPut.IsSuccessStatusCode) {
  # エラー応答のJSONから code / message を抜き出して詳細な例外を作成
  $code = $null; $msg = $null
  if ($bodyPut) {
    try { 
      $errObj = $bodyPut | ConvertFrom-Json
      $code = $errObj.error.code
      $msg = $errObj.error.message
    } catch { }
  }
  if ($code -or $msg) { 
    $client.Dispose(); throw "IPグループ更新失敗: $code $msg" 
  } else { 
    $client.Dispose(); throw "IPグループ更新失敗: HTTP $([int]$respPut.StatusCode)" 
  }
}

# 最後にHttpClientを明示的にDispose（リソース解放）。
$client.Dispose()
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/80aafd92-8d9c-4f54-baa8-fe83bcc81f3a.png)

## 3.アラートルールの作成
ログからVNetの作成を検知して、Automationを起動するアラートルールを作成します。
- アラートルールの作成→リソースの選択からログを送信しているLog Analyticsワークスペースを選択
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/890eaf30-e278-4c12-b0db-a682aab1095c.png)
- 条件で「カスタムログ検索」を指定し、以下のKQLを入力
※KQLはSQLのような構文でログを検索する言語であり、今回は仮想ネットワークのWRITE/DELETE、サブネットのWRITE/DELETEを対象にログを検索している。
※一つのログに対し、「Start」「Accept」「Success」と複数行が出てしまうため、「Start」のみを対象にしている。
```sql
AzureActivity
| where OperationNameValue contains "MICROSOFT.NETWORK/VIRTUALNETWORKS/WRITE" or OperationNameValue contains "MICROSOFT.NETWORK/VIRTUALNETWORKS/DELETE" or OperationNameValue contains "MICROSOFT.NETWORK/VIRTUALNETWORKS/SUBNETS/WRITE" or OperationNameValue contains "MICROSOFT.NETWORK/VIRTUALNETWORKS/SUBNETS/DELETE"
| where ActivityStatusValue contains "Start"
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/80c4fc44-e624-47ee-933b-1893b46e73ac.png)

- 条件の「測定」と「アラートロジック」を入力する

| 項目名 | 説明 | 設定例 |
| --- | --- | --- |
|メジャー|集計対象のテーブルの行または数値列|テーブルの行|
|集計の種類|集計粒度でデータポイントに適用する集計の種類|カウント|
|集計の粒度|データポイントが集計の種類によってグループ化される期間|5分|
|演算子|メトリックの値をしきい値と比較するための演算子を選択|次の値より大きい|
|しきい値|数を入力|0|
|評価の頻度|アラート ルールの実行頻度を選択|1分|

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/288857fe-5c6f-4d6c-8253-fd2220063668.png)

- アクションで「アクショングループの作成」からアクショングループを作成
アクションは先ほど作成したRunbookを指定する
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/1317a217-6467-4001-9041-70a190871de2.png)

- アラートルールを作成
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/1f82d67c-36d0-46ab-992e-e8d3689bb519.png)

# 動作テスト
作成した自動化構成のテストを行います。
※アラートルールを作成してから有効になるまで10分程度のタイムラグがあります。

- 「protectsubnet」を含む仮想ネットワークを作成
 ※protectsubnetのIP範囲はここでは「10.0.200.0/24」とする。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/103b6153-f59d-4ee5-acd8-72f1574faa93.png)

- しばらく待って、アラートが発砲されることを確認
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/598eb102-f762-429b-99a3-207a60dea1ff.png)

- Automationアカウント→「ジョブ」からAutomation Runbookが起動されていることを確認
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f8af2d15-7ddb-4769-ae12-e1d42c830bdd.png)

- IPグループに先ほど作成したprotectsubnetのIP範囲(10.0.200.0/24)が追加されていることを確認
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/2a6e1a43-9f45-4c3d-a7a5-57078ed709f1.png)










