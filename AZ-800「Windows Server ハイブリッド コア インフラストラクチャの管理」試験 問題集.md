- この問題集は、AZ-800「Windows Server ハイブリッド コア インフラストラクチャの管理」試験の問題集です。
- 試験の概要を以下のURLをご参照くだささい。
https://learn.microsoft.com/ja-jp/credentials/certifications/windows-server-hybrid-administrator
- まずは、試験概要にあるラーニングパスを終了してからこの問題集を実施することをお勧めしますが、この問題集だけでも合格することは可能です。

- 筆者は、MSのラーニングパスとこの問題集の問題演習の二つで869/1000(700以上で合格)のスコアで試験に合格しました。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/df7cc89e-fe1c-4180-9b7b-c92982281a30.png)

- この問題集は、問題/解答/解説で構成されており、解説では答えの根拠を丁寧に解説しております。根拠となる公式ドキュメントも示しているので、理解できない場合は参照することをお勧めします。

- 問題については、以下のサイトを参照しました。
https://www.freecram.net/exam/AZ-800-administering-windows-server-hybrid-core-infrastructure-e13937.html

## 問題1
ドメインユーザーが、共有フォルダー \\SRV1\Data に実行可能ファイル（例: .exe）を保存できないようにしなければなりません。
なお、その他のファイルは保存できる必要があります。
タスクを完了するために、必要なコンピューターにサインインしてください。

## 解答1
Windows Server の「ファイル サーバー リソース マネージャー（FSRM）」の「ファイル スクリーン管理」を使用して、対象フォルダーにアクティブ スクリーニングを構成します。

手順（例: SRV1 で実施）
- 1 ) 管理者として SRV1 にサインイン
- 2 ) 未導入の場合は FSRM を追加（サーバーマネージャー > 役割と機能の追加 > ファイル サービスと記憶域サービス > ファイルおよび iSCSI サービス > ファイル サーバー リソース マネージャー）
- 3 ) 「ファイル サーバー リソース マネージャー」を起動し、左ペインの「ファイル スクリーン管理 > ファイル スクリーン」を選択
- 4 ) 右ペイン（または右クリック）から「ファイル スクリーンの作成」をクリック
- 5 ) 「ファイル スクリーンのパス」に 共有 \\SRV1\Data の実体フォルダー（例: D:\Data）を指定（このフォルダーとすべてのサブフォルダーに適用されます）
- 6 ) 「ファイル スクリーンのプロパティをどのように構成しますか」で「カスタム ファイル スクリーンのプロパティを定義」>「カスタム プロパティ」
- 7 ) 「スクリーニングの種類」は「アクティブ」を選択（保存自体をブロック）
- 8 ) 「ファイル グループ」で「実行可能ファイル（Executable Files）」を選択（または *.exe を含むカスタム グループを作成）
- 9 ) 必要に応じて「電子メール」「イベント ログ」「コマンド」「レポート」で通知を構成
- 10 ) OK で閉じ、「作成」をクリック。求められたらテンプレートとして保存しておくと再利用に便利

これにより、\\SRV1\Data には .exe などの実行可能ファイルの保存が拒否され、それ以外のファイルは保存可能な状態になります。

## 解説1
- FSRM の「ファイル スクリーン管理」では、ユーザーが保存できるファイルの種類を制御できます。Microsoft Learn は次のように説明しています:
  - “Create file screens to control the types of files that users can save...” （ユーザーが保存できるファイルの種類を制御するためにファイル スクリーンを作成できる）
  - “Active screening prevents users from saving files that are members of blocked file groups and generates notifications when users try to save unauthorized files.”（アクティブ スクリーニングは、ブロック対象グループに属するファイルの保存を防ぎ、未承認ファイルの保存試行時に通知を生成する）
- また、作成時に指定したフォルダー「およびそのサブフォルダー」に適用されるため、共有配下全体を一括で制御できます。

参考: Create a File Screen（Microsoft Learn） https://learn.microsoft.com/windows-server/storage/fsrm/create-file-screen

## 問題2
あなたのネットワークには、Active Directory Domain Services (AD DS) ドメイン contoso.com があり、ドメインには Server1 という DNS サーバーが含まれています。
Server1 には、DNSSEC で署名済みの DNS ゾーン fabrikam.com がホストされています。
ドメイン内のすべてのメンバー サーバーが、fabrikam.com 名前空間に対して DNSSEC 検証を実行するようにする必要があります。何をすべきですか？

A. Server1 で Add-DnsServerTrustAnchor コマンドレットを実行する。
B. 各メンバー サーバーで Add-DnsServerTrustAnchor コマンドレットを実行する。
C. グループ ポリシー オブジェクト (GPO) から、Name Resolution Policy Table (NRPT) にルールを追加する。
D. グループ ポリシー オブジェクト (GPO) から、Network List Manager のポリシーを変更する。

## 解答2
C

## 解説2
- クライアント（ここではメンバー サーバー）に特定の名前空間で DNSSEC 検証を必須化するには、GPO で NRPT にルールを配布し、その名前空間（fabrikam.com）に対して「DNSSEC を要求（検証）」を有効化します。
- Microsoft 公式ドキュメントでは、NRPT の DNSSEC 設定を Group Policy で構成し、指定した名前空間に対して DNSSEC を要求できることが示されています（意訳）。
  - 参考: DNS Security Extensions (DNSSEC) | Microsoft Learn
    - https://learn.microsoft.com/windows-server/networking/dns/dnssec/dns-security-extensions-dnssec
  - 参考: Deploy DNSSEC | Microsoft Learn（NRPT を用いてクライアント側で DNSSEC を必須にする手順の記載あり）
    - https://learn.microsoft.com/windows-server/networking/dns/dnssec/deploy-dnssec
- A/B は DNS サーバー側の信頼アンカー設定であり、クライアントの検証強制には不適切。D はネットワーク場所の分類に関するポリシーで DNSSEC とは無関係。

## 問題3
Active Directory ドメインに、Server1 という名前のファイル サーバーがあります。Server1 は Windows Server を実行しており、次の共有フォルダーを含んでいます。

|Share 名|パス|
|---|---|
|Users|D:\Users|
|Accounts|D:\Dept\Accounts|
|Marketing|D:\Dept\Marketing|
|CustomerService|D:\Dept\CustomerService|

ユーザーがネットワークにサインインすると、次のネットワーク ドライブが割り当てられます。
- H: は \\server1\users\%UserName% にマップ
- G: は \\server1\%Department% にマップ

Server1 上でユーザーが消費できる容量を制限する必要があります。ソリューションは次の要件を満たす必要があります。
- H: ドライブで、ユーザーが 5 GB を超えて使用できないようにする
- G: ドライブで、Accounts 部門のユーザーが 10 GB を超えて使用できないようにする
- G: ドライブで、Marketing 部門のユーザーが 15 GB を超えて使用できないようにする
- G: ドライブで、Customer Service 部門のユーザーが 2 GB を超えて使用できないようにする
- 管理作業を最小限に抑える

何を使用すべきですか?
A. File Server Resource Manager (FSRM) のクォータ  
B. ストレージ階層化  
C. NTFS ディスク クォータ  
D. Group Policy Preferences

## 解答3
A. File Server Resource Manager (FSRM) のクォータ

## 解説3
FSRM のクォータは「フォルダー単位」で容量を制限でき、クォータ テンプレートと「自動適用クォータ」により、D:\Users 配下の各ユーザー フォルダーに 5 GB のクォータを一括適用できます。また、部門用共有 (D:\Dept\Accounts / Marketing / CustomerService) にはフォルダーごとに 10 GB / 15 GB / 2 GB のクォータを設定できます。これにより要件をすべて満たしつつ、テンプレートと自動適用で管理負担を最小化できます。NTFS ディスク クォータはボリューム単位・所有者ベースであり、同一ボリューム内のフォルダーごとに異なる上限を柔軟に分ける用途には不向きです。ストレージ階層化や Group Policy Preferences では容量制限要件を満たせません。

- 根拠 (Microsoft 公式ドキュメント):
  - FSRM の概要: 「ボリュームまたはフォルダーに対して許可される領域の制限 (クォータ) を作成できる」「特定フォルダーとそのサブフォルダーにクォータを自動適用できる」
    https://learn.microsoft.com/windows-server/storage/fsrm/fsrm-overview
  - クォータの作成: 「テンプレートの自動適用により、既存および新規のサブフォルダーにクォータを作成」
    https://learn.microsoft.com/windows-server/storage/fsrm/fsrm-create-quota
  - クォータ管理 (テンプレート/フォルダー単位のクォータ設定の手順):
    https://learn.microsoft.com/windows-server/storage/fsrm/fsrm-quota-management

## 問題4
オンプレミスの Active Directory Domain Services (AD DS) ドメイン contoso.com があり、Azure AD Connect を用いて Azure AD と同期しています。
contoso.com に対して Password Protection（パスワード保護）を有効化しました。
ユーザーが自分のパスワードに「contoso」という単語を含められないようにする必要があります。
何を使用すべきですか？

A. the Azure Active Directory admin center  
B. Active Directory Users and Computers  
C. Synchronization Service Manager  
D. Windows Admin Center

## 解答4
A. the Azure Active Directory admin center

## 解説4
- Azure AD Password Protection では、カスタムの禁止パスワード リストを Azure ポータル（Azure Active Directory 管理センター）で定義できます。ここに「contoso」を追加することで、その語を含むパスワードを拒否できます。公式: Configure custom banned passwords（Azure portal で設定）
  - https://learn.microsoft.com/azure/active-directory/authentication/howto-password-ban-bad-configure
- オンプレ AD DS に Azure AD Password Protection を展開すると、ドメイン コントローラーは Azure AD のグローバル/カスタム禁止リストをダウンロードしてポリシーを適用します（変形や一般的な置換も評価）。公式: What is Azure AD password protection? / On-premises AD DS support
  - https://learn.microsoft.com/azure/active-directory/authentication/concept-password-ban-bad
  - https://learn.microsoft.com/azure/active-directory/authentication/concept-password-ban-bad-on-premises
- 上記より、禁止語の管理は「Azure Active Directory admin center」で行うのが正解です（ADUCや同期ツール、WACではカスタム禁止語は構成できません）。

## 問題5
あなたは、fabrikam.com ドメインの Sales 組織単位 (OU) に所属する一部ユーザーのプロパティを再構成するよう依頼されました。
変更を行うために PowerShell で使用すべきコマンドレットは次のうちどれですか？

A. New-ADuser  
B. Set-ADuser  
C. Get-ADuser  
D. Change-ADuser

## 解答5
B. Set-ADUser

## 解説5
- ユーザーオブジェクトの属性変更は Set-ADUser を使用します（例: 表示名、部署、電話番号、アカウントオプションなど）。公式ドキュメントには「Modifies an Active Directory user.」と明記されています。
  - Set-ADUser（Microsoft Learn）: https://learn.microsoft.com/powershell/module/addsadministration/set-aduser
- 参考：New-ADUser は新規ユーザー作成、Get-ADUser は取得のためのコマンドレットです。Change-ADuser というコマンドレットは提供されていません。
  - New-ADUser: https://learn.microsoft.com/powershell/module/addsadministration/new-aduser
  - Get-ADUser: https://learn.microsoft.com/powershell/module/addsadministration/get-aduser
- OU 配下の対象に限定して更新する場合は、Get-ADUser -SearchBase "OU=Sales,DC=fabrikam,DC=com" で抽出し、Set-ADUser にパイプして更新できます（必要に応じて）。

# 問題6
あなたはシアトルのサイトで、DC3 という名前のドメイン コントローラーを昇格させる計画があります。
DC3 が「午後8時から午前6時の間に限り」、かつ「DC1 と DC2 とだけ」レプリケーションを行うように構成する必要があります。
必要なコンピューターにサインインして、このタスクを完了しなさい。

# 解答6
1.（必要に応じて）Seattle サイトと、DC1・DC2 が存在するサイトの間にサイト リンクを作成する。既にサイト リンクがある場合はこの手順をスキップする。
2. Active Directory サイトとサービスを開く（[スタート] > [管理ツール] > [Active Directory サイトとサービス]）。
3. コンソール ツリーで [サイト間トランスポート]（通常は [IP]）を展開し、対象のサイト リンクを右クリックして [プロパティ] を開く。
4. [スケジュールの変更] をクリックする。
5. 20:00〜06:00 の時間帯を選択して「レプリケーションを許可」に設定し、それ以外の時間帯は「レプリケーションを許可しない」に設定する。
6.（必要に応じて）Seattle サイトが他のサイトとリンクしていないことを確認する。DC3 が DC1/DC2 のみとレプリケートすることをより確実にする必要がある場合は、DC1/DC2 を優先ブリッジヘッド サーバーに指定する、または手動の接続オブジェクトを構成する。

|項目|設定|
|---|---|
|対象サイト リンク|Seattle ↔ DC1/DC2 のサイト|
|レプリケーション可|20:00〜06:00|
|レプリケーション不可|06:00〜20:00|

# 解説6
- サイト リンクは、KCC がサイト間レプリケーション接続を作成するために用いる論理的なパスで、スケジュールを設定して「いつ」サイト間レプリケーションが許可されるかを制御できます。これにより、Seattle サイトと DC1/DC2 のサイト間のレプリケーションを 20:00〜06:00 に限定できます。
- 2 つのサイトがサイト リンクで接続されると、レプリケーション システムは各サイト内の特定のドメイン コントローラー間に接続（ブリッジヘッド サーバー）を自動作成します。Seattle サイトが DC1/DC2 のサイトにのみリンクされていれば、DC3 はそのサイト（DC1/DC2 がいるサイト）とだけレプリケートします。必要に応じ、優先ブリッジヘッド サーバーや手動接続で相手 DC を DC1/DC2 に限定することも可能です。

根拠（Microsoft 公式ドキュメント）:
- Active Directory のレプリケーションの基本概念（サイト リンク、スケジュール、サイト間レプリケーションの概要）
  https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/replication/active-directory-replication-concepts
- サイト リンクの概要（サイト セットを表し、指定トランスポート上で均一コストで通信できること、KCC による接続作成、ブリッジヘッド サーバーの説明 など）
  https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc770712(v=ws.10)

## 問題7
contoso.com ドメインに Admin1 というユーザーを作成し、SRV1 上でファイルのバックアップと復元を実行できるようにする必要があります。
最小権限の原則に従って構成してください。必要なコンピューターにサインインして作業を完了します。

## 解答7
- 1) ドメイン コントローラーで Active Directory Users and Computers を使用して、ドメイン ユーザー「Admin1」を contoso.com に作成する。
- 2) SRV1 にサインインし、「コンピューターの管理 > ローカル ユーザーとグループ > グループ」で「Backup Operators」に Admin1 を追加する。
  - これにより Admin1 はバックアップ/復元のための必要最小限の権限を得る（管理者権限は不要）。
- （代替/組織的適用が必要な場合）グループ ポリシーまたはローカル セキュリティ ポリシーで SRV1 に対して「ユーザーの権利の割り当て」の「ファイルとディレクトリのバックアップ」「ファイルとディレクトリの復元」を Admin1（または対象グループ）に付与する。

## 解説7
- Backup Operators グループのメンバーは、ファイルのアクセス許可に関わらずバックアップと復元を実行できます。これは管理者権限を与えずにバックアップ/復元のみを許可する「最小権限」の実装に適します。
  - 参考: Active Directory/Windows セキュリティ グループ「Backup Operators」
    - https://learn.microsoft.com/windows/security/identity-protection/access-control/active-directory-security-groups#backup-operators
- バックアップ/復元に必要な権利は「ファイルとディレクトリのバックアップ (SeBackupPrivilege)」「ファイルとディレクトリの復元 (SeRestorePrivilege)」であり、ローカル セキュリティ ポリシーや GPO の「ユーザーの権利の割り当て」で付与できます。
  - Back up files and directories: https://learn.microsoft.com/windows/security/threat-protection/security-policy-settings/back-up-files-and-directories
  - Restore files and directories: https://learn.microsoft.com/windows/security/threat-protection/security-policy-settings/restore-files-and-directories

## 問題8
Azure サブスクリプションに、次の構成を持つ仮想マシン VM1 があります。
- 名前: VM1
- サイズ: Standard DS1 v2
- OS: Windows Server 2022 Datacenter: Azure Edition

VM1 には、次の構成の OS ディスク Disk0 があります。
- サイズ: 127 GB
- 種類: Standard SSD
- 冗長性: ローカル冗長ストレージ (LRS)

あなたは次の操作を行いました。
- Azure ポータルから、VM1 に新しいデータ ディスク Disk1 を追加した。
- VM1 上でアプリ App1 をインストールした。

要件:
- Disk1 を VM1 の D ドライブとして使用できるようにすること。
- App1 が D ドライブに書き込むデータが、VM1 再起動後も保持されること。

最初に何を実行するべきですか？
A. Pagefile.sys を C ドライブに移動する。
B. ディスクの管理で、Disk0 上のボリュームに割り当てられているドライブ文字を変更する。
C. ディスクの管理で、Disk1 上のボリュームに割り当てられているドライブ文字を変更する。
D. VM1 を Standard DS2 v2 にサイズ変更する。


## 解答8
A


## 解説8
- Windows の Azure VM では既定で D: ドライブは「一時ディスク」であり、ページ ファイルの保存先としても使用されます。一時ディスク上のデータはメンテナンスや再展開などで失われる可能性があるため、永続化用途には使えません。
- Disk1 を永続ディスクとして D: に割り当てたい場合、まず D: の占有要因であるページ ファイルを別ドライブ（例: C:）へ移動して D: を解放する必要があります（これが最初の手順）。その後、一時ディスクのドライブ文字を変更し、Disk1 に D: を割り当てます。
- 公式ドキュメントの根拠:
  - Windows VM の一時ディスク（既定で D:）はページ ファイルに使用され、データは保持されない: https://learn.microsoft.com/azure/virtual-machines/windows/faq#what-is-the-temporary-disk-d-drive-on-my-vm
  - 追加したデータ ディスクの初期化とドライブ文字の割り当て手順（管理ディスクのアタッチ）: https://learn.microsoft.com/azure/virtual-machines/windows/attach-managed-disk-portal

# 問題9
あなたのネットワークには、contoso.com という Active Directory Domain Services (AD DS) フォレストがあります。
次のリソースを作成しました。

|Name|Type|Member of|In organizational unit(OU)|
|---|---|---|---|
|User1|User|None|Contoso.com\OU1|
|User2|User|Group1|Contoso.com\OU1|
|User3|User|Group1|Contoso.com\OU2|
|Group1|Group|None|Contoso.com\OU1|
|Comp1|Client computer|Group1|Contoso.com\OU2|

次の GPO を作成しました。

|Name|Linked to|
|---|---|
|GPO1|OU1|
|GPO2|OU2|

次の Group Policy Preferences (GPP) を構成しました。

|GPO|Setting|
|---|---|
|GPO1|Computer Configuration: デスクトップに Link1 というショートカットを追加。 <br>User Configuration: デスクトップに Link2 というショートカットを追加。|
|GPO2|Computer Configuration: デスクトップに Link3 というショートカットを追加。<br> User Configuration: デスクトップに Link4 というショートカットを追加。|

次の各記述について、正しければ Yes、誤っていれば No を選択しなさい。（各設問 1 点）
- User1 が Comp1 にサインインすると、Link4 というショートカットがデスクトップに表示される。
- User2 が Comp1 にサインインすると、Link2 というショートカットがデスクトップに表示される。
- User3 が Comp1 にサインインすると、Link2 というショートカットがデスクトップに表示される。

# 解答9
- User1 が Comp1 にサインインすると Link4 が表示: No
- User2 が Comp1 にサインインすると Link2 が表示: Yes
- User3 が Comp1 にサインインすると Link2 が表示: No

# 解説9
- ユーザー構成の GPO 設定は、既定では「ユーザー アカウントが属する OU にリンクされた GPO」に基づいて適用されます。一方、コンピューター構成の GPO 設定は「コンピューター アカウントが属する OU にリンクされた GPO」に基づいて適用されます（L-S-D-OU の処理順）。
- 本設問ではループバック処理は指定されていないため、ユーザー構成はコンピューターの OU ではなく、ユーザーの OU から適用されます。したがって、
  - User1（OU1）は GPO1 のユーザー設定を受け取り Link2 が対象、GPO2 のユーザー設定（Link4）は受け取りません →「User1 に Link4」は No。
  - User2（OU1）は GPO1 のユーザー設定を受け取り Link2 が表示 → Yes。
  - User3（OU2）は GPO2 のユーザー設定を受け取り Link4 が表示、GPO1 のユーザー設定（Link2）は受け取りません →「User3 に Link2」は No。
- Comp1（OU2）はコンピューター構成として GPO2 の Link3 を受け取りますが、設問は Link2/Link4 の有無のみを問うています。Group1 への所属は、セキュリティ フィルタリングや項目レベル ターゲットを構成しない限り、適用可否に影響しません。

根拠（Microsoft 公式ドキュメント）
- GPO の継承と適用（ユーザー設定はユーザーの SOM、コンピューター設定はコンピューターの SOM に基づく）
  https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-gpo-inheritance
- ループバック処理の動作（指定しない限りユーザー設定はユーザー側の OU から適用）
  https://learn.microsoft.com/en-us/troubleshoot/windows-client/group-policy/loopback-processing-of-group-policy
- Group Policy Preferences の概要（ユーザー構成/コンピューター構成の各項目について）
  https://learn.microsoft.com/en-us/windows/client-management/group-policy-preferences

## 問題10
あなたのネットワークには Active Directory Domain Services (AD DS) ドメインがあり、次のドメイン コントローラーが含まれています。

|Name|Description|
|---|---|
|DC1|Has the schema master, infrastructure master, and domain naming master roles|
|DC2|Has the PDC emulator and RID master roles and is a global catalog server|
|DC3|None|

DC3 をドメインの権限ある時刻サーバー（authoritative time server）として構成する必要があります。どの操作マスター ロールを DC3 に移行し、どのコンソールを使用すべきですか？

選択肢（Answer Area）
- Role: 「Domain naming master」 / 「Infrastructure master」 / 「PDC emulator」 / 「RID master」 / 「Schema master」
- Console: 「Active Directory Administrative Center」 / 「Active Directory Domains and Trusts」 / 「Active Directory Sites and Services」 / 「Active Directory Users and Computers」

## 解答10
- Role: PDC emulator
- Console: Active Directory Users and Computers

## 解説10
- AD ドメインの時刻同期では、「フォレスト ルート ドメインの PDC エミュレーターが組織の権限ある時刻ソース」と定義されています。したがって DC3 を権限ある時刻サーバーにするには、PDC エミュレーター ロールを DC3 に移行します。
  - 参考: Windows Time Service の動作（"the PDC emulator operations master in the forest root domain is the authoritative time server"）
    - https://learn.microsoft.com/windows-server/networking/windows-time-service/how-the-windows-time-service-works
- FSMO ロールの移行は、PDC エミュレーター／RID／インフラストラクチャ マスターについては「Active Directory Users and Computers」から実施できます。
  - 参考: FSMO ロールの表示と移行
    - https://learn.microsoft.com/troubleshoot/windows-server/identity/fsmo-roles

## 問題11
オンプレミスの Windows Server を実行する Server1 があり、Azure サブスクリプションには仮想ネットワーク VNet1 が存在します。
Azure Network Adapter を使用して Server1 を VNet1 に接続する必要があります。
何を使用すべきですか？

A. the Azure portal  
B. Azure AD Connect  
C. Device Manager  
D. Windows Admin Center

## 解答11
D. Windows Admin Center

## 解説11
- Azure Network Adapter は Windows Admin Center の機能で、オンプレミスの Windows Server から Azure 仮想ネットワークへのポイント対サイト VPN 接続をウィザードで簡単に構成できます。公式ドキュメントでは「Azure Network Adapter (ANA) in Windows Admin Center provides a simple, point-and-click experience to configure a point-to-site VPN connection from an on-premises Windows Server to an Azure virtual network.」と説明されています。したがって使用すべきコンソールは Windows Admin Center です。
  - 参考: Use Azure Network Adapter in Windows Admin Center  
    https://learn.microsoft.com/windows-server/manage/windows-admin-center/azure/azure-network-adapter

## 問題12
あなたのネットワークには Active Directory Domain Services (AD DS) フォレストがあり、フォレストには 3 つのドメインが含まれ、それぞれのドメインには 10 台のドメイン コントローラーがあります。
DNS ゾーンをカスタムの Active Directory パーティションに保存する計画です。
ゾーン用の Active Directory パーティションを作成し、そのパーティションが 4 台のドメイン コントローラーのみにレプリケートされるようにする必要があります。使用すべきツールはどれですか？

A. Windows Admin Center  
B. DNS Manager  
C. Active Directory Sites and Services  
D. ntdsutil.exe

## 解答12
D. ntdsutil.exe

## 解説12
- カスタムのアプリケーション ディレクトリ パーティションを作成し、特定のドメイン コントローラーのみにレプリカを追加するには ntdsutil の「partition management」コンテキストを使用します（create nc でパーティション作成、add nc replica で対象 DC を限定）。
  - 参考: ntdsutil（パーティション管理の概要とコマンド）
    - https://learn.microsoft.com/windows-server/identity/ad-ds/manage/ntdsutil/ntdsutil
- DNS の AD 統合ゾーンは、アプリケーション ディレクトリ パーティションに保存でき、そのパーティションに参加している DC 間でのみレプリケートされます。よって、4 台の DC のみにレプリカを構成すれば要件を満たせます。GUI の DNS Manager ではレプリケーション スコープが「ドメイン/フォレストの全 DNS サーバー」等に限定されるため、4 台に限定したカスタム パーティションの新規作成には不向きです。
  - 参考: Active Directory 統合 DNS ゾーンとアプリケーション ディレクトリ パーティションの概要
    - https://learn.microsoft.com/windows-server/networking/dns/advanced/active-directory-integrated-zones

## 問題13
あなたのネットワークには、contoso.com という Active Directory Domain Services (AD DS) ドメインがあります。
ドメイン内で PDC エミュレーターがどのサーバーであるかを特定する必要があります。
提案された解決策: コマンド プロンプトから netdom.exe query fsmo を実行する。
この解決策は目標を満たしていますか？

A. Yes  
B. No

## 解答13
A. Yes

## 解説13
- netdom query fsmo は、ドメイン／フォレスト内のすべての FSMO（操作マスター）ロール保持者を一覧表示します。これには PDC エミュレーター、RID マスター、インフラストラクチャ マスター、ドメイン命名マスター、スキーマ マスターが含まれ、PDC エミュレーターの保持者を特定できます。したがって本解決策は要件を満たします。
  - 参考: netdom query（Microsoft Learn）
    - https://learn.microsoft.com/windows-server/administration/windows-commands/netdom-query
- FSMO ロールの概要と確認/移行手段についての参考情報：
  - https://learn.microsoft.com/troubleshoot/windows-server/identity/fsmo-roles

## 問題14
Windows Server 2022 を実行し、コンテナー ホストとして構成されたサーバー Host1 があります。
Host1 には、Windows Server 2019 をベースにしたコンテナー イメージ image1 が保存されています。
Host1 上で image1 からコンテナーを起動する必要があります。
次のコマンドをどのように完成させるべきですか？回答エリアの適切な選択肢を選択してください。
注意: 各正解は 1 ポイントの価値があります。

Answer Area
```
<「Docker」 or 「MsiExec」 or 「Start」 or 「Start-VM」> run -d <「Container」 or 「-isolation=hyperv」 or 「-isolation=process」> image1
```

## 解答14
Docker run -d -isolation=hyperv image1

## 解説14
- Windows のプロセス分離コンテナーは、ホスト OS とコンテナー ベース イメージのメジャー/マイナー バージョンが一致している必要があります。Host1 は Windows Server 2022、image1 は Windows Server 2019 ベースでバージョンが一致しないため、Hyper-V 分離を使用して起動する必要があります（-isolation=hyperv）。
  - 参考: Windows コンテナーのバージョン互換性
    - https://learn.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility
- Hyper-V 分離は各コンテナーを軽量 VM として実行し、カーネル互換性の差異を分離します。Windows 上での実行時は docker run の --isolation=hyperv（選択肢に合わせて -isolation=hyperv）を指定します。
  - 参考: Hyper-V 隔離コンテナーの管理
    - https://learn.microsoft.com/virtualization/windowscontainers/manage-containers/hyperv-container

## 問題15
あなたの会社では、Kerberos 認証ベースのアプリケーションを展開する必要があります。
オンプレミスのディレクトリ サービスに関する要件は一切ありません。
この場合に最適な展開モデルを選択してください。

A. オンプレミス基盤と Azure VM の両方に AD DS を展開する  
B. オンプレミス AD フォレストのドメインから信頼される別フォレストを展開する  
C. Azure VM 上にのみ AD DS を展開する  
D. どの展開モデルでも選択可能。すべて同等のパフォーマンスとなる

## 解答15
C. Azure VM 上にのみ AD DS を展開する

## 解説15
- Kerberos は Active Directory Domain Services (AD DS) によって提供される認証方式です。アプリが Kerberos/NTLM などの従来の Windows 統合認証に依存する場合、AD DS が必要になります。オンプレ要件がないため、Azure 上の仮想マシンに AD DS をデプロイするのが最小・シンプルな構成です。
  - Kerberos 認証の概要（AD DS と Kerberos）: https://learn.microsoft.com/windows-server/security/kerberos/kerberos-authentication-overview
- Azure で AD DS を必要とするアプリ（Kerberos/LDAP など）を支えるために、Windows Server AD DS を Azure 仮想マシン上にデプロイする設計/リファレンスが提供されています。オンプレを前提としないシナリオにも適用できます。
  - 参考アーキテクチャ（Azure の IaaS 上での AD DS）: https://learn.microsoft.com/azure/architecture/reference-architectures/identity/adds-forest  
  - 追加: AD DS を Azure VM に展開するガイダンス: https://learn.microsoft.com/azure/virtual-machines/workloads/windows/active-directory-ds-deploy

## 問題16
あなたのネットワークには、contoso.com という Azure Active Directory Domain Services (Azure AD DS) ドメインがあります。
contoso.com に参加している Azure 仮想マシン上のローカル ユーザー アカウント向けにパスワード ポリシーを構成する必要があります。
どのように行うべきですか？（各正解は 1 ポイント）

Answer Area
- Sign in by using a user account that is a member of the: 「AAD DC Administrators group」 or 「Administrators group」 or 「Domain Admins group」
- Use a Group Policy Object (GPO) linked to the: 「AADDC Computers organizational unit (OU)」 or 「AADDC Users organizational unit (OU)」 or 「Computers container」

## 解答16
- Sign in: AAD DC Administrators group
- GPO link: AADDC Computers organizational unit (OU)

## 解説16
- Azure AD DS で GPO を管理・編集できるのは「AAD DC Administrators」グループのメンバーです。Azure AD DS には従来の Domain Admins/Enterprise Admins 権限は提供されず、管理は AAD DC Administrators に委ねられます。
  - 参考: Manage Group Policy in Azure AD DS（GPO 管理と AADDC OUs、必要な権限）
    - https://learn.microsoft.com/azure/active-directory-domain-services/manage-group-policy
- Azure AD DS のマネージド ドメインには既定で「AADDC Computers」と「AADDC Users」の 2 つの OU があり、コンピューターに適用するポリシーは AADDC Computers にリンクします。パスワード ポリシーを「コンピューター構成 > Windows の設定 > セキュリティの設定 > アカウント ポリシー」で定義すると、ドメイン参加コンピューター上のローカル アカウントに対して適用されます（ドメイン アカウントのポリシーはドメイン レベルのアカウント ポリシーが適用）。
  - 参考: Account policies（アカウント/パスワード ポリシーの適用範囲: ドメイン vs ローカル）
    - https://learn.microsoft.com/windows/security/threat-protection/security-policy-settings/account-policies
    - https://learn.microsoft.com/windows/security/threat-protection/security-policy-settings/password-policy

## 問題17
SIMULATION
Windows Admin Center を使用して、SRV1 から DC1 を管理できるようにする必要があります。
必要なソース ファイルは \\dc1.contoso.com\install フォルダーにあります。
このタスクを完了するために、必要なコンピューターにサインインしてください。

## 解答17
- 1) DC1 にサインインし、\\dc1.contoso.com\install にある Windows Admin Center インストーラー（MSI）を実行してインストールを完了する（ゲートウェイ モードでインストールし、自己署名証明書の作成を許可）。
- 2) SRV1 にサインインし、ブラウザーでインストーラー最終画面に表示された URL（例: https://dc1.contoso.com:6516）へアクセスして Windows Admin Center に接続する。必要に応じて証明書を信頼し、資格情報でサインイン後、DC1 を接続して管理する。

## 解説17
- Windows Admin Center はローカルにデプロイしてブラウザーでアクセスする管理ゲートウェイであり、Windows Server（ドメイン コントローラーを含む）をリモート管理できます。ゲートウェイをサーバー上にインストールし、他のマシン（SRV1 など）から表示された URL へアクセスして管理します。本設問では DC1 にインストールし、SRV1 のブラウザーから接続することで「SRV1 から DC1 を管理」する要件を満たします。
  - 参考: Install Windows Admin Center（インストールとゲートウェイ アクセス）
    - https://learn.microsoft.com/windows-server/manage/windows-admin-center/deploy/install
  - 参考: Windows Admin Center overview（ブラウザー ベースでサーバーやクラスターを管理）
    - https://learn.microsoft.com/windows-server/manage/windows-admin-center/overview

## 問題18
あなたのネットワークには Active Directory Domain Services (AD DS) ドメインがあり、ドメインには Windows Server を実行するサーバーと Windows を実行するコンピューターが含まれます。
ドメインにリンクしたグループ ポリシー オブジェクト (GPO) で「BranchCache を有効にする (Turn on BranchCache)」を [有効] に設定しました。
BranchCache を「分散キャッシュ モード (Distributed Cache mode)」で有効にする必要があります。
各コンピューターで何を構成する必要がありますか？

A. BITS (Background Intelligent Transfer Service)
B. オフライン ファイル (Offline Files)
C. 共有の詳細設定 (Advanced sharing settings)
D. Windows ファイアウォールの受信トラフィック ルール (Windows Firewall inbound traffic rules)

## 解答18
D. Windows ファイアウォールの受信トラフィック ルール (Windows Firewall inbound traffic rules)

## 解説18
- BranchCache を分散キャッシュ モードで動作させる場合、クライアント同士がピア探索とコンテンツ取得を行えるように、各クライアント コンピューターで Windows Defender ファイアウォールの受信規則を有効にする必要があります（例: BranchCache - Peer Discovery (WSD-In/UDP 3702)、BranchCache - Content Retrieval (HTTP-In/TCP 80, 必要に応じて HTTPS-In/TCP 443) など）。「Turn on BranchCache」を GPO で有効化するだけでは通信が許可されないため、分散モードの要件として受信ファイアウォール規則の構成が必要です。
  - 参考: BranchCache の概要（分散/ホステッド モード、および必要な構成の概観）
    - https://learn.microsoft.com/windows-server/networking/branchcache/branchcache
  - 参考: BranchCache の展開ガイダンス（クライアント構成とファイアウォール要件）
    - https://learn.microsoft.com/windows-server/networking/branchcache/deploy/deploy-branchcache

## 問題19
あなたのネットワークには Active Directory Domain Services (AD DS) ドメインがあり、ドメインには Windows Server を実行するサーバー、および Windows を実行するコンピューターが含まれます。
次の表のとおり、DNS Server ロールがインストールされたサーバーがあります。

|Name|Office|Local DNS zone|IP address|
|---|---|---|---|
|Server1|Paris|contoso.com|10.1.1.1|
|Server2|New York|None|10.2.2.2|

ニューヨーク オフィス内のすべてのクライアント コンピューターは、Server2 を DNS サーバーとして使用しています。

次の要件を満たすように、ニューヨーク オフィスで名前解決を構成する必要があります。
- ニューヨークのクライアント コンピューターが contoso.com の名前を解決できるようにする。
- Server2 がインターネット ホストに対するすべての DNS クエリを 131.107.100.200 に転送するようにする。

この解決策では、Server1 に対する変更を必要としてはなりません。
各正解は解決策の一部を表します。
注: 各正しい選択は 1 ポイントの価値があります。

A. a forwarder  
B. a conditional forwarder  
C. a delegation  
D. a secondary zone  
E. a reverse lookup zone

## 解答18
A. a forwarder  
B. a conditional forwarder

## 解説18
- 条件付きフォワーダーは、特定のドメイン（例: contoso.com）宛てのクエリを、指定した DNS サーバー（本設問では Server1: 10.1.1.1）へ転送するために使用します。これにより、Server2 側の設定だけで contoso.com の解決が可能になり、Server1 への変更は不要です。
  - 参考: Conditional forwarders の概要と構成  
    https://learn.microsoft.com/windows-server/networking/dns/deploy/dns-conditional-forwarding
- フォワーダーは、DNS サーバーが自分で解決できない外部（インターネットなど）の名前解決を、上流の DNS（本設問では 131.107.100.200）に一括転送するために使用します。Server2 に標準フォワーダーを設定することで要件を満たします。
  - 参考: Configure a DNS server to use forwarders  
    https://learn.microsoft.com/windows-server/networking/dns/deploy/forwarders
- セカンダリ ゾーン（D）は Server1 側でゾーン転送の許可などの変更が必要になる可能性が高く要件に反します。委任（C）や逆引きゾーン（E）は本要件の達成に不要です。

## 問題19
あなたのネットワークには Active Directory ドメイン contoso.com があり、次のコンピューターが含まれています。

|Name|Operating system|
|---|---|
|Computer1|Windows 11|
|Server1|Windows Server 2016|
|Server2|Windows Server 2019|
|Server3|Windows Server 2022|

Server3 上で GPO1 というグループ ポリシー オブジェクト (GPO) を作成し、GPO1 を contoso.com にリンクしました。
GPO1 にはショートカットの「基本設定」項目 Shortcut1 が含まれており、アイテム レベルのターゲット設定 (Item-level targeting) は次の図のとおり構成されています。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/75e36e73-bbee-425f-bbc8-7262d054afdf.png)

〈ターゲット設定の内容〉
- the operating system is Windows Server 2022 Family
- Product: Windows Server 2022 Family / Edition: Any / Release: Any / Computer Role: Any

Shortcut1 はどのコンピューターに適用されますか？

A. Server3 のみ  
B. Computer1 と Server3 のみ  
C. Server2 と Server3 のみ  
D. Server1、Server2、Server3 のみ

## 解答19
A. Server3 のみ

## 解説19
- グループ ポリシーの「基本設定 (Preferences)」は、アイテム レベルのターゲット設定 (Item-level targeting) により、対象の OS、グループ、レジストリ条件などに合致する場合にのみ適用されます。今回は「Operating System = Windows Server 2022 Family」と指定しているため、Windows Server 2022 を実行する Server3 のみに適用され、Windows 11（Computer1）、Windows Server 2016（Server1）、Windows Server 2019（Server2）には適用されません。
  - 参考: Group Policy Preferences のアイテム レベル ターゲット設定（Operating System ターゲット項目）
    - https://learn.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc733022(v=ws.11)
    - https://learn.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc753992(v=ws.11)
- また、GPO をドメインにリンクしても、「基本設定」項目自体の適用はアイテム レベルのターゲット条件により制御されます。したがってリンク範囲に含まれていても、条件を満たさない OS には適用されません。

## 問題20
次の表のとおり、Windows Server を実行するオンプレミスのファイル サーバーがあります。
|Name|Relevant folder|
|---|---|
|Server1|D:\Folder1, E:\Folder2|
|Server2|D:\Data|

次の表のとおり、Azure ファイル共有があります。
|Name|Location|
|---|---|
|share1|East US|
|share2|East US|

Storage Sync Service の Sync1 と、Azure File Sync の同期グループ Group1 を追加しました。
Group1 は share1 をクラウド エンドポイントとして使用します。
Server1 と Server2 を Sync1 に登録し、Server1 の D:\Folder1 を Group1 のサーバー エンドポイントとして追加しました。

次の各記述について、正しい場合は Yes、正しくない場合は No を選択してください。
注: 正解 1 つにつき 1 点です。

Answer Area
Statements
- You can add share2 as a cloud endpoint in Group1. 「Yes」or「No」
- You can add E:\Folder2 from Server1 as a server endpoint in Group1.「Yes」or「No」
- You can add D:\Data from Server2 as a server endpoint in Group1.「Yes」or「No」

## 解答20
|記述|回答|
|---|---|
|You can add share2 as a cloud endpoint in Group1.|No|
|You can add E:\Folder2 from Server1 as a server endpoint in Group1.|No|
|You can add D:\Data from Server2 as a server endpoint in Group1.|Yes|

## 解説20
- 同期グループはクラウド エンドポイントを 1 つしか持てません。Group1 は既に share1 をクラウド エンドポイントとして使用しているため、share2 を同じ同期グループに追加することはできません。
  - 公式ドキュメント: 「同期グループには 1 つのクラウド エンドポイント（Azure ファイル共有）と 1 つ以上のサーバー エンドポイントが含まれます。」
    - https://learn.microsoft.com/azure/storage/files/storage-sync-files-planning#sync-groups
- 同一サーバーは同じ同期グループ内で複数のサーバー エンドポイントを持つことはできません。Server1 は既に D:\Folder1 を Group1 に追加済みのため、同じ Group1 に Server1 の E:\Folder2 を追加することはできません。
  - 公式ドキュメント（制限事項/設計指針）: 同期グループあたりクラウド エンドポイントは 1、サーバー エンドポイントは複数可だが、同一サーバー内で同一同期グループに複数のサーバー エンドポイントを構成することはサポートされません（トポロジ制限）。
    - 参考: スケール目標と制限事項（Cloud endpoints per sync group: 1、Server endpoints per sync group: 複数可）
    - https://learn.microsoft.com/azure/storage/files/storage-sync-files-scale-targets
- 別サーバー（Server2）のパスを同じ同期グループにサーバー エンドポイントとして追加することは可能です。同期グループは 1 つのクラウド エンドポイントと複数のサーバー エンドポイント（複数サーバーにまたがる）で構成できます。
  - 公式ドキュメント: 同期グループの概要（1 つのクラウド エンドポイント＋複数サーバー エンドポイント）
    - https://learn.microsoft.com/azure/storage/files/storage-sync-files-planning#sync-groups

## 問題21
あなたは、次のストレージ アカウントを含む Azure サブスクリプションを持っています。

|Name|Location|Kind|Container|File share|
|---|---|---|---|---|
|storage1|East US|StorageV2|cont1|share1|
|storage2|East US|BlockBlobStorage|cont2|Not applicable|
|storage3|East US|FileStorage|Not applicable|share3|
|storage4|Central US|StorageV2|cont4|share4|

East US リージョンに、Sync1 という Storage Sync Service を作成しました。
Sync1 の中に同期グループ (sync group) を作成する必要があります。

どのストレージ アカウントを使用でき、クラウド エンドポイントとして何を指定できますか。該当する選択肢を回答エリアから選択してください。
注意: 各正解は 1 ポイントです。

Answer Area
- Storage accounts: 「storage1 only」/「storage1 and storage3 only」/「storage1 and storage4 only」/「storage1,storage2 and storage3 only」/「storage1,storage2 ,storage3 and storage4 only」
- Cloud endpoints: 「cont1 and cont2 only」/「share1 and share3 only」/「cont1, cont2, and cont4 only」/「share1, share3, and share4 only」/「cont1, cont2, share1, and share3 only」/「cont1, cont2, cont4, share1, share3, and share4」

## 解答21
- Storage accounts: storage1 and storage3 only
- Cloud endpoints: share1 and share3 only

## 解説21
- Azure File Sync のクラウド エンドポイントは「Azure ファイル共有 (Azure file share)」でなければなりません。また、そのファイル共有が存在する「ストレージ アカウントは Storage Sync Service と同一リージョンである必要」があります。本設問では Sync1 は East US にあるため、East US 内のストレージ アカウントが対象です。
  - Cloud endpoint の要件（クラウド エンドポイントは Azure ファイル共有。Storage Sync Service と同一リージョン）:
    - https://learn.microsoft.com/azure/storage/file-sync/file-sync-planning#cloud-endpoint
- Azure Files を利用できるストレージ アカウント種別は、標準の汎用 v2 (StorageV2) と Premium の FileStorage です。BlockBlobStorage はブロック BLOB のみを提供し、ファイル共有をサポートしません。したがって使用可能なストレージ アカウントは East US にある StorageV2 の storage1 と、FileStorage の storage3 のみで、クラウド エンドポイントとして指定できるのはそれぞれのファイル共有 share1 と share3 のみです。
  - ストレージ アカウント種別とサポート対象（Files は StorageV2 と FileStorage が対象／BlockBlobStorage は対象外）:
    - https://learn.microsoft.com/azure/storage/common/storage-account-overview#types-of-storage-accounts
  - Azure Files の概要（Azure file share の定義）:
    - https://learn.microsoft.com/azure/storage/files/storage-files-introduction

## 問題22
Hyper-V サーバー ロールがインストールされた Host1 というサーバーがあります。Host1 には VM1 という仮想マシンがホストされています。
Windows Server を実行する管理サーバー Server1 があり、Server1 から Hyper-V マネージャーを使用して Host1 をリモート管理しています。
Virtual Machine Connection を使って VM1 に接続する際に、Server1 に接続された USB ハード ドライブへアクセスできるようにする必要があります。
次のうち、解決策の一部となる 2 つの操作はどれですか。正しい選択肢を 2 つ選んでください。
注: 正解 1 つにつき 1 点です。

A. Host1 の Hyper-V の設定で「拡張セッション モードを許可する」を選択する。
B. Virtual Machine Connection で「Show Options」を選び、USB ハード ドライブを選択する。
C. Virtual Machine Connection で基本セッションに切り替える。
D. Host1 の「ディスクの管理」で「ディスクの再スキャン」を選択する。
E. Host1 の「ディスクの管理」で仮想ハード ディスクを接続する。

## 解答22
A、B

## 解説22
- USB ハード ドライブなどのローカル リソースを VMConnect 経由でゲストにリダイレクトするには、Hyper-V の拡張セッション モードが必要です。ホスト側で「拡張セッション モードを許可する」を有効化するのが前提となります（A）。
  - 根拠（Microsoft 公式）: Enhanced session mode を有効にすると、ローカル デバイスとリソース（ドライブ、クリップボード、プリンター、オーディオ など）を VM にリダイレクトできます。ホストの Hyper-V 設定で Allow enhanced session mode をオンにします。
    - https://learn.microsoft.com/windows-server/virtualization/hyper-v/learn-more/enhanced-session-mode
- VMConnect からは「Show Options」→「Local Resources」→「More…」でローカル ドライブを選択できます。USB 接続のハード ドライブはクライアント側のドライブとして選択し、ゲストにリダイレクトできます（B）。
  - 根拠（Microsoft 公式）: VMConnect で「Show Options」を使い、ドライブやその他のローカル リソースを選択して接続できます。
    - https://learn.microsoft.com/windows-server/virtualization/hyper-v/learn-more/use-local-resources-on-hyper-v-virtual-machine-with-vmconnect
- なお、基本セッション（C）はローカル リソースのリダイレクトが制限されるため要件に合いません。ディスク再スキャン（D）や VHD の接続（E）はホストのストレージ操作であり、クライアント（Server1）の USB ドライブを VM に転送する目的には該当しません。

## 問題23
SIMULATION
SRV1 を DNS サーバーとして構成する必要があります。
SRV1 は、contoso.com ドメインの名前解決を DC1 を用いて実行できなければなりません。
その他すべての名前は、ルート ヒント サーバーを使用して解決される必要があります。
このタスクを完了するには、必要なコンピューターにサインインして操作してください。

## 解答23
- SRV1 の DNS マネージャーで、正引きのセカンダリ ゾーン「contoso.com」を作成し、マスター DNS サーバーとして DC1（その IP アドレス）を指定する。併せて、DC1 側の「contoso.com」プライマリ ゾーンでゾーン転送を SRV1 に許可する。
- SRV1 の DNS サーバー プロパティでルート ヒントを既定通り有効にしておき、フォワーダーは構成しない（または無効化/削除する）。これにより、contoso.com 以外の名前解決はルート ヒントを使用して行われる。

## 解説23
- セカンダリ ゾーンは、プライマリ ゾーンからゾーン転送で読み取り専用のコピーを保持します。よって SRV1 に「contoso.com」のセカンダリ ゾーンを作成し、マスターとして DC1 を指定し、DC1 側でゾーン転送を許可する必要があります。
  - 公式ドキュメント（DNS ゾーンの種類: セカンダリ ゾーンはプライマリから転送で取得）
    - https://learn.microsoft.com/windows-server/networking/dns/dns-zones/dns-zone-types
- ルート ヒントは、サーバーがローカルに権威を持たない名前を解決する際に使用され、既定で設定されています。その他の名前をルート ヒントで解決させるにはフォワーダーを設定しないようにします（フォワーダーを設定すると通常はそれが優先されます）。
  - 公式ドキュメント（DNS Manager で Root Hints タブからルート ヒントを管理でき、既定で構成済み）
    - https://learn.microsoft.com/windows-server/networking/dns/dns-tools/dns-manager
  - 公式ドキュメント（フォワーダーの動作と優先順位の概念）
    - https://learn.microsoft.com/windows-server/networking/dns/dns-forwarding

## 問題24
オンプレミスのサーバー Server1 から Azure に Azure File Sync を用いてファイルを同期する必要があります。
クラウド階層化（Cloud Tiering）のポリシーは「空き容量 30%」「クーリング日数 70 日」に設定されています。
Server1 のボリューム E は 500 GB です。

1 年前、Server1 の E:\Data を Azure File Sync で同期するように構成しました。現在、E:\Data に“見えている”ファイルは次のとおりです。

|Name|Size|Last accessed|
|---|---|---|
|File1|200GB|2 days ago|
|File2|100GB|10 days ago|
|File3|200GB|60 days ago|
|File4|50GB|100 days ago|

ボリューム E には他のファイルは含まれていません。

File1 と File3 はどこに配置されていますか。
解答するには、解答エリアで適切な選択肢を選んでください。

解答エリア
File1:「Server1 only」または「The Azure file share only」または「Server1 and the Azure file share」
File3:「Server1 only」または「The Azure file share only」または「Server1 and the Azure file share」

## 解答24
- File1: Server1 and the Azure file share
- File3: The Azure file share only

## 解説24
- クラウド階層化は 2 つのポリシーで動作します。（1）日付ポリシー（クーリング日数）：指定日数より長くアクセスされていない“コールド”なファイルを階層化対象とする。（2）ボリューム空き容量ポリシー：ボリュームの空き容量の目標（本問では 30%＝150 GB）を満たすため、最終アクセスが古いファイルから順にローカルの実体を削除していきます。必要な空き容量を確保するために、クーリング日数未満のファイルも階層化され得ます。
- 本問の総論理サイズは 550 GB。空き容量 30% を満たすにはローカル実体は最大 350 GB まで。最近アクセスされた File1（200GB）と File2（100GB）を優先的にローカルに保持して 300 GB。残余 50 GB では File3（200GB）をローカルに保持できないため、File3 は階層化（＝Server1 ではスタブ、実体は Azure のみ）。結果として、File1 は「Server1 と Azure の両方」、File3 は「Azure のみ」となります。
- 参考（Microsoft 公式ドキュメント）
  - Cloud tiering の仕組み（ボリューム空き容量ポリシーと日付ポリシー、空き容量確保のためにアクセスが古いファイルから階層化）：https://learn.microsoft.com/azure/storage/file-sync/file-sync-cloud-tiering
  - Azure File Sync の概要（Azure 側に完全データ、サーバーはキャッシュとして機能）：https://learn.microsoft.com/azure/storage/file-sync/file-sync-overview

## 問題25
あなたのネットワークには、adatum.com という名前の Active Directory Domain Services (AD DS) ドメインがあります。
ドメインには Server1 というサーバーと、次の表のユーザーが含まれます。
|Name|Member of|
|---|---|
|User1|Group1|
|User2|Group2|
|User3|Group3|

Server1 には D:\Folder1 というフォルダーがあります。
Folder1 の「詳細なセキュリティ設定 (NTFS 権限)」は次のとおりです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/465996f2-08a4-4cb0-a20a-4cd3a2c07d31.png)

|Type|Principal|Access|Applies to|
|---|---|---|---|
|Allow|Administrators (SERVER1\Administrators)|Full control|This folder, subfolders and files|
|Allow|Group1 (ADATUM\Group1)|Read|This folder, subfolders and files|
|Allow|Group2 (ADATUM\Group2)|Write|This folder, subfolders and files|

Folder1 は次の設定で共有されています:
- Path: D:\Folder1
- Name: Share1
- ShareType: FileSystemDirectory
- FolderEnumerationMode: Unrestricted（アクセス ベースの列挙を無効）

Share1 の共有アクセス許可は次のとおりです。
|Group|Permission|
|---|---|
|Group1|Allow - Change|
|Group3|Allow - Full Control|

次の各記述について、正しければ Yes、誤りであれば No を選択してください。
注: 正解それぞれ 1 点。
|Statements|Yes|No|
|---|---|---|
|User1 can read the files in Share1.| | |
|User3 can delete the files in Share1.| | |
|If User2 connects to \\Server1.adatum.com from File Explorer, Share1 will be listed.| | |

## 解答25
|Statements|Yes|No|
|---|---|---|
|User1 can read the files in Share1.|Yes| |
|User3 can delete the files in Share1.| |No|
|If User2 connects to \\Server1.adatum.com from File Explorer, Share1 will be listed.|Yes| |

## 解説25
- User1 は Group1 のメンバーです。共有は Group1 に「Change（変更）」を許可しており（変更は読み取りを含む）、NTFS は Group1 に「Read」を許可しています。ネットワーク経由のアクセスでは「共有権限」と「NTFS 権限」のうち“より制限の厳しい方”が有効権限になりますが、この組み合わせなら読み取りは許可されます。
  - 根拠（Microsoft Docs）: 共有と NTFS 権限は組み合わせて評価され、ネットワーク アクセス時は最も制限の厳しい方が適用される。
    - https://learn.microsoft.com/windows-server/storage/file-server/permissions-overview
    - https://learn.microsoft.com/powershell/module/smbshare/grant-smbshareaccess

- User3 は Group3 のメンバーで、共有は Group3 に「Full Control（フル コントロール）」を許可していますが、NTFS 側に Group3 への許可がありません。ネットワーク越しの実効権限は共有と NTFS の“より制限の厳しい方”で決まるため、NTFS が許可していない削除はできません（ファイル削除には、対象ファイルの Delete または親フォルダーの「Delete subfolders and files」が必要）。
  - 根拠（Microsoft Docs）:
    - 共有/NTFS の最も制限の厳しい方が有効: https://learn.microsoft.com/windows-server/storage/file-server/permissions-overview
    - NTFS の削除関連権限（Delete／Delete subfolders and files）: https://learn.microsoft.com/windows/security/identity-protection/access-control/ntfs-permissions#advanced-permissions

- User2 は Group2 のメンバーで、共有権限は割り当てられていませんが、質問は「\\Server1.adatum.com にエクスプローラーで接続したときに Share1 が“一覧表示されるか”」です。設定では FolderEnumerationMode が Unrestricted（アクセス ベースの列挙を無効）であり、アクセス ベースの列挙は“共有内のファイル/フォルダーの表示可否”を制御する機能で、共有そのものの一覧表示を非表示にするものではありません。したがって Share1 は一覧に表示されます（ただし実際に開こうとすると権限がなければアクセス拒否）。
  - 根拠（Microsoft Docs）:
    - Access-based Enumeration は共有内のファイル/フォルダーのみを非表示にする機能: https://learn.microsoft.com/windows-server/storage/file-server/access-based-enumeration
    - New-SmbShare の FolderEnumerationMode（Unrestricted/AccessBased）: https://learn.microsoft.com/powershell/module/smbshare/new-smbshare#-folderenumerationmode

## 問題26
Windows Server を実行する Server1 に、Share1 というファイル共有があります。
Share1 に対して、ユーザーが MP4 ファイルを保存できないようにしつつ、その他の種類のファイルは保存できるようにする必要があります。
Server1 上で何を構成すべきですか。
A. file screens
B. NTFS permissions
C. File Management Tasks
D. NTFS Quotas

## 解答26
A. file screens

## 解説26
- MP4 のような特定の拡張子（ファイルの種類）を保存禁止にするには、File Server Resource Manager (FSRM) の「ファイル スクリーン」を使用します。ファイル スクリーンは、指定したフォルダー/共有に対して定義済みまたはカスタムのファイル グループ（例: オーディオとビデオ ファイル＝.mp4 など）をブロックできます。これにより MP4 は拒否し、その他のファイルは許可するポリシーを実現できます。
  - 公式ドキュメント（FSRM 概要: ファイル スクリーニング管理）: https://learn.microsoft.com/windows-server/storage/fsrm/fsrm-overview#file-screening-management
  - 公式ドキュメント（ファイル スクリーンの作成）: https://learn.microsoft.com/windows-server/storage/fsrm/create-file-screen
- 参考: NTFS 権限（B）は拡張子単位の制御はできず、NTFS クォータ（D）は容量制限であり種類制御ではありません。File Management Tasks（C）は分類・移動・有効期限などの自動処理で、保存そのもののブロック用途ではありません。

## 問題27
あなたのネットワークには、contoso.com という Active Directory Domain Services (AD DS) ドメインがあります。
ドメインの PDC エミュレーターがどのサーバーかを特定する必要があります。
提案された解決策: Active Directory サイトとサービス で、コンソール ツリーの Default-First-Site-Name を右クリックし、[プロパティ] を選択する。
この解決策は目標を満たしていますか。
A. No
B. Yes

## 解答27
A. No

## 解説27
- PDC エミュレーターなどのドメイン内 FSMO 役割の確認は、一般に「Active Directory ユーザーとコンピューター」でドメインを右クリック→[操作マスター]→[PDC] タブ、または netdom query fsmo などで行います。Active Directory サイトとサービス のサイト プロパティからは PDC エミュレーターを特定できません。
  - 公式ドキュメント: FSMO 役割の表示/移行（ADUC の操作マスター ダイアログや netdom の利用）
    - https://learn.microsoft.com/troubleshoot/windows-server/identity/transfer-or-seize-fsmo-roles-in-ad-ds
  - 公式ドキュメント: netdom query fsmo（FSMO 役割保有者の一覧表示）
    - https://learn.microsoft.com/windows-server/administration/windows-commands/netdom-query

## 問題28
Windows Server を実行する Azure 仮想マシン VM1 があり、次の構成になっています。
* サイズ: D2s_v4
* オペレーティング システム ディスク: 127 GiB の Standard SSD
* データ ディスク: 128 GiB の Standard SSD
* 仮想マシンの世代: 第 2 世代 (Gen 2)

次の変更を VM1 に対して実行する予定です。
* 仮想マシンのサイズを D4s_v4 に変更する。
* データ ディスクをデタッチする。
* 新しい Standard SSD を追加する。

どの変更に VM1 のダウンタイムが必要ですか。
A. データ ディスクのデタッチのみ と 新しい Standard SSD の追加
B. データ ディスクのデタッチのみ
C. 仮想マシンのサイズ変更のみ
D. 新しい Standard SSD の追加のみ

## 解答28
C. 仮想マシンのサイズ変更のみ

## 解説28
- サイズ変更（D2s_v4 → D4s_v4）は、同一ハードウェア クラスター上で可能な場合でも VM の再起動が発生し、場合によってはサイズが同一クラスターで利用不可なときにいったん割り当て解除（停止/割り当て解除）が必要です。いずれにせよ再起動はダウンタイムです。
  - 公式ドキュメント: 「一部のサイズ変更では VM の割り当て解除が必要です。サイズ変更では VM が再起動されます。」
    - https://learn.microsoft.com/azure/virtual-machines/windows/resize-vm
- データ ディスクのデタッチおよび新しいデータ ディスク（Standard SSD）のアタッチは、VM の稼働中に実施できます（ゲスト OS 側でのオフライン/オンラインやディスクの再スキャンは必要）。これらは VM の停止を伴いません。
  - 公式ドキュメント（ディスクのデタッチ）: 「Windows VM からデータ ディスクをデタッチできます。」（停止/割り当て解除は不要）
    - https://learn.microsoft.com/azure/virtual-machines/windows/detach-disk
  - 公式ドキュメント（ディスクのアタッチ）: 「Windows VM にマネージド ディスクをアタッチできます。」（VM 稼働中に可能）
    - https://learn.microsoft.com/azure/virtual-machines/windows/attach-managed-disk-portal

## 問題29
オンプレミスの Active Directory Domain Services (AD DS) ドメインがあり、これは Azure Active Directory (Azure AD) テナントと同期されています。
Windows Server を実行する 100 台の新しい Azure 仮想マシンを展開する予定です。
各新規仮想マシンが AD DS ドメインに参加していることを確実にする必要があります。
何を使用すべきですか？

A. Azure AD Connect  
B. グループ ポリシー オブジェクト (GPO)  
C. Azure Resource Manager (ARM) テンプレート  
D. Azure 管理グループ

## 解答29
C. Azure Resource Manager (ARM) テンプレート

## 解説29
- 大量の Windows 仮想マシンを展開時に自動で AD DS ドメインに参加させるには、ARM テンプレートでドメイン参加拡張機能（JsonADDomainExtension）を指定するのが推奨です。これによりデプロイ時にドメイン参加が実行され、全 VM を確実かつ一貫してドメイン参加できます。
- Azure AD Connect はオンプレミス AD DS と Azure AD のディレクトリ同期を提供するもので、仮想マシンのドメイン参加を自動化する機能ではありません。
- 管理グループは Azure リソースのガバナンス（ポリシーや RBAC の適用単位）であり、ドメイン参加の自動化には用いません。GPO はドメイン参加済みのコンピューターに適用されるため、ドメイン参加前の自動化手段としては適切ではありません。
- 参考（Microsoft 公式ドキュメント）
  - Domain Join extension for Windows（ARM テンプレートや拡張機能でのドメイン参加）: https://learn.microsoft.com/azure/virtual-machines/extensions/domain-join-windows
  - What is Azure AD Connect?（ディレクトリ同期の概要）: https://learn.microsoft.com/azure/active-directory/hybrid/whatis-azure-ad-connect
  - Azure 管理グループの概要（ガバナンス用途）: https://learn.microsoft.com/azure/governance/management-groups/overview

## 問題30
Windows Server のコンテナー ホスト Server1 を所有しています。
|Name|Image|Uses Hyper-V isolation|コンテナーで実行中のプロセス|
|---|---|---|---|
|Container1|microsoft/iis|No|ProcessA|
|Container2|microsoft/iis|No|ProcessB|
|Container3|microsoft/iis|Yes|ProcessC|
|Container4|microsoft/iis|Yes|ProcessD|

ProcessA と ProcessC の状態を検証する必要があります。ProcessA と ProcessC が実行中であることをどこで確認できますか。
解答エリアで適切なオプションを選択してください。
注: 各正解は 1 点です。
ProcessA:「Container 1 only」または「Container 1 and Container2 only」または「Container1 and Server1 only」または「Container1, Container2, and Server1 only」または「All the containers and Server1」
ProcessC:「Container 3 only」または「Container 3 and Container4 only」または「Container3 and Server1 only」または「Container3, Container4, and Server1 only」または「All the containers and Server1」

## 解答30
- ProcessA: Container1 and Server1 only
- ProcessC: Container 3 only

## 解説30
- Windows Server コンテナー（Process/Windows Server isolation）はホスト OS のカーネルを共有して動作します。このため、コンテナー内のプロセスはコンテナー内で確認できるだけでなく、ホスト（Server1）側でもプロセスとして観測できます。一方、
- Hyper-V 分離（Hyper-V isolation）は各コンテナーを専用の軽量 VM（ユーティリティ VM）内で実行し、ホストとカーネルを共有しません。このため、コンテナー内プロセスはホストからは見えず、当該コンテナー（ユーティリティ VM）の中でのみ確認できます。

根拠（Microsoft 公式ドキュメント）
- Windows コンテナーの概要（分離モードの違い: Windows Server コンテナーはホストとカーネルを共有、Hyper-V 分離は専用の VM で実行）: https://learn.microsoft.com/virtualization/windowscontainers/about/
- Hyper-V 分離の概要（各コンテナーが特別な軽量 VM 内で実行される）: https://learn.microsoft.com/virtualization/windowscontainers/manage-containers/hyperv-container

## 問題31
あなたのネットワークには Active Directory Domain Services (AD DS) フォレストがあり、次のサーバーが含まれています。
|Name|In domain|Description|
|---|---|---|
|Server1|contoso.com|Domain controller, DNS server|
|Server2|contoso.com|Domain controller, DNS server|
|Server3|contoso.com|DNS server|
|Server4|east.contoso.com|Domain controller, DNS server|
|Server5|east.contoso.com|DNS server|

Server1 上で、次の設定のとおり DNS ゾーン Zone1.com を作成しました。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/138cb2a9-e259-4f69-9846-0f0ce4c344b7.png)

図のプロパティには以下が示されています：
- Type: Active Directory-Integrated
- Replication: All DNS servers in this domain（このドメイン内のすべての DNS サーバー）

Zone1.com はどの DNS サーバーに複製されますか。
A. Server2 and Server3 only
B. Server2, Server3, and Server4 only
C. Server2 only
D. Server2, Server3, Server4, and Server5
E. Server2 and Server4 only

## 解答31
C. Server2 only

## 解説31
- AD 連携ゾーン（Active Directory-integrated zone）は、ゾーン データを Active Directory に格納し、設定された「複製スコープ」に基づいて複製されます。複製スコープが「このドメイン内のすべての DNS サーバー（All DNS servers in this domain）」の場合、DomainDNSZones アプリケーション パーティションを使って「そのドメイン内の DNS サーバー機能を持つドメイン コントローラー」にのみ複製されます。したがって contoso.com ドメインの DC かつ DNS である Server2 のみが対象となり、メンバー サーバーの DNS（Server3）や別ドメイン（east.contoso.com）のサーバー（Server4, Server5）には複製されません。
  - 公式ドキュメント（AD 連携 DNS ゾーンと複製スコープ: ドメイン内の DNS サーバーに複製／DomainDnsZones・ForestDnsZones の説明）
    - https://learn.microsoft.com/windows-server/networking/dns/deploy/dns-active-directory-integrated-zones#replication-scopes-for-active-directory--integrated-dns-zones
  - 公式ドキュメント（Active Directory-integrated zones の概要: ゾーン データは AD に格納され、DC によって複製）
    - https://learn.microsoft.com/windows-server/networking/dns/what-is-dns#active-directory-integrated-zones

## 問題32
あなたのネットワークには、contoso.com という名前のオンプレミス Active Directory Domain Services (AD DS) ドメインがあります。
ドメインには次のオブジェクトが含まれています。

|Name|Type|
|---|---|
|User1|User|
|Group1|Universal security group|
|Group2|Domain local security group|
|Computer1|Computer|

Azure Active Directory (Azure AD) テナントと Azure AD Connect を使用して contoso.com を同期する予定です。
すべてのオブジェクトを条件付きアクセス (Conditional Access) ポリシーで使用できるようにする必要があります。
何を使用すべきですか？

A. Group2 のスコープを Universal に変更する  
B. 「Configure device writeback」オプションをオフにする  
C. Group1 と Group2 のスコープを Global に変更する  
D. 「Configure Hybrid Azure AD join」オプションを選択する

## 解答32
A. Group2 のスコープを Universal に変更する

## 解説32
- 条件付きアクセス（Microsoft Entra 条件付きアクセス）の割り当てで使用できる「ユーザーとグループ」は、Azure AD（Microsoft Entra ID）に存在（同期）している必要があります。Azure AD Connect はドメイン ローカル（Domain Local）のセキュリティ グループを同期しないため、現状の Group2 は条件付きアクセスで使用できません。
- セキュリティ グループのスコープが Global または Universal であれば同期対象になります。したがって、Group2 を Universal に変更すると Azure AD に同期され、条件付きアクセスで利用可能になります。既に Universal の Group1 とユーザー（User1）はそのままで問題ありません。
- なお、オンプレミスのデバイス（Computer1）を条件付きアクセスのデバイス条件で評価したい場合は、別途 Azure AD Connect の「Hybrid Azure AD join」を構成して、デバイスを Azure AD に登録（ハイブリッド参加）する必要があります。ただし、本設問のネックは同期対象外の Domain Local グループ（Group2）であり、選択肢の中で要件を満たす決定打は A です。
- 公式ドキュメント（根拠）
  - Azure AD Connect の同期でサポートされるグループ種別（Domain Local は対象外、Global/Universal は対象）
https://learn.microsoft.com/azure/active-directory/hybrid/whatis-azure-ad-connect
  - 条件付きアクセスの割り当て（ユーザーとグループ）
https://learn.microsoft.com/azure/active-directory/conditional-access/concept-conditional-access-policies#assignments
  - ハイブリッド Azure AD 参加（オンプレ AD 参加デバイスを Azure AD に登録）
https://learn.microsoft.com/azure/active-directory/devices/hybrid-azuread-join-plan

## 問題33
ドメイン内のすべてのコンピューターが、adatum.com ゾーンで名前解決を行う際に DNSSEC を使用するようにする必要があります。

## 解答33
- ゾーン「adatum.com」に DNSSEC 署名を行う（例: Invoke-DnsServerZoneSign、または Add-DnsServerSigningKey と Set-DnsServerDnsSecZoneSetting を使用）。
- クライアントが参照する DNS サーバーで DNSSEC 検証を有効化し、adatum.com のトラスト アンカーを構成（AD 統合ゾーンならトラスト アンカーを AD DS に発行して複製）。
- ドメインにリンクした GPO で NRPT（名前解決ポリシー表）ルールを作成し、名前空間「adatum.com」に対して「DNSSEC を有効化／検証を必須」に設定して全クライアントに適用。

例（サーバー側 PowerShell の一例）:
Invoke-DnsServerZoneSign -ZoneName "adatum.com"
Add-DnsServerTrustAnchor -ZoneName "adatum.com" -DSRecord <必要に応じて指定>
Set-DnsServerSetting -EnableDnsSec $true

## 解説33
- Windows DNS でゾーンを DNSSEC 署名し、検証側（リカーシブ DNS サーバー）でトラスト アンカーを構成・検証を有効にすることで、権威応答に対する真正性検証が行えます。クライアントは DNS サーバーの検証結果を利用します。
- すべてのドメイン クライアントに DNSSEC 使用を強制するには、GPO で NRPT ルールを配布し、対象名前空間（adatum.com）に対して DNSSEC の有効化と検証必須を設定します。

根拠（Microsoft 公式ドキュメント）
- DNSSEC の概要と展開: https://learn.microsoft.com/windows-server/networking/dns/dnssec/dns-security-extensions-dnssec
- DNSSEC の展開手順（ゾーン署名、検証、トラスト アンカー）: https://learn.microsoft.com/windows-server/networking/dns/deploy/dns-deploy-dnssec
- ゾーン署名コマンド（Invoke-DnsServerZoneSign）: https://learn.microsoft.com/powershell/module/dnsserver/invoke-dnsserverzonesign
- トラスト アンカー構成（Add-DnsServerTrustAnchor）: https://learn.microsoft.com/powershell/module/dnsserver/add-dnsservertrustanchor
- NRPT（名前解決ポリシー表）の概要（GPO で DNSSEC 要求を構成）: https://learn.microsoft.com/windows/security/identity-protection/vpn/name-resolution-policy-table-nrpt-overview

## 問題34
オンプレミスのネットワークには、contoso.com という Active Directory ドメインがあります。
Azure AD テナントも保有しています。
Azure AD Connect cloud sync を使用して、contoso.com を Azure AD テナントと同期する計画です。
Azure AD Connect cloud sync が使用するアカウントを作成する必要があります。
どの種類のアカウントを作成すべきですか。
A. system-assigned managed identity
B. user
C. group managed service account (gMSA)
D. InetOrgPerson

## 解答34
C. group managed service account (gMSA)

## 解説34
- Azure AD Connect cloud sync では、オンプレミスにインストールする「Azure AD Connect Provisioning Agent」が Active Directory にアクセスするためのサービス アカウントとして gMSA（グループ管理サービス アカウント）の使用が推奨・前提とされています。インストール手順・前提条件でも gMSA の作成と指定が案内されています。
  - 公式ドキュメント（前提条件: gMSA アカウントの作成）: https://learn.microsoft.com/entra/identity/hybrid/connect/cloud-sync/how-to-prerequisites#create-a-gmsa-account
  - 公式ドキュメント（プロビジョニング エージェントのインストール時に gMSA を指定）: https://learn.microsoft.com/entra/identity/hybrid/connect/cloud-sync/how-to-install-agent
- 誤答:
  - A: system-assigned managed identity は Azure リソース用の ID で、オンプレミス サーバー上のエージェント実行には使用しません。
  - B: 通常ユーザー アカウントは cloud sync のサービス実行アカウントとしては想定されていません（従来の AAD Connect 同期の AD DS コネクタ アカウントとは要件が異なります）。
  - D: InetOrgPerson は人事オブジェクト クラスであり、サービス アカウント用途ではありません。

## 問題35
Windows Server を実行する Server1 があります。
Server1 にはネットワーク インターフェイスが 1 つだけあり、Hyper‑V の仮想スイッチ構成は添付の図のとおりです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/d9cdd0e8-1019-4f1a-a03c-a208d8d007c6.png)


図に示された情報に基づき、各文を完成させる選択肢をドロップダウンから選択してください。
注意: 各正解は 1 ポイントの価値があります。

- On Server1, you can create additional:「internal virtual switches only」または「private virtual switches only」または「private and internal virtual switches only」または「private, internal, and external virtual switches」
- Server1 can access network shares on virtual machines that are connected to:「VSwitch2 only」または「VSwitch3 only」または「VSwitch1 or VSwitch2 only」または「VSwitch2 or VSwitch3 only」または「VSwitch1, VSwitch2, or VSwitch3」

## 解答35
- On Server1, you can create additional: private and internal virtual switches only
- Server1 can access network shares on virtual machines that are connected to: VSwitch2 or VSwitch3 only

## 解説35
- 外部スイッチは物理 NIC にバインドして作成します。1 枚の物理 NIC を複数の外部スイッチで共有することはできないため、追加で作成できるのは物理 NIC を消費しない private/internal のみです（private と internal は複数作成可）。
- 到達性の違い:
  - Private: VM 同士のみ通信。ホスト（管理 OS）は通信不可。
  - Internal: ホストと VM 間で通信可。
  - External: 「管理オペレーティング システムにこのネットワーク アダプターの共有を許可」を有効化すると、ホスト用 vNIC（例: vEthernet）が作成され、ホストと VM も相互到達可。図ではホスト用アダプターが示されているため、VSwitch3 でもホスト→VM の共有アクセスが可能です。従ってホストが VM の共有にアクセスできるのは VSwitch2（Internal）または VSwitch3（External）です。
- 公式ドキュメント（根拠）
  - Hyper‑V 仮想スイッチの種類（External/Internal/Private）と機能: https://learn.microsoft.com/windows-server/virtualization/hyper-v-virtual-switch/hyper-v-virtual-switch
  - 仮想スイッチの作成（外部スイッチは物理 NIC に接続し、管理 OS の共有設定で vNIC を作成）: https://learn.microsoft.com/windows-server/virtualization/hyper-v/get-started/create-a-virtual-switch-for-hyper-v

## 問題36
あなたのネットワークには Active Directory Domain Services (AD DS) ドメインがあります。
ドメインには、次の表に示すオフィスが含まれています。
|Location|Number of VPN servers|Number of remote users|
|---|---|---|
|Boston|2|30|
|Dallas|2|50|
|Seattle|4|100|

すべてのリモート接続に対してネットワーク アクセス ポリシーを適用するため、NPS1 という名前の Network Policy Server (NPS) を展開する必要があります。
NPS1 に追加すべき RADIUS クライアントの最小数はいくつですか？
A. 8
B. 180
C. 1
D. 188
E. 3

## 解答36
A. 8

## 解説36
- NPS における「RADIUS クライアント」は、無線アクセスポイントや 802.1X 認証スイッチ、VPN サーバーなどのネットワーク アクセス サーバー (NAS) を指します。ユーザーやサイト数ではなく、実際に RADIUS 認証要求を転送してくる各 NAS デバイスを個別に RADIUS クライアントとして登録します。
- 本設問では RADIUS を話す NAS は「VPN サーバー」です。したがって拠点合計の VPN サーバー台数 2 + 2 + 4 = 8 台を NPS1 に RADIUS クライアントとして追加する必要があります。
- 参考 (Microsoft 公式ドキュメント):
  - 「RADIUS クライアントは NAS (VPN サーバー、無線 AP、802.1X スイッチ等) です」: https://learn.microsoft.com/ja-jp/windows-server/networking/technologies/nps/nps-radius-clients-servers
  - 「各 RADIUS クライアント (NAS) を NPS に IP アドレスまたは FQDN で登録して構成する」: https://learn.microsoft.com/ja-jp/windows-server/networking/technologies/nps/nps-radius-clients-configure

## 問題37
オンプレミスのネットワークには Server1 というサーバーがあり、IP アドレス空間は 192.168.10.0/24 を使用しています。
Azure には Subnet1 というサブネットを含む仮想ネットワークがあり、Subnet1 も 192.168.10.0/24 を使用しています。
既存の Server1 の IP アドレスを維持したまま Server1 を Subnet1 に移行するために、Azure Extended Network を使用する必要があります。
最小で何台の仮想マシンをデプロイする必要がありますか。
Answer Area の各項目から選択してください。
注: 正解 1 つにつき 1 点です。

Answer Area
Virtual machines that run Windows Server 2022 Azure Edition:「0」or「1」or「2」
Virtual machines that run Windows Server 2019 or Windows Server 2022:「0」or「1」or「2」

## 解答37
- Virtual machines that run Windows Server 2022 Azure Edition: 1
- Virtual machines that run Windows Server 2019 or Windows Server 2022: 1

## 解説37
- Azure Extended Network は、オンプレミスのサブネットを Azure に“拡張”し、移行後も同じプライベート IP を維持できるようにする機能です。ワークロード側は Windows Server 2022 Datacenter: Azure Edition を必要とします（Azure Extended Networking 機能の対象）。
  - 公式: 「Windows Server Datacenter: Azure Edition の機能（Azure Extended Networking）」
    - https://learn.microsoft.com/windows-server/get-started/azure-edition#azure-extended-networking
- さらに、Azure 側にはトラフィックをブリッジ/プロキシするための Azure Extended Network のプロキシ VM（アプライアンス）を 1 台デプロイします。これは Windows Server 2019 または Windows Server 2022 で実行され、最小構成では 1 台で可。高可用性が必要な場合にのみ複数台を配置します。
  - 公式: 「Windows Admin Center で Azure Extended Network をセットアップ（Azure へのプロキシ VM のデプロイが必要）」
    - https://learn.microsoft.com/windows-server/manage/windows-admin-center/azure/azure-extended-network

以上より、最小台数は Azure Edition のワークロード VM を 1 台、プロキシ用の Windows Server 2019/2022 VM を 1 台の計 2 台となります。

## 問題38
オンプレミスの Windows Server を実行するサーバーが次のとおりあります。

|Name|Local content|
|---|---|
|Server1|D:\Folder1\File1.docx|
|Server2|D:\Data1\File3.docx|

Azure には share1 という名前の Azure ファイル共有があり、File2.docx と File3.docx の 2 つのファイルが保存されています。

次のエンドポイントを含む Azure File Sync の同期グループを作成します。
- share1（クラウド エンドポイント）
- Server1 の D:\Folder1（サーバー エンドポイント）
- Server2 の D:\Data1（サーバー エンドポイント）

次の各記述について、正しければ「Yes」、誤りであれば「No」を選択してください。
注意: 各正解は 1 ポイントの価値があります。

|Statements|Yes|No|
|---|---|---|
|You can create a file named File2.docx in D:\Folder1 on Server1.| | |
|You can create a file named File1.docs in D:\Data1 on Server2.| | |
|File3.docx will sync to Server1.| | |

## 解答38
|Statements|Yes|No|
|---|---|---|
|You can create a file named File2.docx in D:\Folder1 on Server1.| |No|
|You can create a file named File1.docs in D:\Data1 on Server2.|Yes| |
|File3.docx will sync to Server1.|Yes| |

## 解説38
- 同期グループでは、クラウド エンドポイント（Azure ファイル共有）と複数のサーバー エンドポイント間で同一の名前空間を共有し、ファイルは双方向に同期されます。share1 に既に存在する File2.docx は D:\Folder1（Server1）へ同期されるため、そのフォルダーで「新規に同名ファイルを作成する」ことはできません（すでに同名ファイルが存在するため）。
- D:\Data1（Server2）に File1.docs は存在しないため、新規作成可能です。
- File3.docx は share1 と Server2 の両方に存在し、同期グループ内のすべてのエンドポイントへ伝搬するため、Server1 にも同期されます。重複や同時更新が発生した場合は Azure File Sync の競合解決（原則 Last writer wins、敗者はリネーム保存）により名前空間が維持されます。
- 根拠（Microsoft 公式ドキュメント）
  - Azure File Sync の概要（エンドポイント、双方向同期、名前空間の共有）: https://learn.microsoft.com/azure/storage/file-sync/file-sync-overview
  - ファイル競合の動作（競合解決とリネーム、最終書き込み優先）: https://learn.microsoft.com/azure/storage/file-sync/file-sync-troubleshoot#file-conflicts
  - 設計/展開の基本（クラウド エンドポイントとサーバー エンドポイントの関係）: https://learn.microsoft.com/azure/storage/file-sync/file-sync-planning

## 問題39
あなたには、次の表に示す仮想マシンを含む Azure サブスクリプションがあります。
|名前|オペレーティング システム|
|---|---|
|VM1|Windows Server 2022 Datacenter:Azure Edition|
|VM2|Windows Server 2022 Datacenter:Azure Edition Core|
|VM3|Windows Server 2022 Datacenter|
|VM4|Windows Server 2019 Datacenter|

あなたは Windows Server 用の Azure Automanage を実装する予定です。
オペレーティング システムの前提条件を特定する必要があります。
どの仮想マシンが Hotpatch をサポートし、どの仮想マシンが SMB over QUIC をサポートするかを特定してください。
解答するには、解答エリアで適切なオプションを選択してください。
注: 各正解は 1 点の価値があります。
解答エリア
Hotpatch:「VM1 only」または「VM2 only」または「VM1 and VM2 only」または「VM1, VM2, and VM3 only」または「VM1, VM2, VM3, and VM4」
SMB over QUIC:「VM1 only」または「VM2 only」または「VM1 and VM2 only」または「VM1, VM2, and VM3 only」または「VM1, VM2, VM3, and VM4」

## 解答39
Hotpatch: VM1 and VM2 only
SMB over QUIC: VM1 and VM2 only

## 解説39
- Hotpatch は Windows Server Azure Edition 専用機能であり、Windows Server 2022 Datacenter: Azure Edition（Desktop と Core の両方）でサポートされます。通常の Windows Server 2022 Datacenter や 2019 Datacenter ではサポートされません。
  - 参考: Microsoft Docs「Automanage Hotpatch」(前提条件に Azure Edition を明記) https://learn.microsoft.com/azure/automanage/automanage-hotpatch
- SMB over QUIC のサーバー機能は Windows Server 2025、または Windows Server 2022 Datacenter: Azure Edition でサポートされます。通常の Windows Server 2022 Datacenter や 2019 ではサーバー側の SMB over QUIC は利用できません。
  - 参考: Microsoft Docs「SMB over QUIC の概要」(サーバー要件に Windows Server 2025 または Windows Server 2022 Datacenter: Azure Edition を明記) https://learn.microsoft.com/windows-server/storage/file-server/smb-over-quic

## 問題40
オンプレミスのネットワークには Active Directory Domain Services (AD DS) ドメインがあります。Microsoft Entra Connect cloud sync を使用して、そのドメインを Microsoft Entra テナントと同期する計画です。次の要件を満たす必要があります。
* ドメインと Microsoft Entra ID を同期するために必要なソフトウェアをインストールする。
* パスワード ハッシュ同期を有効化する。

何をインストールし、パスワード ハッシュ同期の有効化には何を使用すべきですか。Answer Area から選択してください。

Answer Area
Install:「Active Directory Administrative Center」or「Microsoft Entra Connect」or「The AD FS Management console」or「The Microsoft Entra Connect provisioning agent」
Use:「Active Directory Administrative Center」or「Microsoft Entra Connect」or「The AD FS Management console」or「The Azure portal」

## 解答40
- Install: The Microsoft Entra Connect provisioning agent
- Use: The Azure portal

## 解説40
- Microsoft Entra Connect cloud sync では、オンプレミスの Windows Server に「Microsoft Entra Connect provisioning agent（旧 Azure AD Connect Provisioning Agent）」をインストールして同期を行います。これは cloud sync 専用のエージェントです。
  - 公式ドキュメント（エージェントのインストール）: https://learn.microsoft.com/entra/identity/hybrid/connect/cloud-sync/how-to-install-agent
- パスワード ハッシュ同期は Azure ポータルで cloud sync 構成を作成/編集する際に有効化します（クラウド同期の構成は Azure ポータルで実施）。
  - 公式ドキュメント（クラウド同期の構成: Azure ポータルでの手順）: https://learn.microsoft.com/entra/identity/hybrid/connect/cloud-sync/how-to-configure

## 問題41
Azure サブスクリプションに、次の図のとおり VM1 という仮想マシンがあります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ac35d725-9905-4199-aa01-0a3652e25a40.png)

（図より: VM1 は Location: East US、Availability zone: Zone 1）

サブスクリプションには次のディスクがあります。
|Name|Resource group|Location|Availability zone|
|---|---|---|---|
|Disk1|RG1|East US|None|
|Disk2|RG2|East US|1|
|Disk3|RG1|Central US|None|
|Disk4|RG1|Central US|1|

どのディスクを VM1 にデータ ディスクとしてアタッチできますか。
A. Disk4 only
B. Disk1, Disk3, and Disk4 only
C. Disk1 and Disk2 only
D. Disk2 and Disk4 only
E. Disk1, Disk2, Disk3, and Disk4
F. Disk2 only

## 解答41
C. Disk1 and Disk2 only

## 解説41
- データ ディスクは VM と同じリージョンである必要があります。よって Central US の Disk3/4 は不可です。
  - 根拠: 「VM にマネージド ディスクをアタッチするには、VM とディスクは同じリージョンに存在している必要があります。」
    - https://learn.microsoft.com/azure/virtual-machines/windows/attach-managed-disk-portal
- VM1 は East US の可用性ゾーン 1（ゾーナル VM）です。ゾーナル ディスクは同一ゾーンの VM にのみアタッチ可能です（Disk2 は East US/Zone 1 なので可）。一方、可用性ゾーンを指定しないディスク（Availability zone=None）は「リージョン ディスク」であり、特定のゾーンに固定されません。リージョン ディスクは、同一リージョン内であれば任意のゾーンの VM にアタッチ可能です。そのため East US/Zone=None の Disk1 も VM1 にアタッチ可能です。
  - 根拠: Zonal disks: 「ゾーナル ディスクは特定の可用性ゾーンに固定され、同じゾーンの VM にのみ接続できます。」
    - https://learn.microsoft.com/ja-jp/azure/virtual-machines/disks-redundancy?tabs=portal#zonal-disks

## 問題42
あなたのネットワークには、Active Directory Domain Services (AD DS) ドメインがあります。
グループ ポリシーの基本設定（Group Policy Preferences）を含む GPO1 という名前のグループ ポリシー オブジェクト (GPO) があります。
GPO1 をドメインにリンクする予定です。
GPO1 の基本設定がドメイン メンバー サーバーにのみ適用され、ドメイン コントローラーやクライアント コンピューターには適用されないようにする必要があります。
GPO1 に含まれるその他すべてのグループ ポリシーの設定は、すべてのコンピューターに適用されなければなりません。
管理上の労力は最小限に抑える必要があります。
どの種類のアイテム レベル ターゲティングを使用するべきですか。
A. Domain
B. Operating System
C. Security Group
D. Environment Variable

## 解答42
B. Operating System

## 解説42
- 要件は「基本設定（Preferences）のみをメンバー サーバーに適用し、ドメイン コントローラーとクライアントは除外」すること。グループ ポリシーの基本設定はアイテム レベル ターゲティング（Item-level targeting、ILT）で適用対象を細かく絞り込めます。Operating System ターゲットでは「Product type（製品種別）」として Workstation／Server／Domain Controller を判別できるため、Product type = Server を指定すれば「メンバー サーバー」のみに適用され、Domain Controller は除外されます（クライアント OS は Workstation）。
  - 根拠（Microsoft 公式ドキュメント）：
    - アイテム レベル ターゲティングの概要と使い方（Group Policy Preferences）: https://learn.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc731745(v=ws.11)
    - ターゲット項目「Operating System」（製品種別として Workstation／Server／Domain Controller を判別可能）: https://learn.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc733022(v=ws.11)#operating-system
- 他の「グループ ポリシー設定（Preferences 以外）」は GPO の通常の適用スコープに従い、すべてのコンピューターに適用されます。Preferences 側だけに ILT を設定することで、最小の運用コストで要件を満たせます（Security Group による管理はメンバー管理の手間が増えるため本要件では最小化に反します）。

## 問題43
contoso.com という Azure Active Directory Domain Services (Azure AD DS) ドメインがあります。
最小権限の原則に従って、ある管理者に GPO (Group Policy Object) を管理する権限を付与する必要があります。
どのグループにその管理者を追加すべきですか。
A. AAD DC Administrators
B. Domain Admins
C. Schema Admins
D. Enterprise Admins

## 解答43
A. AAD DC Administrators

## 解説43
- Azure AD DS では、GPO の作成・編集などの管理タスクは「AAD DC Administrators」グループのメンバーに委任されます。このグループはマネージド ドメインに対して必要最小限のドメイン管理とグループ ポリシー管理の権限を持ちます。Enterprise Admins や Schema Admins は Azure AD DS では使用せず、最小権限にも反します。Domain Admins ではなく、AAD DC Administrators への追加が推奨です。
  - 公式ドキュメント: 「Azure AD DS のグループ ポリシーの管理 — AAD DC Administrators のメンバーは GPO を作成/編集可能」
    - https://learn.microsoft.com/azure/active-directory-domain-services/manage-group-policy
  - 公式ドキュメント: 「AAD DC Administrators グループの役割（マネージド ドメインの管理権限の委任）」
    - https://learn.microsoft.com/azure/active-directory-domain-services/overview#administration

## 問題44
あなたのネットワークには 2 つの Active Directory フォレストと、次の図のとおりのドメイン信頼が存在します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/4a45ca51-0697-4673-aac4-ff62adf17ebb.png)

ドメイン信頼の構成は次のとおりです:
* Name: adatum.com
* Type: External
* Direction: One-way, outgoing
* Outgoing trust authentication level: Domain-wide authentication

|Name|Domain|
|---|---|
|User1|adatum.com|
|User2|contoso.com|
|User3|east.contoso.com|

フォレストには次のネットワーク共有が存在します。
|Name|In domain|
|---|---|
|Share1|adatum.com|
|Share2|contoso.com|
|Share3|east.contoso.com|

次の各記述について、正しい場合は Yes、そうでない場合は No を選択してください。
注: 各正解は 1 点の価値があります。

|Statements|Yes|No|
|---|---|---|
|User1 can be assigned permissions for Share3.| | |
|User2 can be assigned permissions for Share1.| | |
|User3 can be assigned permissions for Share1.| | |

## 解答44
|Statements|Yes|No|
|---|---|---|
|User1 can be assigned permissions for Share3.|Yes| |
|User2 can be assigned permissions for Share1.| |No|
|User3 can be assigned permissions for Share1.| |No|

## 解説44
- 外部信頼 (External trust) は非推移であり、1 方向または 2 方向に構成できます。設問の「One-way, outgoing」は east.contoso.com から adatum.com への送信 (outgoing) 信頼で、east.contoso.com が adatum.com を信頼する (trusting → trusted) ことを意味します。したがって、trusted 側 (adatum.com) のユーザーは trusting 側 (east.contoso.com) のリソースに対してアクセス許可を割り当て可能です。よって User1 に Share3 (east.contoso.com) のアクセス許可を割り当て可能 (Yes) です。一方、外部信頼は非推移のため contoso.com → adatum.com への信頼は存在せず、また 1 方向のため east.contoso.com のユーザー (User3) や contoso.com のユーザー (User2) を adatum.com 側のリソース (Share1) に割り当てることはできません (No)。
  - 参考: Microsoft Learn「Active Directory のトラスト」(信頼の方向/外部信頼/非推移)
    https://learn.microsoft.com/windows-server/identity/ad-ds/plan/active-directory-trusts
- 「Domain-wide authentication」が外部信頼に設定されている場合、trusted ドメイン (adatum.com) のすべてのユーザーが trusting ドメイン (east.contoso.com) での認証を暗黙に許可され、選択的認証の個別許可は不要です。これは User1 を Share3 に割り当て可能である根拠を補強します。
  - 参考: 同上ドキュメント内「Authentication (Domain-wide/Selective authentication)」

## 問題45
Microsoft Defender for Cloud を使用して DC1 のセキュリティ構成を監視する必要があります。
必要なソース ファイルは \\dc1.contoso.com\install というフォルダーにあります。

## 解答45
次の手順で、最小権限かつ確実に DC1 のセキュリティ構成を Defender for Cloud で監視できるようにします。

1. DC1 に Azure Arc エージェントをインストールして Azure に接続する
   - DC1 で \\dc1.contoso.com\install を開き、Azure Connected Machine agent（Azure Arc エージェント、AzureConnectedMachineAgent.msi など）を実行してインストールする。
   - インストール ウィザードでサブスクリプション／リソース グループに接続し、DC1 を「Arc 対応サーバー」として登録する。

2. ゲスト構成（Guest Configuration）でセキュリティ構成の評価を有効化する
   - Azure ポータルで Microsoft Defender for Cloud を開く。
   - 環境設定（Environment settings）または推奨事項から、Arc 対応サーバー（DC1）に「Guest Configuration」拡張機能をデプロイするように構成する（推奨事項: “Vulnerabilities in security configuration on your Windows machines should be remediated (powered by Guest Configuration)” を修正し、拡張機能を有効化）。
   - これにより、Windows セキュリティ ベースライン（Azure Compute/Windows セキュリティ ベースライン）に基づく準拠状況が継続的に評価され、Defender for Cloud の推奨事項/コンプライアンスに表示される。

補足: 既に Azure VM であれば VM 拡張機能として自動展開されますが、オンプレミス サーバーは Arc を介して拡張機能を適用する必要があります。

## 解説45
- オンプレミスの Windows サーバーを Microsoft Defender for Cloud で評価するには、まず Azure Arc に接続（Azure Connected Machine agent のインストール）して Azure リソースとして認識させます。
  - 公式ドキュメント: Azure Arc 対応サーバーの概要／Windows への接続
    - https://learn.microsoft.com/azure/azure-arc/servers/overview
    - https://learn.microsoft.com/azure/azure-arc/servers/onboard-windows
- Defender for Cloud の「セキュリティ構成の脆弱性（Guest Configuration による）」の評価は、Azure Policy の Guest Configuration によって実施され、対象マシンに Guest Configuration 拡張機能が必要です。
  - 公式ドキュメント: Guest Configuration（Azure Policy）概要／拡張機能
    - https://learn.microsoft.com/azure/governance/policy/concepts/guest-configuration
  - 公式ドキュメント: Defender for Cloud の推奨事項（Guest Configuration によるセキュリティ構成の評価）
    - https://learn.microsoft.com/azure/defender-for-cloud/recommendations-reference#guest-configuration

この構成により、DC1 のセキュリティ構成が Defender for Cloud 上で継続的に監視・評価されます。

## 問題46
あなたには Server1 という Windows Server コンテナー ホストと、Azure サブスクリプションがあります。
そのサブスクリプションに、Registry1 という名前の Azure Container Registry (ACR) をデプロイしました。
Server1 上で、image1 という名前のコンテナー イメージを作成しました。
このイメージを Registry1 に保存する必要があります。
Server1 上で実行するコマンドはどれですか。
回答するには、解答エリアで適切なオプションを選択してください。
注: 正解 1 つにつき 1 点です。

Answer Area
<「docker」or「azcopy」or「xcopy」or「git」> <「export」or「import」or「pull」or「push」>  Registry1.Azurecr.io /image1

## 解答46
Docker push Registry1.azurecr.io/image1

## 解説46
- ACR は Docker Registry v2 互換のレジストリであり、イメージの保存は docker push で行います（事前にレジストリのログインと、レジストリ名でのタグ付けが必要）。azcopy/xcopy/git は対象外、export/import はローカルへの書き出し/読み込みでありレジストリへの保存ではありません。
  - 根拠（Microsoft 公式ドキュメント）:
    - Docker CLI を使用した ACR へのイメージのプッシュ手順（タグ付けと push）: https://learn.microsoft.com/azure/container-registry/container-registry-get-started-docker-cli?tabs=azure-cli#push-image-to-acr
    - ACR の概要（Docker 互換レジストリ）: https://learn.microsoft.com/azure/container-registry/container-registry-intro

## 問題47
グループ メンバーシップ「MemberServers」にのみ適用される GPO 名「GPO1」を作成する必要があります。

## 解答47
- GPMC で新しい GPO「GPO1」を作成し、対象サーバーが存在する OU（またはドメイン）にリンクする。
- GPO の「セキュリティ フィルター」で「Authenticated Users」を削除し、代わりにグループ「MemberServers」を追加する（MemberServers に「読み取り」「グループ ポリシーの適用」を付与）。
- 併せて GPO の「委任」タブで「Authenticated Users」または「Domain Computers」に「読み取り」権限のみを付与しておく（処理時の読み取りを確保）。

## 解説47
- GPO を適用するには「スコープに含まれている」ことに加えて、対象ユーザー/コンピューターがその GPOに対して「読み取り (Read)」と「グループ ポリシーの適用 (Apply Group Policy)」の両方の権限を持つ必要があります。セキュリティ フィルタリングで対象グループ（MemberServers）を指定するのが推奨です。
  - 参考: Microsoft Learn「Group Policy の概要（Security filtering と WMI filtering）」
    https://learn.microsoft.com/windows-server/identity/ad-ds/get-started/group-policy/group-policy-overview
- GPO の作成・リンクは GPMC から実施します。スコープ（リンク先 OU/ドメイン）とセキュリティ フィルタリングの両方で対象を絞ることで、MemberServers のみが GPO1 を受け取るようにできます。
  - 参考: Microsoft Learn「Group Policy Management Console (GPMC)」
    https://learn.microsoft.com/windows-server/identity/ad-ds/get-started/group-policy/group-policy-management-console

## 問題48
あなたのネットワークには、クライアント用の VLAN が 2 つと、データセンター用の VLAN が 1 つあります。
各 VLAN には IPv4 サブネットが割り当てられています。
現在、すべてのクライアント コンピューターは静的 IP アドレスを使用しています。

データセンター内の VLAN に DHCP サーバーをデプロイする予定です。

DHCP サーバーを使用して、すべてのクライアント コンピューターに IP 構成を提供する必要があります。

作成すべきスコープ数と DHCP リレーの最小数はどれですか。
回答するには、解答エリアで適切なオプションを選択してください。

NOTE: Each correct selection is worth one point.

Answer Area
DHCP scopes:「1」or「2」or「3」or「4」
DHCP relays:「1」or「2」or「3」or「4」

## 解答48
|項目|選択|
|---|---|
|DHCP scopes|2|
|DHCP relays|2|

## 解説48
- 各クライアント VLAN は別サブネットであるため、DHCP サーバーは「サブネットごとに少なくとも 1 つのスコープ」が必要です。今回はクライアント サブネットが 2 つなのでスコープは 2 つです。
  - 根拠（Microsoft 公式ドキュメント）: スコープは「1 つのサブネット上で割り当て可能な IP アドレス範囲」であり、複数サブネットを扱うには複数スコープが必要 https://learn.microsoft.com/windows-server/networking/technologies/dhcp/dhcp-overview
- DHCP サーバーはブロードキャストを越えて他サブネットのクライアントに直接応答できないため、各リモート サブネット（= 各クライアント VLAN）で DHCP リレー（IP ヘルパー/BOOTP Relay）を 1 つずつ構成する必要があります。よってリレーは 2 つです（データセンター VLAN には不要）。
  - 根拠（Microsoft 公式ドキュメント）: DHCP はルーターを越えるには DHCP リレー エージェントが必要 https://learn.microsoft.com/windows-server/networking/technologies/dhcp/dhcp-overview

## 問題49
あなたは Server1 という名前の Windows Server コンテナー ホストを所有しています。
df1 という名前の Dockerfile を作成しました。
dt1 を使用してコンテナー イメージを生成する必要があります。
実行すべきコマンドはどれですか?
A. docker images
B. docker exec
C. docker build
D. docker create

## 解答49
C. docker build

## 解説49
- コンテナー イメージは Dockerfile から「docker build」で生成します。タグ名を dt1 とし、Dockerfile 名が df1 の場合の例: docker build -t dt1 -f df1 .
  - 参考: Microsoft Learn「Windows Server での Windows コンテナーのクイック スタート」(docker build によるイメージ作成の手順を記載)
    https://learn.microsoft.com/virtualization/windowscontainers/quick-start/quick-start-windows-server
- 他の選択肢:
  - docker images はローカル イメージの一覧表示。
  - docker exec は実行中コンテナー内でコマンドを実行。
  - docker create は既存イメージからコンテナーを作成（ビルドはしない）。

## 問題50
分離（ディスアグリゲート）構成のクラスターを展開しています。
この展開には、Windows Server を実行するスケールアウト ファイル サーバー（SOFS）クラスターと、Hyper-V 役割が有効化されたコンピュート クラスターが含まれます。
Storage Quality of Service（QoS）を実装する必要があります。
SOFS クラスターと Hyper-V クラスター間の帯域幅使用量を制御できるようにする必要があります。
各クラスターで実行すべき cmdlet はどれですか。
回答するには、適切な cmdlet を正しいクラスターにドラッグしてください。
各 cmdlet は 1 回以上使用される場合も、使用されない場合もあります。
ペイン間の仕切りをドラッグするか、スクロールして内容を表示する必要がある場合があります。
注: 各正解は 1 ポイントの価値があります。

選択肢:
- New-StorageQosPolicy
- Set-VMHardDiskDrive
- Set-VMSan

回答欄
SOFS:
Hyper-V:

## 解答50
SOFS: New-StorageQosPolicy
Hyper-V: Set-VMHardDiskDrive

## 解説50
- 集中管理型の Storage QoS ポリシーは SOFS クラスター側で作成・保持します。New-StorageQosPolicy は「スケールアウト ファイル サーバー クラスター上に新しい Storage QoS ポリシーを作成」するためのコマンドレットです。参考: https://learn.microsoft.com/powershell/module/storage/new-storageqospolicy
- Hyper-V 側では、各 VM の仮想ハード ディスクに SOFS 上の QoS ポリシーを割り当てて帯域・IOPS を制御します。Set-VMHardDiskDrive の -QoSPolicyID や -QoSPolicyFriendlyName によりポリシーを適用できます。参考: https://learn.microsoft.com/powershell/module/hyper-v/set-vmharddiskdrive
- Storage QoS の概要として、SOFS と Hyper-V を分離した構成で集中ポリシーにより帯域・IOPS を制御できることが解説されています。参考: https://learn.microsoft.com/windows-server/storage/storage-qos/storage-qos-overview

## 問題51
稼働中の仮想マシンをダウンタイムなしに SRV1 と SRV2 の間で移動できるように Hyper-V を構成する必要があります。
現時点で仮想マシンを移動する必要はありません。

## 解答51
SRV1 と SRV2 の両方で Hyper‑V のライブ マイグレーションを有効化し、Kerberos 認証を用いた制約付き委任（Microsoft Virtual System Migration Service への委任）を構成する。

## 解説51
- ライブ マイグレーションは、稼働中の仮想マシンをホスト間で「停止させずに」移動する機能です。Microsoft 公式ドキュメントは、ライブ マイグレーションによって「実行中の仮想マシンを別の物理コンピューターに移動」できることを明言しています（無停止移行）［参考: Live migration overview］。
- フェールオーバー クラスタなしでも、2 台のスタンドアロン Hyper‑V ホスト間でライブ マイグレーション（共有なしライブ マイグレーション）を行えます。そのために、各ホストで [Hyper‑V 設定] → [ライブ マイグレーション] にて「ライブ マイグレーションを許可」し、ネットワークおよび認証方式を構成します。遠隔からの移行を可能にし自動化するには、Active Directory で Kerberos の制約付き委任（Microsoft Virtual System Migration Service への委任、必要に応じて CIFS も）を設定します［参考: Perform live migration without Failover Clustering / Configure constrained delegation for live migration］。
- 本設問は「今は移動を行わない」ため、実際の移行作業は不要で、上記の事前構成（ライブ マイグレーション有効化と Kerberos 制約付き委任の設定）だけで要件を満たします。

参考（Microsoft 公式ドキュメント）:
- Live migration overview: https://learn.microsoft.com/windows-server/virtualization/hyper-v/manage/live-migration-overview
- Perform live migration without Failover Clustering: https://learn.microsoft.com/windows-server/virtualization/hyper-v/deploy/performing-live-migration-without-failover-clustering
- Configure constrained delegation for live migration: https://learn.microsoft.com/windows-server/virtualization/hyper-v/deploy/configure-constrained-delegation-for-live-migration

## 問題52
SRV1 上の「scope!」という名前の DHCP スコープが、クライアントの要求に応答できるようにする必要があります。

## 解答52
SRV1 上の DHCP スコープ「scope!」をアクティブ化（有効化）する。

## 解説52
- ポイント: DHCP スコープは作成直後は非アクティブで、アクティブ化しない限りクライアントへ IP アドレスをリースできません。したがって、スコープ「scope!」をアクティブ化すれば、クライアント要求に応答できるようになります。
- 参考手順（GUI）: 管理ツールから「DHCP」（`dhcpmgmt.msc`）を開き、[IPv4] → スコープ「scope!」を右クリック → [アクティブ化] を選択。
- ドメイン環境の注意: SRV1 が Active Directory ドメイン内の DHCP サーバーである場合、サーバー自体が AD DS に「承認」されている必要があります（未承認の DHCP サーバーはクライアントに応答しません）。

|状態|目印|意味|
|---|---|---|
|有効|緑の矢印|スコープはアクティブでリース可能|
|無効|赤の矢印|スコープは非アクティブでリース不可|

- Microsoft 公式ドキュメント（根拠）:
  - DHCP スコープの管理（アクティブ化/非アクティブ化の操作を含む）: https://learn.microsoft.com/windows-server/networking/technologies/dhcp/dhcp-manage-scopes
  - DHCP サーバーは AD DS で承認が必要（ドメイン内）: https://learn.microsoft.com/windows-server/networking/technologies/dhcp/dhcp-authorize-server

動作確認の例: クライアントで `ipconfig /renew` を実行して、スコープからのアドレス取得を確認します。

## 問題53
Active Directory Domain Services (AD DS) ドメインがあり、そのドメインには Group1 という名前のグループが含まれています。
Account1 という名前のグループ マネージド サービス アカウント (gMSA) を作成する必要があります。
ソリューションは、Group1 が Account1 を使用できるようにする必要があります。
スクリプトをどのように完成させるべきですか。
回答するには、解答エリアで適切なオプションを選択してください。
注: 各正解は 1 ポイントの価値があります。

解答エリア
```
<「Add-ADComputerServiceAccount」or「Install-ADServiceAccount」or「New-ADObject」or「New-ADServiceAccount」> "Account1" -DNSHostName "website.contoso.com" <「-AuthenticationPolicy」or「-Instance」or「-PrincipalsAllowedToDelegateToAccount」or「-PrincipalsAllowedToRetrieveManagedPassword」> "Group1"
```

## 解答53
New-ADServiceAccount "Account1" -DNSHostName "website.contoso.com" -PrincipalsAllowedToRetrieveManagedPassword "Group1"

## 解説53
- New-ADServiceAccount は、スタンドアロン MSA と gMSA を新規作成するコマンドレットです。gMSA を誰が使用できるかは、-PrincipalsAllowedToRetrieveManagedPassword で指定します。これにグループを指定すると、そのグループ（通常はコンピュータ アカウントを含む）が gMSA のパスワードを取得して使用できます。
  - 参考: New-ADServiceAccount（Microsoft Learn）
    https://learn.microsoft.com/powershell/module/activedirectory/new-adserviceaccount
- -PrincipalsAllowedToRetrieveManagedPassword パラメーターは、アカウントの管理パスワードを取得できるセキュリティ プリンシパル（ユーザー、コンピューター、またはグループ）を指定します。Group1 を指定することで、Group1 が Account1 を使用可能になります。
  - パラメーターの説明:
    https://learn.microsoft.com/powershell/module/activedirectory/new-adserviceaccount#-principalsallowedtoretrievemanagedpassword
- 参考（対比）: Add-ADComputerServiceAccount はサービス アカウントをコンピューターに関連付ける操作、Install-ADServiceAccount はローカル コンピューターへインストールする操作であり、作成ではありません。
  - Add-ADComputerServiceAccount:
    https://learn.microsoft.com/powershell/module/activedirectory/add-adcomputerserviceaccount
  - Install-ADServiceAccount:
    https://learn.microsoft.com/powershell/module/activedirectory/install-adserviceaccount

## 問題54
あなたのネットワークには、adatum.com という名前の Active Directory Domain Services (AD DS) ドメインがあります。

ドメインには、Server1 という名前のサーバーが 1 台と、User1、User2、User3 という 3 人のユーザーが存在します。
Server1 には Share1 という共有フォルダーがあり、次の構成になっています
（画像参照。FolderEnumerationMode は AccessBased）。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/76731044-2743-4670-8130-3f3723f4a285.png)

Share1 の共有アクセス許可は、[Share Permissions] タブに示されているとおりに構成されています（画像参照）。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/8a5f96fa-39e5-42d0-9e49-db985066327c.png)

Share1 には File1.txt というファイルが含まれており、File1.txt の詳細なセキュリティ設定は [File Permissions] タブに示されているとおりに構成されています（画像参照）。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ed26d86b-3891-46c4-a77a-e81971bddc5e.png)

次の各記述について、正しければ Yes を、誤りであれば No を選択しなさい。
注: 各正答は 1 ポイント。

|記述|Yes|No|
|---|---|---|
|User1 が \\Server1.adatum.com\Share1\ に接続したとき、File1.txt の所有権を取得できる。| | |
|User2 が \\Server1.adatum.com\Share1\ に接続したとき、File1.txt は表示される。| | |
|User3 が \\Server1.adatum.com\Share1\ に接続したとき、File1.txt は表示される。| | |

## 解答54
|記述|Yes|No|
|---|---|---|
|User1 が \\Server1.adatum.com\Share1\ に接続したとき、File1.txt の所有権を取得できる。|Yes| |
|User2 が \\Server1.adatum.com\Share1\ に接続したとき、File1.txt は表示される。|Yes| |
|User3 が \\Server1.adatum.com\Share1\ に接続したとき、File1.txt は表示される。| |No|

## 解説54
- 共有と NTFS の両方のアクセス許可が設定されている場合、実効アクセス許可は「より制限の厳しい方」になります。本設問では共有側は Domain Users に Full Control（フル コントロール）が付与されており、最終的な可否は File1.txt の NTFS 権限で決まります。
  - 根拠: Share と NTFS 権限の組み合わせ（Microsoft Learn）https://learn.microsoft.com/troubleshoot/windows-server/networking/share-and-ntfs-permissions
- 画像の File1.txt の NTFS 権限:
  - User1: Full control／User2: Read／User3: Write（所有者は Administrators）
- 記述1（Yes）: Full control には「所有権の取得 (Take ownership)」が含まれ、所有者でなくても自分自身を所有者にできます。
  - 根拠: 基本/詳細アクセス許可（Microsoft Learn）https://learn.microsoft.com/windows/security/identity-protection/access-control/basic-and-advanced-permissions
- 記述2（Yes）: Share1 は Access-Based Enumeration (ABE) が有効です。ABE はユーザーがアクセス権を持つ項目のみを表示します。User2 は File1.txt に Read を持つため、一覧に表示されます。
  - 根拠: ABE 概要（Microsoft Learn）https://learn.microsoft.com/windows-server/storage/file-server/enable-access-based-enumeration
- 記述3（No）: User3 は File1.txt に対して Write のみで Read を持ちません。Write は Read を含まないため、ABE により File1.txt は表示されません。
  - 根拠: 基本/詳細アクセス許可（Write は Read を含まない）https://learn.microsoft.com/windows/security/identity-protection/access-control/basic-and-advanced-permissions および ABE の仕様 https://learn.microsoft.com/windows-server/storage/file-server/enable-access-based-enumeration

## 問題55
Windows Server を実行する Server1 というサーバーがあり、C と D という名前の 2 つのドライブが含まれています。
Server1 は複数のファイル共有をホストしています。
ドライブ D で Data Deduplication（データ重複除去）を有効にし、ワークロードとして General purpose file server を選択しました。
最近変更または削除されたファイルによって消費される領域を最小化する必要があります。
何をすべきですか？
A. Set-DedupSchedule コマンドレットを実行し、最適化 (Optimization) ジョブを構成する。
B. set-dedupvolume コマンドレットを実行し、スクラビング (Scrubbing) ジョブを構成する。
C. set-Dedupvoiume コマンドレットを実行し、InputOutputScale 設定を構成する。
D. Set-DedupSchedule コマンドレットを実行し、GarbageCollection ジョブを構成する。

## 解答55
D. Set-DedupSchedule コマンドレットを実行し、GarbageCollection ジョブを構成する。

## 解説55
- 目的: 最近「変更」や「削除」に伴って参照されなくなったデータ チャンクを速やかに回収し、ディスク空き容量を増やすこと。
- 根拠: Data Deduplication の Garbage Collection ジョブは、参照のない（不要になった）データをチャンク ストアから削除して空き容量を回収する保守ジョブです。これに対し、Optimization は新規/変更ファイルの重複排除処理、Scrubbing は整合性チェックと修復を目的としており、空き容量の回収という観点では Garbage Collection が直接的に有効です。Garbage Collection のスケジュール構成は Set-DedupSchedule で行います。
- 参考（Microsoft 公式ドキュメント）:
  - Data Deduplication のジョブ（Optimization / Garbage Collection / Scrubbing）の役割: https://learn.microsoft.com/windows-server/storage/data-deduplication/overview
  - Set-DedupSchedule（-Type に GarbageCollection を指定してスケジュールを構成）: https://learn.microsoft.com/powershell/module/deduplication/set-dedupschedule

|ジョブ|主な目的|空き容量への効果|
|---|---|---|
|Optimization|重複の検出・チャンク化・重複除去（新規/変更ファイルの処理）|間接的（重複除去により削減）|
|Scrubbing|データ整合性の検証と修復|ほぼなし（信頼性向上が目的）|
|GarbageCollection|参照のないチャンクの削除によるスペース回収|直接的（削除/変更後の不要データを回収）|

## 問題56
Active Directory Domain Services (AD DS) サイトで、192.168.2.0 から 192.168.2.255 の IP アドレス範囲に関連付けられた Site2 という名前のサイトを作成する必要があります。

## 解答56
New-ADReplicationSite -Name "Site2"
New-ADReplicationSubnet -Name "192.168.2.0/24" -Site "Site2"

## 解説56
- 対応方針:
  - AD DS では「サイト（Site）」と「サブネット（Subnet）」を定義し、サブネットをサイトに関連付けることで、その IP 範囲のクライアントやドメイン コントローラー (DC) をサイトにマッピングします。
  - したがって、Site2 を作成し、サブネット 192.168.2.0/24（= 192.168.2.0 ～ 192.168.2.255）を Site2 に関連付ければ要件を満たします。
- PowerShell での正答（上記解答）:
  - New-ADReplicationSite は新しい AD DS サイト オブジェクトを作成します。
    - 公式: https://learn.microsoft.com/powershell/module/addsadministration/new-adreplicationsite
  - New-ADReplicationSubnet は CIDR 表記（例: 192.168.2.0/24）でサブネットを作成し、-Site パラメーターでサイトに関連付けます。
    - 公式: https://learn.microsoft.com/powershell/module/addsadministration/new-adreplicationsubnet
- GUI（Active Directory サイトとサービス）での参考手順（要点）:
  - dssite.msc を開く → 左ペインの「Sites」を右クリック → New Site → 名前に「Site2」→（任意で）DEFAULTIPSITELINK を選択
  - 左ペインの「Subnets」を右クリック → New Subnet → Prefix に「192.168.2.0/24」→ Site に「Site2」を指定 → OK
  - 変更はレプリケーション後に反映されます。
- 根拠（設計の考え方）:
  - サブネットはサイトに関連付けられ、クライアント/サーバーがどのサイトに属するかの決定に使われます（Sites の概念）。
    - 公式（Sites の説明）: https://learn.microsoft.com/windows-server/identity/ad-ds/plan/active-directory-logical-structure#sites

## 問題57
あなたのネットワークには Active Directory Domain Services (AD DS) ドメインがあります。
ドメインには次の表に示すサーバーが含まれています。
|名前|種類|
|---|---|
|DC1|ドメイン コントローラー|
|Server1|メンバー サーバー|
|Server2|メンバー サーバー|

ドメインには次の表に示すユーザーが含まれています。
|名前|所属グループ|
|---|---|
|User1|Contoso\Administrators|
|User2|Contoso\Remote Management Users|
|User3|Server2\Power Users|

Server2 で Enable-PSRemoting コマンドレットを実行します。

次の各記述について、記述が正しければ Yes を、そうでなければ No を選択してください。
注: 各正解は 1 ポイントの価値があります。

|記述|Yes|No|
|---|---|---|
|User1 は Server1 から Server2 への PowerShell リモート セッションを確立できる。| | |
|User2 は Server2 から DC1 への PowerShell リモート セッションを確立できる。| | |
|User3 は Server1 から Server2 への PowerShell リモート セッションを確立できる。| | |

## 解答57
|記述|Yes|No|
|---|---|---|
|User1 は Server1 から Server2 への PowerShell リモート セッションを確立できる。|Yes| |
|User2 は Server2 から DC1 への PowerShell リモート セッションを確立できる。| |No|
|User3 は Server1 から Server2 への PowerShell リモート セッションを確立できる。| |No|

## 解説57
- 既定では、PowerShell リモート セッションを受け入れるコンピューター側（宛先）の既定エンドポイントは、ローカル Administrators グループのメンバーのみが接続できます。したがって管理者権限（ドメインの管理者権限を含む）を持つ User1 は Server2 にリモート接続できます。一方、User2（Remote Management Users）や User3（Power Users）は、既定設定のままでは接続できません。
- Enable-PSRemoting は「受信側（リモートされる側）」で実行して WinRM リスナーやファイアウォールを構成します。本設問では Server2 でのみ実行されているため、DC1 側については前提が不明であり、加えて非管理者の User2 では既定エンドポイントに接続できません。
- 非管理者に PowerShell リモート接続を許可するには、セッション構成（エンドポイント）の ACL を変更して対象ユーザー/グループに「実行」権限を付与する等の追加設定が必要です（Remote Management Users への追加だけでは不十分）。

- Microsoft 公式ドキュメント（根拠）:
  - Enable-PSRemoting（受信側で有効化する）: https://learn.microsoft.com/powershell/module/microsoft.powershell.core/enable-psremoting
  - リモート トラブルシューティング（既定では管理者のみが接続可能、非管理者はエンドポイント権限の構成が必要）: https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_Remote_Troubleshooting
  - セッション構成（非管理者を許可するにはエンドポイントの権限付与が必要）: https://learn.microsoft.com/powershell/module/microsoft.powershell.core/register-pssessionconfiguration

## 問題58
Azure サブスクリプションがあります。
サブスクリプションには、Windows Server を実行する VM1 という名前の仮想マシンが含まれています。PowerShell ランブックを使用して VM1 を管理する予定です。
ランブックを作成する必要があります。
最初に何を作成する必要がありますか？
A. Microsoft Power Automate フロー
B. Azure ワークブック
C. Azure Automation アカウント
D. Log Analytics ワークスペース

## 解答58
C. an Azure Automation account

## 解説58
- Azure のランブックは Azure Automation アカウント内で作成・実行・管理されます。したがって最初に Azure Automation アカウントを作成する必要があります。
  - 公式ドキュメント: 「Automation アカウントの作成」では、ランブック作成に先立ち Automation アカウントを用意する手順が示されています。
    https://learn.microsoft.com/azure/automation/automation-create-standalone-account?tabs=azureportal
  - 公式チュートリアル: 「PowerShell ランブックの作成」でも、前提として Automation アカウントを作成してからランブックを作成します。
    https://learn.microsoft.com/azure/automation/learn/automation-tutorial-runbook-textual-powershell?tabs=azureportal
- 他の選択肢について:
  - Power Automate フローはランブックの作成要件ではありません。
  - Azure ワークブックは可視化/ダッシュボード用途です。
  - Log Analytics ワークスペースは Update Management などで利用されますが、ランブックの作成自体には必須ではありません。

## 問題59
あなたのネットワークには、contoso.com という名前の Active Directory Domain Services (AD DS) フォレストがあります。
フォレスト ルート ドメインには、server1.contoso.com という名前のサーバーが含まれています。 
contoso.com フォレストと、fabrikam.com という AD DS フォレストとの間には双方向のフォレスト信頼があります。fabrikam.com フォレストには 10 個の子ドメインが含まれます。
あなたは、fabrikam\Group1 というグループのメンバーだけが server1.contoso.com に対して認証できるようにする必要があります。
最初に何をすべきですか？
A. fabrikam\Group1 を server1.contoso.com のローカル Users グループに追加する。
B. 信頼を片方向の外部信頼に変更する。
C. 信頼に対して SID フィルタリングを有効にする。
D. 信頼に対して選択的認証 (Selective authentication) を有効にする。

## 解答59
D

## 解説59
目的は、信頼先フォレスト（fabrikam.com）のうち特定グループ（fabrikam\Group1）だけが、信頼元フォレスト（contoso.com）の特定サーバー（server1.contoso.com）に対して認証できるよう制限することです。これを実現する標準手順は以下の通りです。
1) フォレスト信頼で「選択的認証 (Selective authentication)」を有効化する。
2) 対象コンピューター（server1）のコンピューター オブジェクトに対して、fabrikam\Group1 に「Allowed to authenticate（認証を許可）」権限を付与する。

Microsoft 公式ドキュメントでは、選択的認証を有効にすると、明示的に「Allowed to authenticate」許可を与えられたアカウントのみが、信頼元フォレスト内のリソースに対して認証できるようになると説明されています。そのため「最初に」行うべきは、信頼に対して選択的認証を有効化することです（その後、server1 で fabrikam\Group1 に Allowed to authenticate を付与）。

根拠（Microsoft 公式ドキュメント）:
- Securing Active Directory trusts – Use selective authentication: 選択的認証により、信頼先フォレストのうち認証を許可されたユーザー/グループだけが認証可能になる。https://learn.microsoft.com/windows-server/identity/ad-ds/plan/security-best-practices/securing-active-directory-trusts#use-selective-authentication
- Active Directory Domains and Trusts（Forest-wide authentication と Selective authentication の概要）: 選択的認証では対象コンピューターに対する「Allowed to authenticate」権限の付与が必要。https://learn.microsoft.com/windows-server/identity/ad-ds/manage/domains-and-trusts/active-directory-domains-and-trusts

（誤答のポイント）
- A: ローカル Users への追加は認証の可否制御ではなく、選択的認証なしではフォレスト全体のユーザーが認証できてしまう。
- B: 外部信頼への変更や片方向化は要件達成に不要で、かつ制御が粗い。
- C: SID フィルタリングは SID 履歴悪用の防止が目的で、認証のスコープ制御には使わない。

## 問題60
あなたのネットワークには、conto.com という名前の単一ドメインの Active Directory Domain Services (AD DS) フォレストが含まれています。
フォレストには、次の表に示すサーバーが含まれています。
|Name|Description|
|---|---|
|DC1|Domain controller|
|Server1|Member server|

あなたは Server1 に業務用 (LOB) アプリケーションをインストールする予定です。
アプリケーションはカスタムの Windows サービスをインストールします。
新しい企業のセキュリティ ポリシーでは、すべてのカスタム Windows サービスはグループ管理対象サービス アカウント (gMSA) のコンテキストで実行しなければならないとされています。
あなたはルート キーを展開しました。
新しいアプリケーションで使用する gMSA を作成、構成、およびインストールする必要があります。
どの 2 つの操作を実行する必要がありますか? 
それぞれの正解は解決策の一部を示します。
注: 各正解は 1 ポイントの価値があります。
A. Server1 で Install-ADServiceAccount コマンドレットを実行する。
B. DC1 で New-ADServiceAccount コマンドレットを実行する。
C. DC1 で Install-ADServiceAccount コマンドレットを実行する。
D. Server1 で Get-ADServiceAccount コマンドレットを実行する。
E. DC1 で Set_ADComputer コマンドレットを実行する。

## 解答60
A, B

## 解説60
- 正しい手順は、(1) AD に gMSA オブジェクトを作成する（New-ADServiceAccount）、(2) gMSA を使用するサーバー（Server1）にインストールする（Install-ADServiceAccount）です。gMSA 作成時には PrincipalsAllowedToRetrieveManagedPassword に Server1（または含むグループ）を指定して、Server1 がパスワードを取得できるようにします。
- C は gMSA を DC1 にインストールしてしまうため目的のサーバー（Server1）ではありません。D は情報取得のみでインストールは行いません。E は gMSA 構成とは無関係です（許可主体の設定は New-/Set-ADServiceAccount で行います）。
- Microsoft 公式ドキュメント（根拠）:
  - gMSA の概要と手順（作成・使用）: https://learn.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview
  - New-ADServiceAccount（gMSA の作成と PrincipalsAllowedToRetrieveManagedPassword の指定）: https://learn.microsoft.com/powershell/module/addsadministration/new-adserviceaccount
  - Install-ADServiceAccount（対象サーバーへの gMSA のインストール）: https://learn.microsoft.com/powershell/module/addsadministration/install-adserviceaccount

## 問題61
Windows Server を実行する Azure 仮想マシン VM1 があります。
Microsoft Defender for Cloud が有効な Azure サブスクリプションを所有しています。
Azure Policy の Guest Configuration 機能を使用して VM1 を管理できるようにする必要があります。
何をすべきですか？
A. PowerShell Desired State Configuration (DSC) 拡張機能を VM1 に追加する
B. VM1 をユーザー割り当てマネージド ID を使用するように構成する
C. VM1 をシステム割り当てマネージド ID を使用するように構成する
D. Custom Script Extension を VM1 に追加する

## 解答61
C. VM1 をシステム割り当てマネージド ID を使用するように構成する

## 解説61
- Azure Policy の Guest Configuration は、VM 内の構成評価/適用を行うために、VM 側でマネージド ID（特にシステム割り当てマネージド ID）を有効化することが前提条件です。ポリシーの前提条件を自動展開する組み込みイニシアチブでも、システム割り当てマネージド IDの有効化と Guest Configuration 拡張機能の展開が行われます。
  - 公式: 「Guest Configuration の概要（前提条件/要件）」
    https://learn.microsoft.com/azure/governance/policy/concepts/guest-configuration
  - 公式: 「ポータルで Guest Configuration ポリシーを割り当てる（前提条件として VM のシステム割り当てマネージド ID 有効化と拡張機能の展開）」
    https://learn.microsoft.com/azure/governance/policy/assign-policy-guest-configuration-portal
- DSC 拡張機能や Custom Script Extension は Guest Configuration の必須前提条件ではありません。ユーザー割り当てマネージド ID も利用可能なケースはありますが、組み込みの展開イニシアチブと前提条件ではシステム割り当てマネージド ID が用いられます。

## 問題62
あなたは次のリソースを含む Azure サブスクリプションを所有しています。
* Azure Log Analytics ワークスペース
* Azure Automation アカウント
* Azure Arc

オンプレミス サーバーの Server1 は Azure Arc にオンボードされています。
Azure Arc を使用して Server1 上の Microsoft 更新プログラムを管理する必要があります。
実行すべき 2 つのアクションはどれですか？
各正解はソリューションの一部を示します。
注: 各正解は 1 ポイントの価値があります。
A. server1.contoso.com のローカル Users グループに fabrikam\Group1 を追加する。
B. Server1 に Azure Monitor エージェントをインストールする。
C. Automation アカウントから、Server1 に対して Update Management を有効化する。
D. Log Analytics ワークスペースの「Virtual machines」データ ソースから、Server1 を接続する。

## 解答62
C, D

## 解説62
Azure Automation の Update Management（従来機能）で Azure Arc 対応サーバーを更新管理するには、以下が必要です。
- Update Management を有効化（Automation アカウントと Log Analytics ワークスペースを関連付け、対象マシンを登録）すること → C
- 対象マシンが Log Analytics ワークスペースに接続され、Log Analytics エージェント（MMA）がインストールされていること（Arc 対応サーバーはワークスペースの「Virtual machines」ブレードから接続操作で自動展開できる）→ D

補足:
- Azure Automation Update Management は Azure Monitor Agent (AMA) ではなく Log Analytics エージェント (MMA) を使用します。したがって B は不適切です。

根拠（Microsoft 公式ドキュメント）:
- Update Management の概要と有効化（Automation アカウントと Log Analytics の前提/有効化手順）
  https://learn.microsoft.com/azure/automation/update-management/overview
  https://learn.microsoft.com/azure/automation/update-management/enable-from-automation-account
- Log Analytics エージェントのインストール/接続（Arc 対応サーバーをワークスペースに接続して MMA を展開）
  https://learn.microsoft.com/azure/azure-monitor/agents/log-analytics-agent#ways-to-install
  https://learn.microsoft.com/azure/azure-arc/servers/monitor#connect-to-azure-monitor-logs
- エージェントのサポート状況（Update Management は AMA 非対応）
  https://learn.microsoft.com/azure/azure-monitor/agents/agents-overview#supported-services

## 問題63
あなたのネットワークには、次の図に示す 2 つの Active Directory Domain Services (AD DS) フォレストが含まれています。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/c5468ea9-0cbd-45a2-aa5c-d945f7646b27.png)

フォレストには、次の表に示すドメイン コントローラーが含まれています。
|Name|Domain|Global catalog|Schema master|
|---|---|---|---|
|DC1|adatum.com|Yes|Yes|
|DC2|adatum.com|No|No|
|DC3|west.adatum.com|Yes|No|
|DC4|contoso.com|Yes|Yes|

DC1 上で次の操作を実行します。
* User1 というユーザーを作成する。
* Attribute1 という新しい属性でスキーマを拡張する。

User1 と Attribute1 は、どのドメイン コントローラーにレプリケートされますか。
解答するには、解答領域で適切なオプションを選択してください。
注: 各正しい選択は 1 ポイントの価値があります。
User1:「DC2 only」or「DC3 only」or「DC2 and DC3 only」or「DC3 and DC4 only」or「DC2, DC3, and DC4」
Attribute:「DC2 only」or「DC4 only」or「DC2 and DC3 only」or「DC2, DC3, and DC4」

## 解答63
- User1: DC2 and DC3 only
- Attribute1: DC2 and DC3 only

## 解説63
- ユーザー オブジェクトは「ドメイン パーティション」に格納され、同一ドメイン内の DC にのみフル レプリケーションされます（DC1→DC2）。一方、グローバル カタログ (GC) はフォレスト内すべてのオブジェクトの部分レプリカ（Partial Attribute Set）を保持するため、同じフォレスト内の GC である DC3 にも User1 の部分レプリカがレプリケートされます。別フォレストの DC4 にはレプリケートされません。
  - 参考: 「グローバル カタログはフォレスト内のすべてのオブジェクトの部分的・読み取り専用レプリカを保持する」Understanding the Global Catalog: https://learn.microsoft.com/windows-server/identity/ad-ds/plan/understanding-the-global-catalog
  - 参考: 「ディレクトリ パーティション: ドメイン パーティションは同一ドメイン内、スキーマ/構成はフォレスト全体へレプリケート」Understanding Active Directory Domain Services（Directory partitions）: https://learn.microsoft.com/windows-server/identity/ad-ds/plan/understanding-active-directory-domain-services#directory-partitions
- スキーマ拡張（Attribute1）は「スキーマ パーティション」への変更であり、フォレスト内のすべての DC にレプリケートされます（DC1→DC2 と DC3）。別フォレストの DC4 にはレプリケートされません。
  - 参考: Active Directory Schema（スキーマはフォレスト単位で 1 つ、変更はフォレスト内のすべての DC にレプリケート）: https://learn.microsoft.com/windows-server/identity/ad-ds/plan/active-directory-schema

## 問題64
あなたのネットワークには、次の図に示すドメインが含まれています。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/73c376d8-aa7d-4d30-bd6f-5643de31a9b0.png)

あなたは、次の図に示す信頼関係を確立する必要があります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/a10ba82c-dc74-4dcc-948f-9a9437b292ab.png)

Trust1 と Trust2 に対して使用できる信頼の種類はどれですか。
回答するには、解答エリアで適切なオプションを選択してください。
注: それぞれの正しい選択は 1 点の価値があります。
解答エリア
Trust1:「External trust only」または「Shortcut trust only」または「Forest trust or external trust only」または「Forest trust, shortcut trust, or external trust」
Trust2:「Forest trust only」または「External trust or shortcut trust only」または「Forest trust or shortcut trust only」または「Forest trust or external trust only」または「Forest trust, shortcut trust, shortcut trust, or external trust」

## 解答64
Trust1: Shortcut trust only
Trust2: Forest trust or external trust only

## 解説64
- Trust1 は同一フォレスト内（contoso.com ツリー内）のドメイン間の信頼であるため、作成できるのはショートカット信頼（同一フォレスト内ドメイン間の認証経路短縮のためのトランジティブ信頼）のみです。フォレスト信頼と外部信頼はいずれも異なるフォレスト間で使用するため不適切です。
- Trust2 は contoso.com と fabrikam.com の異なるフォレスト間の信頼です。フォレスト間では、フォレスト ルート ドメイン間のトランジティブな「Forest trust」か、特定ドメイン間の非トランジティブな「External trust」のいずれかを使用できます。ショートカット信頼はフォレスト内専用のため該当しません。

根拠（Microsoft 公式ドキュメント）:
- Active Directory trusts の種類（ショートカット信頼は同一フォレスト内のドメイン間で使用）: https://learn.microsoft.com/windows-server/identity/ad-ds/manage/trusts/active-directory-trusts
- Forest trust（異なるフォレスト間、フォレスト ルート同士のトランジティブ信頼）: https://learn.microsoft.com/windows-server/identity/ad-ds/manage/trusts/forest-trust
- External trust（別フォレストまたは NT4 ドメインとの非トランジティブ信頼）: https://learn.microsoft.com/windows-server/identity/ad-ds/manage/trusts/external-trusts

## 問題65
オンプレミスの Active Directory Domain Services (AD DS) ドメイン contoso.com を含むネットワークがあります。
ドメインには、Windows Server を実行し Hyper‑V サーバー ロールがインストールされた 3 台のサーバーが含まれています。
各サーバーには Switch Embedded Teaming (SET) チームがあります。
フェールオーバー クラスターをサポートするために、各サーバーで Remote Direct Memory Access (RDMA) と必要な Windows Server の設定が正しく構成されていることを検証する必要があります。
何を使用すべきですか？
A. Server Manager
B. Get-NetAdapter コマンドレット
C. Failover Cluster Manager
D. Validate-DCB コマンドレット

## 解答65
D. Validate-DCB コマンドレット

## 解説65
- Validate-DCB は、SMB Direct (RDMA) を利用するために必要な Data Center Bridging (DCB)/QoS 構成および RDMA 設定が正しいかをホスト上で検証する専用コマンドレットです。RoCE/SMB Direct を用いる Hyper‑V/クラスター環境で推奨され、各サーバーの構成チェックに最適です。
- Server Manager (A) や Failover Cluster Manager の検証 (C) では RDMA/DCB の詳細な妥当性検査は行いません。Get‑NetAdapter (B) はアダプター情報の取得に留まり、RDMA/DCB の一貫性検証までは実施しません。
- Microsoft 公式ドキュメント（根拠）:
  - Validate-DCB（SMB Direct/RDMA のための DCB 構成検証）: https://learn.microsoft.com/powershell/module/datacenterbridging/validate-dcb
  - SMB Direct (RDMA) の概要（RDMA と DCB の要件）: https://learn.microsoft.com/windows-server/storage/file-server/smb-direct
  - RDMA と Switch Embedded Teaming (SET): https://learn.microsoft.com/windows-server/networking/technologies/hpn/rdma-and-switch-embedded-teaming

## 問題66
あなたのネットワークには Active Directory Domain Services (AD DS) フォレストがあります。
フォレストには 3 つのドメインがあり、各ドメインには 10 台のドメイン コントローラーが含まれています。
あなたは DNS ゾーンをカスタムの Active Directory パーティションに格納する計画です。
ゾーン用の Active Directory パーティションを作成する必要があり、そのパーティションは 4 台のドメイン コントローラーのみにレプリケートされる必要があります。
何を使用すべきですか？
A. dnscmd.exe
B. Active Directory Sites and Services
C. Active Directory Administrative Center
D. DNS Manager

## 解答66
A. dnscmd.exe

## 解説66
- カスタムのアプリケーション ディレクトリ パーティション（DNS 用）を作成し、レプリカに参加させるドメイン コントローラーを限定するには `dnscmd.exe` を使用します。`/CreateDirectoryPartition` でパーティションを作成し、`/EnlistDirectoryPartition` でレプリカにする DC を必要な 4 台だけ参加させます。
- DNS Manager は既存の「指定したアプリケーション ディレクトリ パーティション」へゾーンを保存する設定は可能ですが、新規にカスタム パーティションそのものを作成する機能はありません。AD 管理ツール（Sites and Services/Administrative Center）も DNS のアプリケーション パーティション作成には対応していません。
- Microsoft 公式ドキュメント（根拠）:
  - dnscmd（DNS サーバーの管理。Directory Partition の作成/参加コマンドを含む）: https://learn.microsoft.com/windows-server/administration/windows-commands/dnscmd
  - アプリケーション ディレクトリ パーティションの概要（任意の DC サブセットにレプリケート可能）: https://learn.microsoft.com/windows/win32/ad/application-directory-partitions
  - SMB: DNS ゾーンの AD 統合とレプリケーション スコープ（指定アプリケーション ディレクトリ パーティションの利用）: https://learn.microsoft.com/windows-server/administration/windows-commands/dnscmd#remarks

## 問題67
あなたの会社には本社と支社があり、2 つのオフィスは WAN リンクで接続されています。
各オフィスには、WAN トラフィックをフィルタリングするファイアウォールがあります。
支社のネットワークには、Windows Server を実行する 10 台のサーバーがあり、すべてのサーバーは本社からのみ管理されています。
支社のサーバーを Windows Admin Center ゲートウェイで管理する計画です。
支社内のあるサーバーに、既定の設定を使用して Windows Admin Center ゲートウェイをインストールしました。
Windows Admin Center ゲートウェイへの必要な受信接続を許可するために、支社のファイアウォールを構成する必要があります。
どの受信 TCP ポートを許可するべきですか？
A. 443
B. 6516
C. 5985
D. 3389

## 解答67
A. 443

## 解説67
- Windows Admin Center (WAC) を Windows Server に「既定の設定」でインストールした場合、ゲートウェイは既定で TCP 443 で待ち受けます。そのため、支社側ファイアウォールでは 443/TCP の受信を許可します。
- 参考: Windows 10 クライアントにインストールする場合の既定ポートは 6516 ですが、本設問は「サーバー」に既定でインストールしているため 443 です。WinRM (5985/5986) はゲートウェイから管理対象サーバーへのアウトバウンドで使用されます。RDP(3389)は WAC の待ち受けには不要です。
- Microsoft 公式ドキュメント（根拠）:
  - Install Windows Admin Center（Windows Server では既定ポート 443、Windows 10 では 6516）: https://learn.microsoft.com/windows-server/manage/windows-admin-center/deploy/install
  - Windows Admin Center のネットワーク要件/ポート: https://learn.microsoft.com/windows-server/manage/windows-admin-center/plan/ports

## 問題68
あなたのネットワークには contoso.com という名前の Active Directory Domain Services (AD DS) フォレストがあります。
ルート ドメインには次の表に示すドメイン コントローラーが含まれています。
|Name|FSMO role|
|---|---|
|DC1|Domain naming master|
|DC2|RID master|
|DC3|PDC emulator|
|DC4|Schema master|
|DC5|Infrastructure master|

どのドメイン コントローラーが障害になると、アプリケーション パーティションを作成できなくなりますか？
A. DC1  
B. DC2  
C. DC3  
D. DC4  
E. DC5

## 解答68
A. DC1

## 解説68
- アプリケーション ディレクトリ パーティション（アプリケーション パーティション）の作成・削除は、フォレスト全体で一意な名前付けと名前付けコンテキストの追加/削除を調整する「Domain naming master」FSMO によって制御されます。したがって、Domain naming master（DC1）が利用できないと新規のアプリケーション パーティションを作成できません。
- 参考（Microsoft 公式ドキュメント）:
  - Operations master roles（Domain naming master はドメインやアプリケーション パーティションの追加/削除を担当）: https://learn.microsoft.com/windows-server/identity/ad-ds/plan/operations-master-roles
  - Application directory partitions（作成/削除にはドメイン命名マスターが必要）: https://learn.microsoft.com/windows/win32/ad/application-directory-partitions

## 問題69
Azure File Sync を使用して、SRV1 上の C:\app の内容を Azure ファイル共有 sharel1 にレプリケートする必要があります。
必要なソース ファイルは \dc1.contoso.com\install という名前のフォルダーにあります。

## 解答69
1. Azure ポータルで Storage Sync Service を（sharel1 と同じリージョンに）作成する。
2. ストレージ アカウントで Azure ファイル共有 sharel1 を作成する。
3. SRV1 に Azure File Sync エージェントをインストールし、SRV1 を Storage Sync Service に登録する。
4. Storage Sync Service で Sync Group を作成し、Cloud Endpoint に sharel1 を追加する。
5. 同じ Sync Group に Server Endpoint を追加し、パスに C:\app を指定する。
6. \dc1.contoso.com\install から必要ファイルを C:\app に配置（コピー）する。以後、自動的に sharel1 へ同期される。

## 解説69
- Azure File Sync は「Sync Group」に Cloud Endpoint（Azure ファイル共有）と Server Endpoint（オンプレミスのパス）を関連付けて双方向同期します。本要件では、Cloud Endpoint に sharel1、Server Endpoint に C:\app を設定します。SRV1 側ではエージェントのインストールとサーバー登録が必須です。
- 既存のソースが \dc1.contoso.com\install にあるため、初回同期前に C:\app にコンテンツを配置しておくと、初回のクラウド シードとして sharel1 にレプリケートされます。
- 参考（Microsoft 公式ドキュメント）:
  - Azure File Sync のデプロイ手順（Storage Sync Service 作成、エージェント インストール/登録、Sync Group とエンドポイント作成）: https://learn.microsoft.com/azure/storage/file-sync/file-sync-deployment-guide
  - Azure File Sync エージェントのインストール: https://learn.microsoft.com/azure/storage/file-sync/file-sync-server-installation
  - Azure File Sync の計画としくみ（Cloud/Server Endpoint と同期動作）: https://learn.microsoft.com/azure/storage/file-sync/file-sync-planning

## 問題70
Windows Admin Center を SRV1 から使用して DC1 を管理できるようにする必要があります。
必要なソース ファイルは \\dc1.contoso.com\install という名前のフォルダーにあります。

## 解答70
1. SRV1 で \\dc1.contoso.com\install から WindowsAdminCenter.msi を取得し、SRV1 に既定設定でインストールする（ゲートウェイ/サービスとして）。必要に応じてポートは既定の 443 を使用し、証明書は生成または指定する。
2. SRV1 上で Windows Admin Center を開く（例: https://localhost/ または https://SRV1/）。
3. Windows Admin Center の [Add] → [Server] を選び、DC1（FQDN など）を追加して接続する。

## 解説70
- Windows Admin Center はゲートウェイを Windows Server にインストールし、ブラウザー経由で管理します。既定ではサーバー インストール時の待受ポートは 443 です。ターゲット サーバー（DC1）との管理通信には WinRM（5985/5986）を使用します。
- 本要件では、SRV1 に WAC をインストールし、接続先として DC1 を追加すれば管理可能になります。インストール用 MSI は指定の共有（\\dc1.contoso.com\install）から取得します。
- 参考（Microsoft 公式ドキュメント）:
  - Windows Admin Center のインストール（サーバー既定ポート 443）: https://learn.microsoft.com/windows-server/manage/windows-admin-center/deploy/install
  - Windows Admin Center のネットワーク要件/ポート: https://learn.microsoft.com/windows-server/manage/windows-admin-center/plan/ports
  - サーバーの管理（接続の追加手順）: https://learn.microsoft.com/windows-server/manage/windows-admin-center/use/manage-servers

## 問題71
Windows Server を実行する Azure 仮想マシン VM1 があります。
リモート デスクトップ接続を確立する前に、管理者が VM1 へのアクセスを要求するようにする必要があります。
何を構成すべきですか？
A. ネットワーク セキュリティ グループ (NSG)
B. Azure Front Door
C. Microsoft Defender for Cloud
D. Azure AD Privileged Identity Management (PIM)

## 解答71
C. Microsoft Defender for Cloud

## 解説71
- Microsoft Defender for Cloud の「Just-in-time VM access (JIT)」を有効化すると、RDP (TCP 3389) などの管理ポートを既定で閉じ、接続前に管理者がアクセスを“要求”するフローになります。要求が承認されると、許可した送信元と時間に限定して NSG が自動的に一時開放されます。これにより、接続前に要求が必須となります。
- NSG 単体 (A) では承認付きの要求ワークフローは提供されません。Front Door (B) は Web トラフィック向けで RDP には不適。PIM (D) はロールの JIT 付与であり、RDP ポート開放の要求フローとは別です。
- 公式ドキュメント（根拠）:
  - Just-in-time VM access の概要: https://learn.microsoft.com/azure/defender-for-cloud/just-in-time-access-overview
  - JIT の使い方（要求・承認と NSG 自動構成）: https://learn.microsoft.com/azure/defender-for-cloud/just-in-time-access-usage

## 問題72
オンプレミスの Active Directory Domain Services (AD DS) ドメインがあり、Azure Active Directory (Azure AD) テナントと同期しています。
オンプレミス ネットワークは Site-to-Site VPN を使用して Azure に接続されています。次の表に示す DNS ゾーンがあります。

|Name|Location|Description|
|---|---|---|
|contoso.com|オンプレミス ネットワーク上のドメイン コントローラー DC1|オンプレミスでの名前解決を提供する|
|fabrikam.com|Azure のプライベート DNS ゾーン|すべての Azure 仮想ネットワークでの名前解決を提供する|

オンプレミス ネットワークから fabrikam.com の名前を解決できるようにする必要があります。どの 2 つのアクションを実行する必要がありますか？正しい回答は解の一部を構成します。注意: 各正解は 1 ポイントの価値があります。

A. DC1 上に fabrikam.com の条件付きフォワーダーを作成する。
B. DC1 上に fabrikam.com のスタブ ゾーンを作成する。
C. DC1 上に fabrikam.com のセカンダリ ゾーンを作成する。
D. Windows Server を実行する Azure 仮想マシンをデプロイする。仮想ネットワークの DNS サーバー設定を変更する。
E. Windows Server を実行する Azure 仮想マシンをデプロイする。その仮想マシンを DNS フォワーダーとして構成する。

## 解答72
A、E

## 解説72
- オンプレミスから Azure Private DNS の名前を解決するには、Azure 内に DNS プロキシ/フォワーダーを用意し（E）、オンプレミス DNS（DC1）から該当ゾーン宛ての問い合わせをそのプロキシへ条件付きフォワードする（A）のが定石です。
- Azure の既定 DNS リゾルバー（168.63.129.16）は Azure 仮想ネットワーク内部からのみ到達可能で、オンプレミスから直接問い合わせることはできません。そのため Azure 側に DNS フォワーダー（例: Windows Server DNS）を置き、同サーバーのフォワーダーを 168.63.129.16 に向けます。
- B/C が不適切な理由: Azure Private DNS はゾーン転送（AXFR/IXFR）をサポートしないため、オンプレミスにスタブ/セカンダリ ゾーンを作成して同期する方式は取れません。
- D は仮想ネットワーク内の VM が使用する DNS を変える設定であり、オンプレミスからの名前解決可否には直接関与しません。

参考（Microsoft 公式ドキュメント）
- Azure Private DNS の概要（Azure 提供 DNS 168.63.129.16 による解決、プライベート ゾーンの基本）: https://learn.microsoft.com/azure/dns/private-dns-overview
- Azure の名前解決（オンプレミスからは Azure 提供 DNS に直接到達できないため、Azure 内で DNS フォワーダー/プロキシを使う設計）: https://learn.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances
- 仮想ネットワークの DNS サーバー設定（VNet 内 VM の解決に影響し、オンプレミスには影響しない）: https://learn.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances#dns-servers

## 問題73
あなたのネットワークには、contoso.com という名前の Active Directory Domain Services (AD DS) ドメインが存在します。

ドメインの PDC エミュレーターがどのサーバーかを特定する必要があります。

解決策: Active Directory ドメインと信頼関係 から、コンソール ツリーの Active Directory ドメインと信頼関係 を右クリックし、[操作マスター] を選択します。

これは目標を満たしていますか？
A. いいえ
B. はい

## 解答73
A. いいえ

## 解説73
提示の手順 (Active Directory ドメインと信頼関係 の [操作マスター]) で確認できるのは、フォレストの「ドメイン名前付けマスター」FSMO のみです。PDC エミュレーターはドメイン単位の FSMO であり、GUI で特定・管理するには Active Directory ユーザーとコンピューター の [操作マスター] (PDC タブ) を使用します。したがって、提示の解決策は目標を満たしません。

根拠 (Microsoft 公式ドキュメント):
- 「Active Directory ユーザーとコンピューター を使用して、RID マスター、PDC エミュレーター、インフラストラクチャ マスターの役割を転送します。」および「Active Directory ドメインと信頼関係 を使用して、ドメイン名前付けマスターの役割を転送します。」と明記されています。
  https://learn.microsoft.com/troubleshoot/windows-server/active-directory/transfer-or-seize-fsmo-roles-in-windows-server
- FSMO 役割の概要 (PDC エミュレーターはドメイン単位の役割):
  https://learn.microsoft.com/windows-server/identity/ad-ds/plan/operations-master-roles

## 問題74
あなたのネットワークには Active Directory Domain Services (AD DS) フォレストがあります。
フォレストには、Site1、Site2、Site3 という名前の 3 つの Active Directory サイトが含まれます。
各サイトには 2 台のドメイン コントローラーがあります。
サイトは DEFAULTIPSITELINK を使用して接続されています。

新しい支社を開設しましたが、そこにはクライアント コンピューターのみが存在します。

新しいオフィス内のクライアント コンピューターが主に Site1 のドメイン コントローラーによって認証されるようにする必要があります。

解決策: Site1 にリンクされた GPO で「Try Next Closest Site」グループ ポリシー設定を構成します。

この方法は目標を満たしますか?
A. Yes
B. No

## 解答74
B

## 解説74
- 「Try Next Closest Site」はクライアント側の DC Locator 挙動を制御するポリシーであり、クライアントの所属サイトにドメイン コントローラーが存在しない場合に、サイト リンク コストに基づく“次に近いサイト”の DC を探すようにします。この設定はクライアントに適用される必要があります（Site1 ではなく、新支社のクライアントが適用対象でなければ効果がありません）。
- 問題の解決策は GPO を Site1 にリンクしているため、Site1 のコンピューターにしか適用されません。新オフィスのクライアントには適用されず、期待する認証先の制御にならないため要件を満たしません。
- さらに、特定サイト（Site1）を優先させたい場合は、サイト リンクのコスト設計により Site1 が“次に近いサイト”として評価される必要があります。既定の DEFAULTIPSITELINK（同一コスト）だけでは Site1 を優先できる保証はありません。

根拠（Microsoft 公式ドキュメント）
- DC Locator の「次に近いサイト」機能（Try Next Closest Site）の概要と要件（クライアント側の設定であり、ローカル サイトに DC がいない場合に適用）: https://learn.microsoft.com/troubleshoot/windows-server/identity/next-closest-site-improvements-dc-locator
- サイト/サブネット/サイト リンクとコストによるトポロジ（クライアントがサブネット割り当てによりサイトに関連付けられ、コストで“近さ”が決まる）: https://learn.microsoft.com/windows-server/identity/ad-ds/plan/understanding-sites-subnets-and-site-links

## 問題75
Windows Server を実行する Server1 という名前のサーバーがあり、C、D、E という 3 つのボリュームを含みます。
Server1 には次の表のとおりにファイルが保存されています。
|Name|On volume|Size|
|---|---|---|
|File1|C|500 KB|
|File2|D|10 KB|
|File3|D|1 MB|

ボリューム D では、Data Deduplication (重複除去) が有効化され、用途 (Usage) は General purpose file server に設定されています。

次の操作を実行します:
* File1 をボリューム D に移動します。
* File2 をボリューム D にコピーし、そのコピーに File4 と名前を付けます。
* File3 をボリューム E に移動します。

次の各記述について、正しければ Yes を、正しくなければ No を選択してください。
注意: 各正解は 1 ポイントの価値があります。
|Statements|Yes|No|
|---|---|---|
|File1 is deduplicated after the deduplication job runs.| | |
|File3 is deduplicated after the deduplication job runs.| | |
|File4 is deduplicated after the deduplication job runs.| | |

## 解答75
|Statements|Answer|
|---|---|
|File1 is deduplicated after the deduplication job runs.|No|
|File3 is deduplicated after the deduplication job runs.|No|
|File4 is deduplicated after the deduplication job runs.|No|

## 解説75
- Data Deduplication はボリューム単位の機能です。重複除去は有効化したボリューム内のファイルのみに適用されます。E ボリュームでは重複除去を有効化していないため、E に移動した File3 は重複除去されません（No）。
- General purpose file server プロファイルの既定では、最小ファイル年齢 (MinimumFileAgeDays) は 3 日で、最小ファイルサイズは 32 KB です。直前に移動/コピーしたファイルは通常この年齢条件を満たさないため、D に移動した File1 は最適化ジョブ直後には重複除去されません（No）。
- File4 は 10 KB と 32 KB 未満のため、サイズ条件を満たさず重複除去の対象外です（No）。

根拠（Microsoft 公式ドキュメント）:
- Data Deduplication の概要と要件（ボリューム単位で有効化、適用対象、最小ファイルサイズ 32 KB）
  https://learn.microsoft.com/windows-server/storage/data-deduplication/overview
- 一般用途 (General purpose file server) の既定設定（MinimumFileAgeDays = 3 日 など）
  https://learn.microsoft.com/windows-server/storage/data-deduplication/plan/plan-deduplication

## 問題76
ユーザー設定のみを含むグループ ポリシー オブジェクト (GPO) の GPO1 を所有しています。
GPO1 を Group1 というグローバル セキュリティ グループに適用する予定です。
GPO1 をドメインにリンクし、Authenticated Users グループに付与されていたすべての権限を削除しました。
次の要件を満たすように GPO1 の権限を構成する必要があります。
* GPO1 は Group1 のユーザーのみに適用されること。
* 解決策は最小特権の原則に従っていること。

Answer Area
Group1: Apply group policy and Read または Apply group policy only または Read only
Domain Computers: Apply group policy and Read または Apply group policy only または Read only

## 解答76
| 対象 | 設定する権限 |
|---|---|
| Group1 | Apply group policy and Read |
| Domain Computers | Read only |

## 解説76
- GPO を適用するには、対象のセキュリティ プリンシパル（ユーザーまたはコンピューター）に「Read」と「Apply group policy」の両方が必要です。したがって、GPO1 を適用したい Group1 には両方の権限を付与します。
- 本 GPO はユーザー設定のみであり、Authenticated Users を完全に削除したため、クライアントが GPO を検出・読取できるように、コンピューター アカウント（Domain Computers）には読み取り (Read) のみを付与します。Domain Computers に「Apply group policy」は不要で、最小権限の原則に反するため付与しません。

根拠（Microsoft 公式ドキュメント）
- Manage Group Policy using GPMC > Security filtering: 「ユーザーまたはコンピューターに GPO を適用するには、その GPO に対する Read と Apply group policy の両方のアクセス許可が必要」
  https://learn.microsoft.com/windows-server/identity/grouppolicy/manage-group-policy-using-gpmc#security-filtering
- Group Policy の適用の仕組み（概要）: セキュリティ フィルタリングによる対象の限定と、必要なアクセス許可についての説明
  https://learn.microsoft.com/windows-server/identity/grouppolicy/how-group-policy-works#group-policy-application

## 問題77
Windows Server 2022 のコンテナー ホストである Host1 と、次の表に示すコンテナー イメージを含むコンテナー レジストリがあります。

|Name|Container base image OS version|
|---|---|
|image1|Windows Server 2022|
|image2|Windows Server 2019|

Host1 上でコンテナーを実行する必要があります。
各イメージで使用できる分離モードはどれですか？回答するには、解答領域で適切なオプションを選択してください。
注意: 各正解は 1 ポイントの価値があります。

解答領域
Image1:「Hyper-V isolation only」or「Process isolation only」or「Hyper-V isolation or process isolation」
image2:「Hyper-V isolation only」or「Process isolation only」or「Hyper-V isolation or process isolation」

## 解答77
|項目|選択肢|
|---|---|
|Image1|Hyper-V isolation or process isolation|
|Image2|Hyper-V isolation only|

## 解説77
- Windows の process 分離は、ホストとコンテナーの OS バージョン（カーネル/ビルド）が一致している必要があります。したがって、Windows Server 2022 ホストでは、Windows Server 2022 ベース イメージは process 分離でも Hyper-V 分離でも実行できます。
- Windows Server 2019 ベース イメージは、Windows Server 2022 ホストと OS バージョンが一致しないため、process 分離は不可で、Hyper-V 分離のみがサポートされます（コンテナーを軽量 VM として実行し、イメージに合致するカーネルを提供）。

根拠（Microsoft 公式ドキュメント）
- Windows コンテナーのバージョン互換性（ホストとコンテナーの OS バージョン整合性、Hyper-V 分離で異なるバージョンを許容）: https://learn.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility
- Hyper-V 分離の概要（軽量 VM によりカーネルを分離し、互換性を確保）: https://learn.microsoft.com/virtualization/windowscontainers/manage-containers/hyperv-container
- Windows コンテナーの基本（分離モードの概要）: https://learn.microsoft.com/virtualization/windowscontainers/about/

## 問題78
単一ドメインの Active Directory Domain Services (AD DS) フォレスト contoso.com を展開しました。
ドメインにサーバーを 1 台展開し、そのサーバーでサービスを実行するように構成しました。
サービスがグループ マネージド サービス アカウント (gMSA) を使用して認証できるようにする必要があります。
次の 3 つの PowerShell コマンドレットを、正しい順序で実行する必要があります。回答するには、次のコマンドレットの一覧から適切なものを選び、正しい順序に並べ替えてください。
選択肢：
・Add-ADComputerServiceAccount
・Set-KdsConfiguration
・Add-ADGroupMember
・Add-KdsRootKey
・New-ADServiceAccount
・Install-ADServiceAccount

## 解答78
1. Add-KdsRootKey
2. New-ADServiceAccount
3. Install-ADServiceAccount

## 解説78
- gMSA を使用するには、まず KDS ルート キーを作成する必要があります (Add-KdsRootKey)。
- 次に、gMSA を作成します (New-ADServiceAccount)。この際、パスワード取得を許可するコンピューター/グループを指定します。
- 最後に、サービスを実行するサーバー上で gMSA をインストールします (Install-ADServiceAccount)。
- Add-ADComputerServiceAccount は主に単体 MSA (sMSA) で使用し、gMSA では必須ではありません。Set-KdsConfiguration や Add-ADGroupMember は本設問の最小手順には不要です。

根拠（Microsoft 公式ドキュメント）:
- KDS ルート キーの作成 (Add-KdsRootKey): https://learn.microsoft.com/powershell/module/activedirectory/add-kdsrootkey
- gMSA の概要: https://learn.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview
- gMSA の作成 (New-ADServiceAccount): https://learn.microsoft.com/powershell/module/activedirectory/new-adserviceaccount
- gMSA のインストール (Install-ADServiceAccount): https://learn.microsoft.com/powershell/module/activedirectory/install-adserviceaccount

## 問題79
SRV1 を Azure ファイル共有と同期できるように登録する必要があります。
登録には「34646045 Storage Sync Service」を使用しなければなりません。
必要なソース ファイルは \\dc1.contoso.com\install という名前のフォルダーにあります。
現時点ではファイル共有の同期を構成する必要はなく、エージェントを更新する必要もありません。

## 解答79
1. （SRV1 上で）Azure File Sync エージェントをインストールする。
   - \\dc1.contoso.com\install にある Azure File Sync エージェント（例: StorageSyncAgent.msi）を実行してインストールします（例: `msiexec /i \\dc1.contoso.com\install\StorageSyncAgent.msi /passive`）。
   - インストール後に更新を求められても、今回は「更新しない」を選択します（本タスクではエージェント更新は不要）。

2. PowerShell（管理者）で Az.StorageSync モジュールを準備する。
   - 必要に応じてインストール: `Install-Module -Name Az.StorageSync -Force`
   - 読み込み: `Import-Module -Name Az.StorageSync`

3. Azure にサインインし、対象サブスクリプションを選択する。
   - `Connect-AzAccount`
   - `Select-AzSubscription -SubscriptionId <サブスクリプションID>`

4. 「34646045 Storage Sync Service」へ SRV1 を登録する。
   - `Register-AzStorageSyncServer -ResourceGroupName <リソースグループ名> -StorageSyncServiceName 34646045`

5. 登録の確認（任意）。
   - `Get-AzStorageSyncServer -ResourceGroupName <リソースグループ名> -StorageSyncServiceName 34646045`

（注）本タスクではサーバー エンドポイント／クラウド エンドポイントの作成などファイル共有同期の構成は行いません。

## 解説79
- Azure File Sync は、Windows Server を Storage Sync Service に「サーバー登録」して信頼関係を確立した後に、サーバー エンドポイント／クラウド エンドポイントを作成して同期を構成します。本設問では登録のみが要件であるため、エージェントをインストールしてから Register-AzStorageSyncServer を用いて対象の Storage Sync Service（34646045）に登録すれば要件を満たします。
- Windows Server へのエージェント インストールとサーバー登録は Azure File Sync の標準手順であり、同期設定は登録後に別途実施します。エージェント更新は本タスクでは不要です。

根拠（Microsoft 公式ドキュメント）
- Azure File Sync のデプロイ手順（エージェントのインストールとサーバー登録）: https://learn.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide
- Register-AzStorageSyncServer（サーバーを Storage Sync Service に登録）: https://learn.microsoft.com/powershell/module/az.storagesync/register-azstoragesyncserver

## 問題80
Windows Admin Center がインストールされた Server1 という名前のサーバーがあります。
Windows Admin Center で使用している証明書は認証局 (CA) から取得したものです。
その証明書の有効期限が切れました。
証明書を置き換える必要があります。
次の操作から、実行する 3 つの操作を正しい順序で選択しなさい。
回答するには、操作の一覧から適切なものを解答欄に移動し、正しい順序に並べ替えてください。
選択肢：
- Copy the certificate thumbprint.
- From Internet Information Services (IIS) Manager, bind a certificate.
- Rerun Windows Admin Center Setup and select Change.
- Rerun Windows Admin Center Setup and select Repair.
- Rerun Windows Admin Center Setup and select Remove.

## 解答81
1. Copy the certificate thumbprint.
2. Rerun Windows Admin Center Setup and select Change.
3. Rerun Windows Admin Center Setup and select Repair.

## 解説81
- Windows Admin Center (WAC) は既定で IIS を使用せず、HTTP.sys 上のゲートウェイ サービスとして動作します。そのため証明書のバインドは IIS マネージャーではなく、WAC のセットアップで再構成します（IIS でのバインドは不要）。
- 新しい証明書をサーバーのローカル コンピューター個人ストアにインポートした後、その拇印 (thumbprint) を控えます（Copy the certificate thumbprint）。
- 証明書またはポートを変更するには、WAC のインストーラーを再実行して［Change］を選択し、新しい証明書を指定します（Rerun Windows Admin Center Setup and select Change）。
- 環境によってはコマンドライン/サイレント適用などで修復モード（Repair）を用いて新しい拇印を反映させる手順が案内されています。セットアップを［Repair］で再実行すると、設定の再適用・サービス再登録が行われ、証明書の更新が確実に反映されます（Rerun Windows Admin Center Setup and select Repair）。

根拠（Microsoft 公式ドキュメント）:
- 「Windows Admin Center のポートや SSL 証明書を変更するには、インストーラーを再実行して Change を選択する」
  https://learn.microsoft.com/windows-server/manage/windows-admin-center/deploy/install#change-the-port-or-ssl-certificate
- 「独自の SSL 証明書の使用（証明書をコンピューター ストアに配置し、拇印を指定してインストール/再構成可能）」
  https://learn.microsoft.com/windows-server/manage/windows-admin-center/configure/use-ssl-certificate
- WAC は IIS を使用しない（HTTP.sys ベースのゲートウェイ サービスである）旨の説明と、インストーラー/コマンドラインでの再構成方法の詳細:
  https://learn.microsoft.com/windows-server/manage/windows-admin-center/understand/faq

## 問題82
あなたは Azure サブスクリプションを所有しています。そのサブスクリプションには、Windows Server を実行する VM1 という名前の仮想マシンが含まれています。
あなたは App1 というアプリを作成しました。
App1 を VM1 へ継続的インテグレーションおよび継続的デプロイ (CI/CD) できるように構成する必要があります。
最初に作成すべきものはどれですか？
A. マネージド ID
B. Azure DevOps 組織
C. App Service Environment
D. Azure Automation アカウント

## 解答82
B. an Azure DevOps organization

## 解説82
- Azure Pipelines による CI/CD を始めるには、まず Azure DevOps 組織が必要です。これによりプロジェクトやリポジトリ、パイプライン（ビルド・リリース）を作成でき、VM へのデプロイ環境（Environments）設定も行えます。
- マネージド ID、App Service Environment、Azure Automation アカウントはいずれも VM へのアプリの CI/CD 構成を開始するための前提ではありません。

根拠（Microsoft 公式ドキュメント）
- Create an organization: 「Azure DevOps Services を使い始めるには、組織を作成します。」https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization?view=azure-devops
- Create your first pipeline（前提条件）: Azure DevOps 組織とプロジェクトが必要。https://learn.microsoft.com/azure/devops/pipelines/create-first-pipeline?view=azure-devops&tabs=browser
- Deploy to Windows VMs with Azure Pipelines: Azure DevOps のパイプラインと Environments を用いて仮想マシンへデプロイする手順。https://learn.microsoft.com/azure/devops/pipelines/deploy/virtual-machines-windows?view=azure-devops

## 問題83
オンプレミスの Windows Server を実行する Server 1 という名前のサーバーがあります。
Server 1 には Share 1 という名前のファイル共有が含まれています。
Azure サブスクリプションがあります。
次の操作を実行しました:
* Azure File Sync をデプロイする
* Server1 に Azure File Sync エージェントをインストールする
* Server1 を Azure File Sync に登録する

Share1 を Azure File Sync のサーバー エンドポイントとして追加できるようにする必要があります。
どの 3 つのアクションを順番に実行する必要がありますか？回答するには、アクションの一覧から適切なアクションを解答領域に移動し、正しい順序に並べてください。

選択肢：
- Deploy the Azure Connected Machine agent.
- Deploy an Azure VPN gateway.
- Create a private endpoint.
- Create an Azure Storage account.
- Create an Azure file share.
- Create a sync group.

## 解答83
|順序|アクション|
|---|---|
|1|Create an Azure Storage account.|
|2|Create an Azure file share.|
|3|Create a sync group.|

## 解説83
- Azure File Sync でサーバー エンドポイント（オンプレミスのパス）を追加するには、その前提として「クラウド エンドポイント（Azure ファイル共有）」を含む同期グループが必要です。クラウド エンドポイントは特定の Azure Storage アカウント内の Azure ファイル共有を指すため、まずストレージ アカウントを作成し、その中に Azure ファイル共有を作成してから、同期グループを作成します。
- 既に Server1 は登録済みであり、ネットワークは既定でアウトバウンド 443/TLS を使用してサービスへ接続するため、VPN ゲートウェイやプライベート エンドポイント、Azure Connected Machine（Azure Arc）エージェントは本要件（Share1 をサーバー エンドポイントとして追加可能にする）には不要です。

根拠（Microsoft 公式ドキュメント）
- デプロイ ガイド（ストレージ アカウントとファイル共有の作成 → 同期グループとクラウド エンドポイント作成 → サーバー エンドポイント追加）: https://learn.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide
- クラウド エンドポイントは Azure ファイル共有を指す（同期グループの構成要素）: https://learn.microsoft.com/azure/storage/files/storage-sync-files-planning#cloud-endpoints
- ネットワーク要件（既定はインターネット経由の 443、VPN/プライベート エンドポイントは任意）: https://learn.microsoft.com/azure/storage/files/storage-sync-files-firewall-and-proxy

## 問題84
Active Directory Domain Services (AD DS) ドメインがあります。
ドメインには、Windows Server を実行する Server1、Server2、Server3 という名前の 3 台のサーバーが含まれます。
あなたはドメイン アカウントで Server1 にサインインし、Server2 へのリモート PowerShell セッションを開始します。
リモート PowerShell セッションから Server3 上のリソースにアクセスしようとしますが、リソースへのアクセスは拒否されます。
あなたの資格情報が Server1 から Server3 に渡されるようにする必要があります。
ソリューションは管理上の労力を最小限に抑える必要があります。
何をすべきですか？
A. ドメインの「ユーザーのログオン制限を強制する」ポリシー設定を無効にする。
B. ドメインの選択的な認証を構成する。
C. Just Enough Administration (JEA) を構成する。
D. Kerberos 制約付き委任 (Kerberos constrained delegation) を構成する。

## 解答84
D. Configure Kerberos constrained delegation.

## 解説84
- 事象は PowerShell リモーティングの「二重ホップ」問題で、既定では中継サーバー (Server2) がクライアント (Server1) のユーザー資格情報を背後のサーバー (Server3) に委任できないため発生します。Kerberos の制約付き委任 (KCD) を構成することで、Server2 が特定サービス (例: HOST/、CIFS/ など) に対してのみユーザーになり代わって Server3 へ委任できるようになり、資格情報が Server1 → Server3 に安全に渡ります。
- JEA (C) は管理権限を最小化する枠組みであり、資格情報の二重ホップ問題の解決策ではありません。選択的認証 (B) はフォレスト間/信頼シナリオ向けで今回の要件には不適合です。「ユーザーのログオン制限を強制する」(A) は Kerberos の発行時検査に関する設定で、二重ホップの解消には関係しません。

根拠（Microsoft 公式ドキュメント）:
- about_Remote_Troubleshooting: 二重ホップ問題とその対策（CredSSP または Kerberos の委任で解決可能）
  https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_remote_troubleshooting#second-hop
- Kerberos 制約付き委任の概要（KCD の仕組みと構成方法）
  https://learn.microsoft.com/windows-server/security/kerberos/kerberos-constrained-delegation-overview

## 問題85
Windows Server を実行するオンプレミス サーバーが次の表のとおりあります。
|名前|種類|
|---|---|
|Server1|物理サーバー|
|VM2|Hyper-V 仮想マシン|

VM1 という名前の仮想マシンを含む Azure サブスクリプションを所有しています。
Azure Arc を使用してすべてのサーバーを管理できるようにする必要があります。
ソリューションは管理上の労力を最小限に抑えなければなりません。
Azure Connected Machine エージェントはどのサーバーにインストールすべきですか？
A. Server1、VM1、および VM2
B. Server1 のみ
C. VM2 のみ
D. VM1 と VM2 のみ
E. VM1 のみ
F. Server1 と VM2 のみ

## 解答85
F. Server1 と VM2 のみ

## 解説85
- Azure Arc 対応サーバー（Arc-enabled servers）は「Azure 外部（オンプレミスや他クラウド）」の Windows/Linux マシンを Azure リソースとして投影・管理するための機能です。したがって、対象は非 Azure マシンであり、Azure VM に対しては使用しません。
- Azure Connected Machine エージェントは、Azure ではないマシンにのみインストールします。Azure VM（選択肢の VM1）のように既に Azure 上にあるマシンにエージェントを入れることは不要かつ非推奨です。最小特権・最小労力の観点からも、オンプレミスの Server1 と VM2 にのみインストールします。

根拠（Microsoft 公式ドキュメント）
- What is Azure Arc-enabled servers?: Azure 外のマシン（オンプレミスや他クラウド）を管理対象とする旨が明記。
  https://learn.microsoft.com/azure/azure-arc/servers/overview
- FAQ: Azure VM を Azure Arc 対応サーバーに接続できるか（できない/不要である旨）。
  https://learn.microsoft.com/azure/azure-arc/servers/faq

## 問題86
Windows Server 2022 を実行し、DHCP Server ロールを持つ Server1 という名前のサーバーがあります。
Server1 には Scope1 という名前の単一の DHCP スコープが含まれています。

ネットワークに 5 台のプリンターを展開しました。

プリンターに常に同じ IP アドレスが割り当てられるようにする必要があります。

解決策: 各プリンターに対して DHCP アドレスの除外を作成します。

この要件を満たしますか?
A. No
B. Yes

## 解答86
A

## 解説86
- DHCP の「アドレス除外」は、指定したアドレス範囲をスコープから貸与不可にするだけで、特定クライアント（プリンター）へ固定的に割り当てる仕組みではありません。そのため、除外だけでは「プリンターが常に同じ IP を取得する」ことは保証されません。
- 目的を満たす正しい方法は「DHCP 予約（Reservation）」です。予約はクライアントの MAC アドレスと IP アドレスを関連付け、当該クライアントに常に同一 IP を割り当てます。
- 公式ドキュメントのとおり、除外は貸与対象から外す機能であり、予約は特定クライアントにアドレスを確保・割り当てるための機能です。

根拠（Microsoft 公式ドキュメント）
- Add-DhcpServerv4Reservation（特定クライアントのための IPv4 予約を追加）: https://learn.microsoft.com/powershell/module/dhcpserver/add-dhcpserverv4reservation
- Add-DhcpServerv4ExclusionRange（スコープの除外範囲を追加。除外されたアドレスはクライアントへリースされない）: https://learn.microsoft.com/powershell/module/dhcpserver/add-dhcpserverv4exclusionrange

## 問題87
Windows Server を実行するオンプレミス サーバーが 2 台あり、名前は Server1 と Servet2 です。
storage1 という名前の Azure Storage アカウントがあり、その中に share1 という名前のファイル共有が含まれています。
Server1 は Azure File Sync を使用して share1 と同期しています。Server2 が share1 と同期できるように構成する必要があります。
どの 3 つのアクションを順番に実行すべきですか。回答するには、アクションの一覧から該当するものを選び、正しい順序で並べてください。

選択肢:
- Server2 を Storage Sync Service に登録する。
- Azure サブスクリプションに Storage Sync Service を追加する。
- Server2 に Azure File Sync エージェントをインストールする。
- 同期グループにクラウド エンドポイントを追加する。
- 同期グループにサーバー エンドポイントを追加する。

## 解答87
|順序|アクション|
|---|---|
|1|Server2 に Azure File Sync エージェントをインストールする。|
|2|Server2 を Storage Sync Service に登録する。|
|3|同期グループにサーバー エンドポイントを追加する。|

## 解説87
- 既に Server1 が share1 と同期しているため、Storage Sync Service と同期グループおよびクラウド エンドポイント（share1 向け）は作成済みである前提です。追加サーバー（Server2）を同期に参加させるには、最小手順として (1) Server2 にエージェントをインストールし、(2) そのサーバーを Storage Sync Service に登録し、(3) 既存の同期グループにサーバー エンドポイントを追加すればよいです。

根拠（Microsoft 公式ドキュメント）
- Deploy Azure File Sync: 手順として「クラウド エンドポイント作成 → Windows Server にエージェントをインストール → サーバー登録 → サーバー エンドポイント追加」の順序が示されている。
  https://learn.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide
- Server endpoints in Azure File Sync: 同期グループに参加させるためにサーバー エンドポイントを追加する必要がある旨。
  https://learn.microsoft.com/azure/storage/files/storage-sync-files-server-endpoints

## 問題88
オンプレミスのネットワークには contoso.com という名前の Active Directory ドメインと、Windows Server を実行する 500 台のサーバーがあります。
すべてのサーバーは Azure Arc に対応しており、contoso.com に参加しています。
すべてのサーバーに PowerShell Desired State Configuration (DSC) を実装する必要があります。
ソリューションは管理上の労力を最小限に抑えなければなりません。
DSC スクリプトはどこに保存し、何を使用してサーバーに DSC を適用しますか。
回答するには、解答エリアで適切なオプションを選択してください。
注意: 各正解は 1 ポイントの価値があります。
Answer Area
Store in:「An Azure App Configuration store」or「An Azure Automation account」or「An Azure policy definition」
Use:「A Group Policy Object (GPO) in Active Directory Domain Services (AD DS)」or「Azure virtual machine extentions」or「Guest configuration in Azure policy」

## 解答88
- Store in: An Azure policy definition
- Use: Guest configuration in Azure policy

## 解説88
- Azure Arc 対応サーバーに対しては、Azure Policy の Guest Configuration を用いることで、スケールと一貫性をもって DSC による監査/構成適用を行えます。最小の管理労力で多数サーバーに一括適用でき、Arc 対応サーバーは自動的に Guest Configuration 拡張機能がデプロイされます。
- カスタムの Guest Configuration では、DSC コンテンツをパッケージ化し、その参照（contentUri・contentHash など）を含む「Azure Policy の定義」によって配布・適用します。そのため「保存場所」は Azure Policy 定義で表現するのが正解です。
- GPO はドメイン管理のオーバーヘッドが大きく、Azure 側の準拠管理とも分断されます。Azure VM Extensions は Azure 仮想マシン向けであり、Arc サーバーの推奨経路は Guest Configuration ポリシーです。

根拠（Microsoft 公式ドキュメント）:
- Azure Policy Guest Configuration の概念（Arc 対応サーバーでのサポート、ポリシーでの配布/適用）
  https://learn.microsoft.com/azure/governance/policy/concepts/guest-configuration
- カスタム Guest Configuration ポリシーの作成（パッケージの contentUri/contentHash をポリシー定義に含める）
  https://learn.microsoft.com/azure/governance/policy/concepts/guest-configuration#custom-content
- Azure Arc 対応サーバーでの Guest Configuration の管理（拡張機能の自動デプロイとポリシー適用）
  https://learn.microsoft.com/azure/azure-arc/servers/guest-configuration

## 問題89
あなたのネットワークには contoso.com という名前の Active Directory Domain Services (AD DS) ドメインがあります。
ドメインには次の表に示すサーバーが含まれています。

|Name|Role|
|---|---|
|Server1|DFS Namespaces|
|Server2|DFS Replication|
|Server3|DFS Namespaces, DFS Replication|
|Server4|None|

次の要件を満たす分散ファイル システム (DFS) ネームスペースを作成する必要があります。
- ドメイン ベースのネームスペース \contoso.com\Public
- Finance という名前のフォルダー

Finance フォルダーのフォルダー ターゲットとして構成できるサーバーはどれですか？
A. Server2 と Server3 のみ
B. Server3 のみ
C. Server1、Server2、Server3、Server4
D. Server1 と Server3 のみ
E. Server1、Server2、Server3 のみ

## 解答89
C

## 解説89
- DFS Namespaces の「フォルダー ターゲット」は、共有フォルダー（または別のネームスペース）への UNC パスであり、SMB 共有を提供できるサーバー上であれば構成可能です。フォルダー ターゲット側に DFS Namespaces の役割サービスは必須ではありません。
- 複数のターゲット間でデータを自動的に同期したい場合にのみ、DFS Replication が関与します（その場合、レプリケーション対象サーバーに DFS Replication の役割サービスが必要）。本問は「フォルダー ターゲットとして構成できるか」の可否のみを問うため、共有をホスト可能な全てのサーバー（Server1〜Server4）が該当します。

根拠（Microsoft 公式ドキュメント）
- DFS Namespaces の概要（フォルダーとフォルダー ターゲットの概念）: https://learn.microsoft.com/windows-server/storage/dfs-namespaces/dfs-overview
- DFS フォルダーとターゲットの作成（フォルダー ターゲットは共有フォルダーの UNC パス）: https://learn.microsoft.com/windows-server/storage/dfs-namespaces/create-a-dfs-folder-with-targets
- DFS Replication の概要（レプリケーション役割は同期に必要で、フォルダー ターゲットの存在自体には不要）: https://learn.microsoft.com/windows-server/storage/dfs-replication/dfsr-overview

## 問題90
あなたの会社にはボストンとモントリオールにオフィスがあります。
オフィス間はしばしば輻輳する 10 Mbps の WAN 回線で接続されています。
ボストンのオフィスには次が含まれます:
* DC1 という名前の Active Directory Domain Services (AD DS) ドメイン コントローラー。
* Windows Server を実行し、ファイル サーバーの役割がインストールされている Server1 という名前のサーバー。

モントリオールのオフィスには Windows 10 を実行するクライアント コンピューターが 20 台あります。
モントリオールにはサーバーはありません。
会社は新しい業務 (LOB) アプリケーションをすべてのクライアント コンピューターに展開する計画です。
アプリケーションのインストール元ファイルは \\Server\Apps にあります。

Answer Area
On Server1:「Enable BranchCache in hosted cache mode.」or「Enable BranchCache in Distributed Cache mode.」or「Install tha BranchCache for network files role service.」
On the client computer:「Enable BranchCache in hosted cache mode.」or「Enable BranchCache in Distributed Cache mode.」or「Install the BranchCache for network files role service.」

## 解答90
- On Server1: Install the BranchCache for network files role service.
- On the client computer: Enable BranchCache in Distributed Cache mode.

## 解説90
- モントリオールにはサーバーがないため、クライアント同士でキャッシュを共有する Distributed Cache モードが最小の管理労力で適用できます。Hosted Cache モードはブランチ側にホスト キャッシュ サーバーが必要なため不適です。
- SMB 共有 (\\Server\Apps) を提供するコンテンツ サーバー側 (Server1) では、BranchCache for Network Files ロール サービスをインストールしてハッシュの公開を有効化する必要があります。これによりクライアントはコンテンツのハッシュを取得し、ブランチ内でキャッシュを共有できます。

根拠 (Microsoft 公式ドキュメント):
- BranchCache の概要と 2 つのモード (Distributed/Hosted)。ブランチにサーバーがない場合は Distributed Cache モードを使用: https://learn.microsoft.com/windows-server/networking/branchcache/branchcache
- ファイル サーバーでの BranchCache 構成 (BranchCache for Network Files の有効化とハッシュ公開): https://learn.microsoft.com/windows-server/networking/branchcache/deploy/branchcache-deploy

## 問題91
Serve1 という名前の Azure 仮想マシンがあり、ネットワーク管理アプリケーションを実行しています。
Server1 には次のネットワーク構成があります:
- ネットワーク インターフェイス: Nic1
- IP アドレス: 10.1.1.1/24
- 接続先: Vnet1/Subnet1

Server1 を Vnet1/Subnet2 という追加のサブネットに接続する必要があります。
何を行うべきですか？
A. Nic1 の IP 構成を変更する。
B. Subnet2 にプライベート エンドポイントを作成する。
C. Server1 にネットワーク インターフェイスを追加する。
D. Nic1 に IP 構成を追加する。

## 解答91
C. Server1 にネットワーク インターフェイスを追加する。

## 解説91
- 1 つのネットワーク インターフェイス (NIC) は 1 つのサブネットにのみ接続できます。複数サブネットへ同時接続したい場合、VM サイズが対応していれば追加の NIC をアタッチし、それぞれの NIC を同一仮想ネットワーク内の別サブネットへ接続します。
- 「IP 構成の追加/変更 (A, D)」は同一サブネット内で複数のプライベート IP を持たせるための設定であり、別サブネットへの接続にはなりません。
- 「プライベート エンドポイント (B)」は PaaS リソース等へのプライベート接続用であり、VM を別サブネットに接続する手段ではありません。

根拠（Microsoft 公式ドキュメント）
- Overview of network interfaces: 各 NIC はサブネットに接続される。複数 NIC のサポートや制約についての説明。
  https://learn.microsoft.com/azure/virtual-network/virtual-network-network-interface
- Create a VM with multiple NICs: 複数 NIC を用いて VM を複数サブネットに接続する手順と要件（対応 VM サイズ、同一 VNet 内サブネットなど）。
  https://learn.microsoft.com/azure/virtual-network/virtual-network-create-vm-multiple-nics?tabs=portal
- Multiple IP addresses for Azure VMs: 追加 IP 構成は同一サブネットでの複数 IP 割り当てに用いる旨。
  https://learn.microsoft.com/azure/virtual-network/ip-services/virtual-network-multiple-ip-addresses
- Private endpoints: プライベート エンドポイントは PaaS リソース等へのプライベート接続を提供する機能であり、VM のサブネット接続用途ではない。
  https://learn.microsoft.com/azure/private-link/private-endpoint-overview

## 問題92
Windows Server を実行するファイル サーバーが 5 台あります。
ユーザーが .mov 拡張子のビデオ ファイルをファイル サーバー上の共有フォルダーにアップロードできないようにブロックする必要があります。
他の種類のファイルはすべて許可されなければなりません。
ソリューションは管理上の労力を最小限に抑える必要があります。
何を作成すべきですか？
A. Dynamic Access Control の中央アクセス ポリシー
B. ファイル スクリーン (File Screen)
C. Dynamic Access Control の中央アクセス ルール
D. データ損失防止 (DLP) ポリシー

## 解答92
B. a file screen

## 解説92
- 共有フォルダーに保存できるファイルの種類を拡張子などで制御・ブロックするには、File Server Resource Manager (FSRM) の「ファイル スクリーン」を使用します。テンプレートやファイル グループを使って一貫したポリシーを複数サーバーに展開でき、最小の管理労力で要件を満たせます。
- Dynamic Access Control (中央アクセス ルール/ポリシー) は、分類やユーザー/デバイス クレームに基づくアクセス制御の仕組みであり、特定拡張子の保存自体をブロックする機能ではありません。
- DLP ポリシーは主に Microsoft Purview/Microsoft 365 などのクラウド サービスで機密データの流出を防ぐためのもので、Windows Server の SMB 共有に対する拡張子ベースの保存禁止には適しません。

根拠（Microsoft 公式ドキュメント）:
- 「ファイル スクリーンを使用すると、ユーザーがサーバーに保存できるファイルの種類を制御できます」(FSRM: File Screening Management)
  https://learn.microsoft.com/windows-server/storage/fsrm/file-screening-management
- FSRM の概要（ファイル制御/クォータ/分類の機能）
  https://learn.microsoft.com/windows-server/storage/fsrm/fsrm-overview
- Dynamic Access Control の概要（分類・クレーム・中央アクセス ポリシーによるアクセス制御）
  https://learn.microsoft.com/windows-server/identity/solution-guides/dac-overview

## 問題93
ネットワークには DHCP サーバーが存在します。
新しいサブネットを追加し、そのサブネットに Windows Server を展開する予定です。
そのサーバーを DHCP リレー エージェントとして使用できるようにする必要があります。
サーバーにどの役割をインストールすべきですか？
A. Remote Access
B. Network Policy and Access Services
C. Network Controller
D. DHCP Server

## 解答93
A. Remote Access

## 解説93
- Windows Server で DHCP リレー エージェント機能を提供するのは、Routing and Remote Access Service (RRAS) を含む「Remote Access」サーバー役割です。RRAS の Routing 機能内で DHCP Relay Agent を構成します。
- DHCP Server 役割は実際にアドレスを配布するサーバーに用いるもので、リレー用途には不要です。Network Policy and Access Services は NPS などのポリシー管理、Network Controller は SDN 管理の役割であり、本要件には該当しません。

根拠（Microsoft 公式ドキュメント）
- Remote Access の役割サービス（Routing を含む）: Remote Access 役割でルーティング機能を提供
  https://learn.microsoft.com/windows-server/remote/remote-access/ras/ras-role-services
- DHCP の概要とリレーの必要性: 異なるサブネット間で DHCP を利用する場合は DHCP リレーまたはルーターでの転送が必要
  https://learn.microsoft.com/windows-server/networking/technologies/dhcp/dhcp-top

## 問題94
Azure サブスクリプションに VNet1 という名前の仮想ネットワークがあります。
VNet1 には Subnet1、Subnet2、Subnet3 という 3 つのサブネットが含まれています。
次の設定で仮想マシンを 1 台デプロイしました。
- Name: VM1
- Subnet: Subnet2
- Network interface name: NIC1
- Operating system: Windows Server 2022

VM1 が Subnet1 と Subnet3 間のトラフィックをルーティングできるようにする必要があります。
ソリューションは管理上の労力を最小化しなければなりません。
何をすべきですか？回答するには、解答領域で適切なオプションを選択してください。
注意: 各正解は 1 ポイントの価値があります。

解答領域
From the Azure portal:「Associate a routing table with Subnet2.」or「Attach two additional interfaces to VM1.」or「Enable IP forwarding for NIC1.」
On VM1:「Install and configure a network controller.」or「Install and configure Routing and Remote Access.」or「Run the route add command.」

## 解答94
|項目|選択肢|
|---|---|
|From the Azure portal|Enable IP forwarding for NIC1.|
|On VM1|Install and configure Routing and Remote Access.|

## 解説94
- Azure で VM をネットワーク仮想アプライアンス（NVA）としてルーティングに使用する場合、NIC 側で IP 転送を有効にし（Azure 側の IP forwarding）、ゲスト OS 側でもパケット転送を有効化する必要があります。Windows Server では RRAS（Routing and Remote Access）で LAN ルーティングを構成します。
- 追加 NIC の取り付けは必須ではありません（単一 NIC の NVA でも UDR と併用してルーティングは可能）。また Subnet2 へのルート テーブル関連付けだけでは Subnet1/3 のトラフィック経路は変わらないため不十分です。本問の選択肢内で管理負荷を最小にする正解は「NIC の IP 転送を有効化」＋「RRAS によるルーティング構成」です。

根拠（Microsoft 公式ドキュメント）
- Azure VM の IP 転送（NVA の要件）: https://learn.microsoft.com/azure/virtual-network/virtual-network-network-interface#ip-forwarding
- ユーザー定義ルート（NVA 経由でのトラフィック誘導の一般原則）: https://learn.microsoft.com/azure/virtual-network/virtual-networks-udr-overview
- Windows Server の RRAS による LAN ルーティング: https://learn.microsoft.com/windows-server/remote/remote-access/remote-access

## 問題95
あなたのネットワークには、contoso.com という名前の単一ドメインの Active Directory Domain Services (AD DS) フォレストがあり、フォレストには単一の Active Directory サイトが含まれています。

新しいデータセンターにある Server1 という名前のサーバーに、読み取り専用ドメイン コントローラー (RODC) を展開する予定です。
User1 という名前のユーザーは、Server1 のローカル Administrators グループのメンバーです。

次の要件を満たす展開計画を推奨する必要があります:
- User1 が Server1 上で RODC のインストールを実行できるようにすること
- Server1 への AD DS レプリケーション スケジュールを制御できるようにすること
- Server1 が RemoteSite1 という新しいサイトに属すること
- 最小特権の原則に従うこと

どの 3 つのアクションを順番に実行することを推奨しますか？回答するには、アクションの一覧から適切なアクションを解答領域に移動し、正しい順序に並べてください。

Select and Place（アクション一覧）:
- Instruct User1 to run the Active Directory Domain Services installation Wizard on Server1.
- Create a site and a subnet.
- Create a site link.
- Pre-create an RODC account.
- Add User1 to the Contoso\Administrators group.

## 解答95
|順序|アクション|
|---|---|
|1|Create a site and a subnet.|
|2|Pre-create an RODC account.|
|3|Instruct User1 to run the Active Directory Domain Services installation Wizard on Server1.|

## 解説95
- レプリケーション スケジュールを制御するには、Server1 を別サイト（RemoteSite1）に配置する必要があります。サイト間レプリケーションはサイト リンクでスケジュール管理され、既定の DEFAULTIPSITELINK を編集して制御できます。よって新しい「サイトとサブネット」の作成が前提です。
- 最小特権で User1 に RODC インストールを実施させるには、RODC アカウントを事前作成（Pre-create）し、そのウィザードで「インストールの委任」を User1 に付与します。これによりドメイン管理者権限を付与せずにインストール可能です。
- その後、User1 に Server1 上で AD DS インストール ウィザードを実行させ、事前作成済みの RODC アカウントにアタッチして RODC を構成します。
- ドメインの Administrators グループに User1 を追加するのは最小特権に反し不要です。「サイト リンクの作成」は必須ではありません（新サイトは既定のサイト リンクに参加し、スケジュールはそのリンクで制御可能）。

根拠（Microsoft 公式ドキュメント）
- RODC の概要と委任されたインストール（事前作成と委任）: https://learn.microsoft.com/windows-server/identity/ad-ds/plan/rodc-plan
- RODC のインストール（事前作成済みアカウントを使用した手順）: https://learn.microsoft.com/windows-server/identity/ad-ds/deploy/install-a-read-only-domain-controller-rodc
- サイト/サブネット/サイト リンク（サイト間レプリケーションはサイト リンクでスケジュール制御）: https://learn.microsoft.com/windows-server/identity/ad-ds/plan/understanding-sites-subnets-and-site-links

## 問題96
あなたのネットワークには、contoso.com という名前の Active Directory Domain Services (AD DS) フォレストがあります。
ルート ドメインには、次の表に示すドメイン コントローラーが含まれます。
|Name|FSMO role|
|---|---|
|DC1|Domain naming master|
|DC2|RID master|
|DC3|PDC emulator|
|DC4|Schema master|
|DC5|Infrastructure master|

どのドメイン コントローラーの障害により、アプリケーション パーティションを作成できなくなりますか？

A. DC1
B. DC2
C. DC3
D. DC4
E. DC5

## 解答96
A. DC1

## 解説96
- アプリケーション ディレクトリ パーティション（Application partition）の追加・削除は、フォレスト内で「ドメイン名前付けマスター」(Domain naming master) を保持するドメイン コントローラーのみが実行できます。したがって、この役割を持つ DC1 が利用不能だと、アプリケーション パーティションを新規作成できません。
- スキーマ変更は Schema master（DC4）が必要ですが、アプリケーション パーティションの作成自体は Domain naming master の役割に依存します。

根拠（Microsoft 公式ドキュメント）:
- Operations master roles（Domain naming master はドメインやアプリケーション ディレクトリ パーティションの追加/削除を一元的に処理）
  https://learn.microsoft.com/windows-server/identity/ad-ds/plan/operations-master-roles

## 問題97
あなたのネットワークには、複数サイトで構成された Active Directory Domain Services (AD DS) フォレストがあります。
各 Active Directory サイトは、手動で構成されたサイト リンクと自動生成された接続によって接続されています。
Active Directory への変更の収束時間を最小化する必要があります。
何をすべきですか？

A. 各サイト リンクのレプリケーション スケジュールを変更する。
B. 各サイト リンクのコストを変更する。
C. すべてのサイト リンクを含むサイト リンク ブリッジを作成する。
D. 各サイト リンクの options 属性を変更する。

## 解答97
D

## 解説97
- 目的は「収束時間の最小化（最短での複製）」です。既定ではサイト間レプリケーションはスケジュール/間隔（既定180分、最小15分）に従いますが、サイト リンクに対して「変更通知によるサイト間レプリケーション」を有効化すると、変更が発生した直後にレプリケーションをトリガーできます。これはサイト リンク オブジェクトの options 属性（Use intersite change notification）を有効にすることで行います。最短での収束を狙う要件に合致します。
- A（スケジュール/間隔の調整）は最小15分までしか短縮できず、「最小化」という観点では変更通知に劣ります。
- B（コスト変更）は経路選択に影響するだけで、レプリケーション頻度や遅延の短縮には直結しません。
- C（サイト リンク ブリッジ）はトポロジの連結性に関する設定であり、収束時間短縮の直接手段ではありません（既定の「すべてのサイト リンクをブリッジ」設定が有効な環境も多い）。

根拠（Microsoft 公式ドキュメント）
- サイト、サブネット、サイト リンク（サイト間レプリケーションはスケジュール/間隔で制御、変更通知を有効化可能）: https://learn.microsoft.com/windows-server/identity/ad-ds/plan/understanding-sites-subnets-and-site-links
- AD レプリケーションの仕組み（サイト内は変更通知、サイト間は既定でスケジュール。収束時間短縮には変更通知の有効化が有効）: https://learn.microsoft.com/windows-server/identity/ad-ds/plan/active-directory-replication

## 問題98
オンプレミスの Active Directory Domain Services (AD DS) ドメインがあり、Azure Active Directory (Azure AD) テナントと同期しています。
複数の Windows 10 デバイスが Azure AD ハイブリッド参加 (hybrid-joined) です。
ユーザーがこれらのデバイスにサインインする際に、Windows Hello for Business を使用できるようにする必要があります。
Azure AD Connect のどのオプション機能を選択すべきですか？

A. Device writeback
B. Group writeback
C. Azure AD app and attribute filtering
D. Password writeback
E. Directory extension attribute sync

## 解答98
A. Device writeback

## 解説98
- ハイブリッド環境で Windows Hello for Business (WHfB) を使用してオンプレミス リソースにサインインさせるには、Azure AD に登録/参加したデバイスをオンプレミス AD に書き戻す「Device writeback」を Azure AD Connect で有効化するのが前提となります。これにより、オンプレミス側の認証およびポリシー評価でデバイス状態を活用でき、WHfB のハイブリッド シナリオが成立します。
- 併せて、WHfB のキー信頼 (Key trust) 方式では、ユーザーの公開鍵がオンプレミス AD のユーザー オブジェクト (msDS-KeyCredentialLink) に格納され、Azure AD Connect により同期/書き戻しされます。
- Password writeback (D) は SSPR のパスワード書き戻し機能であり本件には不要、Group writeback (B) は M365 グループの書き戻し、Directory extension attribute sync (E) は拡張属性同期、App and attribute filtering (C) は同期対象の制限で、いずれも WHfB の有効化要件ではありません。

根拠（Microsoft 公式ドキュメント）:
- Device writeback の概要と用途（オンプレミス条件付きアクセスや Windows Hello for Business で使用）
  https://learn.microsoft.com/azure/active-directory/hybrid/how-to-connect-device-writeback
- Windows Hello for Business のハイブリッド展開（キー信頼の仕組みと AD へのキー格納/msDS-KeyCredentialLink）
  https://learn.microsoft.com/windows/security/identity-protection/hello-for-business/hello-architecture

## 問題99
あなたのネットワークには contoso.com という名前の Active Directory Domain Services (AD DS) フォレストがあり、フォレストには east.contoso.com という子ドメインが含まれます。
contoso.com ドメインに Admin1 と Admin2 という 2 人のユーザーを作成しました。
次の要件を満たす必要があります:
- Admin1 は Active Directory サイトを作成および管理できること。
- Admin2 は east.contoso.com ドメインにドメイン コントローラーを展開できること。
ソリューションは最小特権の原則に従う必要があります。

各ユーザーをどのグループに追加すべきですか？
回答するには、解答領域で適切なオプションを選択してください。
注意: 各正解は 1 ポイントの価値があります。

Hot Area（解答領域の選択肢）
Admin1: 「Contoso\Administrators」 または 「Contoso\Domain Admins」 または 「Contoso\Enterprise Admins」 または 「East\Administrators」 または 「East\Domain Admins」
Admin2: 「Contoso\Administrators」 または 「Contoso\Domain Admins」 または 「Contoso\Enterprise Admins」 または 「East\Administrators」 または 「East\Domain Admins」

## 解答99
|ユーザー|追加するグループ|
|---|---|
|Admin1|Contoso\Enterprise Admins|
|Admin2|East\Domain Admins|

## 解説99
- Active Directory サイト/サブネット/サイト リンクはフォレスト全体の構成 (Configuration パーティション) に格納されるため、既定ではフォレスト範囲の管理権限が必要です。最小特権でサイト管理を行う既定のグループは Enterprise Admins です（必要に応じて委任も可能）。
- 既存ドメインにドメイン コントローラーを追加（昇格）するには、対象ドメインの Domain Admins（または Enterprise Admins）メンバーシップ、もしくは同等の委任が必要です。最小特権の観点では、east.contoso.com 側の「East\\Domain Admins」が適切です。
- ルート ドメインの Domain Admins（Contoso\\Domain Admins）は子ドメインに対する管理権限を持ちません。また Administrators グループはフォレスト全体や子ドメイン全体の権限を自動的に付与するものではありません。

根拠（Microsoft 公式ドキュメント）
- Enterprise Admins（フォレスト全体の変更を実行できる組み込みグループ）: https://learn.microsoft.com/windows-server/identity/ad-ds/manage/understand-security-groups#enterprise-admins
- Domain Admins（ドメイン全体の管理権限を持つ）: https://learn.microsoft.com/windows-server/identity/ad-ds/manage/understand-security-groups#domain-admins
- ドメイン コントローラーのインストールに必要なアカウント要件（Domain Admins または Enterprise Admins、または委任）: https://learn.microsoft.com/windows-server/identity/ad-ds/deploy/install-a-domain-controller#account-requirements

## 問題100
contoso.com という名前の Active Directory Domain Services (AD DS) フォレストを新規に展開しました。
ドメインには DC1、DC2、DC3 という 3 台のドメイン コントローラーが含まれます。
Default-First-Site-Name を Site1 に名前変更しました。
DC1、DC2、DC3 をそれぞれ異なる場所のデータセンターに出荷する予定です。
次の要件を満たすように DC1、DC2、DC3 間のレプリケーションを構成する必要があります:
✑ 各ドメイン コントローラーは、それぞれ独自の Active Directory サイトに配置されていること。
✑ 各サイト間のレプリケーション スケジュールは、それぞれ独立して制御できること。
✑ レプリケーションの中断を最小限に抑えること。
Active Directory サイトとサービス コンソールで、どの 3 つの操作をどの順序で実行する必要がありますか? 回答するには、操作の一覧から適切なものを解答エリアに移動し、正しい順序に並べ替えてください。
Select and Place:
- Create a connection object between DC1 and DC2.
- Create an additional site link that contains Site1 and Site2.
- Create two additional sites named Site2 and Site3. Move DC2 to Site2 and DC3 to Site3.
- Create a connection object between DC2 and DC3.
- Remove Site2 from DEFAULTIPSITELINK.

## 解答100
1. Create two additional sites named Site2 and Site3. Move DC2 to Site2 and DC3 to Site3.
2. Create an additional site link that contains Site1 and Site2.
3. Remove Site2 from DEFAULTIPSITELINK.

## 解説100
- 要件「各 DC は独自サイトに」は、新規に Site2、Site3 を作成し、それぞれに DC2、DC3 を移動して満たします。
- サイト間レプリケーションのスケジュールやコストは「サイト リンク」で制御します。Site1–Site2 用に専用のサイト リンクを作成し、Site2 を DEFAULTIPSITELINK から外すことで、Site1–Site2 と Site1–Site3 のスケジュールを独立して設定できます（ハブ&スポーク）。
- レプリケーション中断を最小化するため、サイト リンクを先に作成してから DEFAULTIPSITELINK のメンバー変更を行い、接続性を維持します。接続オブジェクトは KCC がサイト リンクに基づいて自動生成するため、手動作成は不要です。

根拠（Microsoft 公式ドキュメント）:
- Active Directory のサイト、サブネット、サイト リンクの概念（サイト リンクでレプリケーション スケジュール/コストを設定）
  https://learn.microsoft.com/windows-server/identity/ad-ds/plan/understanding-active-directory-sites-subnets-and-site-links
- サイト間レプリケーションの仕組み（サイト リンクに基づき KCC が接続オブジェクトを自動生成、スケジュールはサイト リンクで制御）
  https://learn.microsoft.com/windows-server/identity/ad-ds/plan/active-directory-replication-topology

## 問題101
あなたのネットワークには、次の図に示すとおり 3 つの Active Directory Domain Services (AD DS) フォレストがあります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/6c798677-9f84-4e47-acc2-868667d79332.png)


ネットワークには、次の表のユーザーが含まれています。
|名前|ドメイン|
|---|---|
|User1|east.contoso.com|
|User2|fabrikam.com|

ネットワークには、次の表のセキュリティ グループが含まれています。
|名前|種類|ドメイン|
|---|---|---|
|Group1|ドメイン ローカル|west.adatum.com|
|Group2|ユニバーサル|contoso.com|
|Group3|ユニバーサル|east.contoso.com|

次の各記述について、正しければ Yes、正しくなければ No を選択してください。
注: 各正解は 1 点の価値があります。
Hot Area:
Answer Area
|記述|Yes|No|
|---|---|---|
|User1 を Group1 に追加できる。| | |
|User2 を Group3 に追加できる。| | |
|Group2 に fabrikam.com ドメイン内のリソースに対する権限を付与できる。| | |

## 解答101
|記述|Yes|No|
|---|---|---|
|User1 を Group1 に追加できる。|Yes| |
|User2 を Group3 に追加できる。| |No|
|Group2 に fabrikam.com ドメイン内のリソースに対する権限を付与できる。| |No|

## 解説101
- User1 → Group1 (Yes): ドメイン ローカル グループは、同一フォレスト内だけでなく信頼された外部ドメイン/フォレストのメンバーも含められます。信頼された外部主体をグループに追加すると「Foreign Security Principal」が自動生成されます。
  - 参考: Microsoft Docs「Foreign security principals」(別フォレストのセキュリティ プリンシパルをグループに追加すると FSP が作成される) https://learn.microsoft.com/windows-server/identity/ad-ds/manage/understand-foreign-security-principals 
  - 参考: Microsoft Docs「Understand security groups — Group scope (Domain local)」(ドメイン ローカルはリソース割り当てに使用し、メンバーとして他ドメイン/ユニバーサル/グローバルを含められる) https://learn.microsoft.com/windows-server/identity/ad-ds/manage/understand-security-groups#domain-local-groups 
- User2 → Group3 (No): ユニバーサル グループのメンバーシップは同一フォレスト内に限定され、他フォレストのユーザーを直接メンバーにできません。
  - 参考: Microsoft Docs「Understand security groups — Group scope (Universal)」(ユニバーサルはフォレスト内のアカウント/グローバル/ユニバーサルのみを含められる) https://learn.microsoft.com/windows-server/identity/ad-ds/manage/understand-security-groups#universal-groups 
- Group2 → fabrikam.com リソース (No): 図の信頼は contoso.com ↔ adatum.com、adatum.com ↔ fabrikam.com のフォレスト トラストですが、フォレスト トラストは第三のフォレストへは非推移です。したがって contoso.com と fabrikam.com 間に信頼がなく、fabrikam.com 側で contoso.com のグループに権限を付与できません。
  - 参考: Microsoft Docs「Appendix C: Active Directory trusts — Forest trust」(フォレスト トラストは当事者間では推移的だが、他フォレストへは非推移) https://learn.microsoft.com/windows-server/identity/ad-ds/plan/security-best-practices/appendix-c--active-directory-trusts#forest-trust 
  - 参考: Microsoft Docs「Understand security groups」(異フォレストのアクセスには信頼が必要) https://learn.microsoft.com/windows-server/identity/ad-ds/manage/understand-security-groups

## 問題102
あなたのネットワークには Active Directory フォレストがあります。
フォレストには contoso.com と east.contoso.com という 2 つのドメインと、次の表に示すサーバーが含まれています。
|Name|Domain|Configuration|
|---|---|---|
|DC1|contoso.com|Domain Controller|
|Server1|contoso.com|Member server|
|DC2|east.contoso.com|Domain controller|
|Server2|east.contoso.com|Member server|

Contoso.com には User1 というユーザーが存在します。

User1 を contoso.com の組み込みの Backup Operators グループに追加します。

User1 はどのサーバーをバックアップできますか?

A. DC1 only
B. Server1 only
C. DC1 and DC2 only
D. DC1 and Server1 only
E. DC1, DC2, Server1, and Server2

## 解答102
|解答|
|---|
|A. DC1 only|

## 解説102
- ドメインの「Backup Operators」（組み込み）はドメイン コントローラー上でのバックアップ/復元権限を付与します。Microsoft 公式「セキュリティ グループを理解する」では、Backup Operators の説明として「このグループのメンバーは、ドメイン内のドメイン コントローラー上のファイルのバックアップと復元が可能」だと記載されています。よって contoso.com の DC（DC1）で有効です。参考: https://learn.microsoft.com/windows-server/identity/ad-ds/manage/understand-security-groups#backup-operators
- メンバー サーバーのバックアップ権限は、そのサーバーのローカル「Backup Operators」グループ（ローカル ユーザー権利: SeBackupPrivilege/SeRestorePrivilege）により付与されます。ドメイン側の Backup Operators に追加しただけでは、メンバー サーバーのローカル グループ メンバーには自動的に含まれません。参考: https://learn.microsoft.com/windows/security/identity-protection/access-control/default-local-groups#backup-operators
- また、ドメイン ローカル グループの権限付与は同一ドメイン内のリソースに範囲が限定され、east.contoso.com の DC2 には及びません。参考: https://learn.microsoft.com/windows-server/identity/ad-ds/manage/understand-security-groups#group-scope

以上より、User1 が既定でバックアップできるのは contoso.com のドメイン コントローラー (DC1) のみです。

## 問題103
あなたのネットワークには、contoso.com という名前の Azure Active Directory Domain Services (Azure AD DS) ドメインがあります。

contoso.com に参加している Azure 仮想マシン上のローカル ユーザー アカウントに対して、パスワード ポリシーを構成する必要があります。

何をすべきですか？回答するには、解答エリアで適切なオプションを選択してください。

注意: 各正解は 1 ポイントの価値があります。
Answer Area
Sign in by using a user account that is a member of the:「AAD DC Administrators group」or「Administrators group」or「Domain Admins group」
Use a Group Policy Object(GPO) linked to the:「AADDC Computers organizational unit (OU)」or「AADDC Users organizational unit(OU)」or「Computers container」

## 解答103
- Sign in by using a user account that is a member of the: AAD DC Administrators group
- Use a Group Policy Object (GPO) linked to the: AADDC Computers organizational unit (OU)

## 解説103
- Azure AD DS では、ドメイン全体や GPO の管理は Domain Admins ではなく「AAD DC Administrators」グループのメンバーが行います。よって、GPMC でのパスワード ポリシー構成には AAD DC Administrators が必要です。
- ローカル アカウントのパスワード ポリシーは、対象コンピューターに適用される GPO の「アカウント ポリシー (Computer Configuration\Windows Settings\Security Settings\Account Policies\Password Policy)」で管理できます。Azure AD DS の管理ドメインでは、コンピューター アカウントは既定で「AADDC Computers」OU に配置されるため、該当 OU にリンクした GPO でローカル アカウントのパスワード要件を構成します（ユーザー対象の「AADDC Users」OUや既定の Computers コンテナーではありません）。

根拠（Microsoft 公式ドキュメント）:
- Azure AD DS の GPO 管理（AAD DC Administrators の役割、AADDC Users/Computers OU での GPO 適用）
  https://learn.microsoft.com/azure/active-directory-domain-services/tutorial-configure-group-policy
  https://learn.microsoft.com/azure/active-directory-domain-services/concepts-azure-ad-ds-default-forest#organizational-units-in-a-managed-domain
- パスワード ポリシー（アカウント ポリシーの場所と設定）
  https://learn.microsoft.com/windows/security/threat-protection/security-policy-settings/password-policy

## 問題104
contoso.com に Admin1 というユーザーを作成する必要があります。
Admin1 は SRV1 上のファイルをバックアップおよび復元できなければなりません。
ソリューションは最小特権の原則を満たす必要があります。

## 解答104
- Azure ポータルにユーザー管理者としてサインインし、Azure Active Directory > Users > New user からユーザー「Admin1」を作成する。
- SRV1 を保護する（または保護予定の）Recovery Services コンテナー（Vault）に移動し、Access control (IAM) > Add role assignment を開く。
- 役割に「Backup Operator」を選択し、メンバーに「Admin1」を割り当てる（スコープは最小＝該当の Vault）。

## 解説104
- 最小特権でバックアップ／復元を許可するロールは「Backup Operator」です。バックアップの削除や登録解除などの破壊的操作はできないため、要件と最小特権を同時に満たします。
- 権限は対象 Vault スコープでの RBAC で割り当てます。ユーザー作成は Azure AD (Entra ID) で行います。

根拠（Microsoft 公式ドキュメント）:
- Azure AD でユーザーを追加: https://learn.microsoft.com/azure/active-directory/fundamentals/add-users-azure-active-directory
- Azure Backup の RBAC（Backup Operator の権限）: https://learn.microsoft.com/azure/backup/backup-rbac-rs-vault

## 問題105
BranchAdmins グループのメンバーの最小パスワード長を 12 文字にする必要があります。
解決策は BranchAdmins グループのみに影響する必要があります。

このタスクを完了するには、必要なコンピューターにサインインしてください。

## 解答105
以下は Active Directory 管理センター（ADAC）を用いた正答手順です。

1. 管理者権限でサインインし、ADAC（dsac.exe）を開く。
2. ナビゲーションで対象ドメインを選択し、左ペインの「Password Settings」を開く（見当たらない場合は［管理］>［ナビゲーション ノードの追加］で対象ドメインを追加）。
3. 右側の［タスク］ペインで［新規作成］>［Password Settings］をクリック。
4. 「Create Password Settings」で次を設定：
   - Name: PSO-BranchAdmins-MinLen12（任意名）
   - Precedence: 1（他 PSO と競合しない最優先の小さい数値）
   - Enforce minimum password length にチェックし、Minimum password length (characters): 12
   - そのほかの項目は要件に合わせ既定のまま。
5. 画面下部の「Directly Applies To」で［Add］をクリックし、「BranchAdmins」グループを追加して［OK］。
6. ［OK］で作成を完了。これにより、グループ BranchAdmins のメンバーに対してのみ最小 12 文字のパスワード長が適用される。

## 解説105
- AD DS の「細分化されたパスワード ポリシー（Fine-Grained Password Policies, FGPP）」は、同一ドメイン内でユーザーや（グローバル）セキュリティ グループごとに異なるパスワード/ロックアウト設定を適用できます。FGPP は Password Settings Object（PSO）として作成し、「Directly applies to」で対象のユーザーまたはグループにリンクします。これによりドメイン全体ではなく指定対象のみに影響します。
  - 根拠（Microsoft Docs）: Fine-Grained Password Policies — 「ドメイン内の特定のユーザーまたはグローバル セキュリティ グループに対して異なるパスワードおよびアカウント ロックアウト ポリシーを構成できる」
    https://learn.microsoft.com/windows-server/identity/ad-ds/manage/component-updates/fine-grained-password-policies
- ADAC では［新規作成］>［Password Settings］から PSO を作成し、Minimum password length を指定、Directly Applies To にグループ（例: BranchAdmins）を追加して適用できます。
  - 根拠（Microsoft Docs）: Active Directory Administrative Center での FGPP 管理（Password Settings オブジェクト作成/適用）
    https://learn.microsoft.com/windows-server/identity/ad-ds/get-started/adac/introduction-to-active-directory-administrative-center-enhancements--level-100-
- PowerShell でも同等に実施可能です（参考例）。
  - New-ADFineGrainedPasswordPolicy -Name "PSO-BranchAdmins-MinLen12" -Precedence 1 -MinPasswordLength 12
  - Add-ADFineGrainedPasswordPolicySubject -Identity "PSO-BranchAdmins-MinLen12" -Subjects "BranchAdmins"
  - 根拠（Microsoft Docs）:
    New-ADFineGrainedPasswordPolicy: https://learn.microsoft.com/powershell/module/activedirectory/new-adfinegrainedpasswordpolicy
    Add-ADFineGrainedPasswordPolicySubject: https://learn.microsoft.com/powershell/module/activedirectory/add-adfinegrainedpasswordpolicysubject

## 問題106
組織単位 (OU)「Server Admins」に属するユーザーが、ドメイン内のコンピューターにサインインしたとき、デスクトップに \\srv1.contoso.com\data という名前のフォルダーへのショートカットを必ず持つように、グループ ポリシーの基本設定 (Group Policy Preferences) を構成する必要があります。

このタスクを完了するには、必要なコンピューターにサインインしてください。

## 解答106
1. 管理端末またはドメイン コントローラーで「グループ ポリシーの管理 (gpmc.msc)」を開く。
2. OU「Server Admins」を右クリックし「このドメインに GPO を作成し、ここにリンク」を選択して新しい GPO を作成 (例: CreateDesktopShortcut_ServerAdmins)。
3. 作成した GPO を右クリックして「編集」。
4. ユーザーの構成 > 基本設定 > Windows の設定 > Shortcuts を開き、右クリックして「新規」>「Shortcut」。
5. ショートカット項目を次のように設定して OK。
   - Action: Create (必要に応じて Replace)
   - Name: Data on srv1 (任意名)
   - Target type: File System Object
   - Location: Desktop
   - Target path: \\srv1.contoso.com\data
   - Start in/Icon (任意)／必要に応じて「Common」タブで「Run in logged-on user’s security context (user policy option)」にチェック
6. GPO は OU「Server Admins」にリンク済みのため、当該 OU のユーザーがドメイン内の任意のコンピューターへサインインすると、デスクトップに上記ショートカットが作成される。必要に応じて「gpupdate /force」または再サインインで適用を確認。

## 解説106
- Group Policy Preferences の「Shortcuts」拡張は、ユーザー構成からデスクトップ等にファイル システム オブジェクトへのショートカットを作成できます。ターゲット パスに共有フォルダー (例: \\srv1.contoso.com\data) を指定すれば、サインイン時にショートカットが展開されます。根拠: Microsoft Learn「Group Policy Preferences – Shortcuts (Shortcuts preference items)」
  https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc753092(v=ws.10)
- ユーザー設定の GPO を該当ユーザーが属する OU にリンクすると、そのユーザーがログオンする任意のコンピューターで設定が適用されます。根拠: Microsoft Learn「Group Policy の基礎 (What is Group Policy?)」および「GPO のリンクと適用範囲」
  https://learn.microsoft.com/en-us/windows/client-management/group-policy-overview
  https://learn.microsoft.com/en-us/windows/client-management/linking-gpos
- 適用確認や即時反映には gpupdate を使用可能。根拠: Microsoft Learn「gpupdate コマンド」
  https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/gpupdate

## 問題107
シアトルのサイトで、DC3 という名前のドメイン コントローラーを昇格させる予定です。

DC3 が DC1 および DC2 とのみ、午後 8 時から午前 6 時の間にレプリケーションを行うようにする必要があります。

このタスクを完了するには、必要なコンピューターにサインインしてください。

## 解答107
以下は Active Directory サイトとサービス (dssite.msc) を用いて達成する手順です。

1. 管理コンピューターまたは任意のドメイン コントローラーで「Active Directory サイトとサービス」を開く。
2. サイト > Seattle > Servers > DC3 > NTDS Settings を展開する。
3. DC3 の NTDS Settings 直下にある既存の接続オブジェクトのうち、DC1/DC2 以外からのものがあれば削除する（DC3 が他の DC とレプリケートしないようにする）。
4. DC3 の NTDS Settings を右クリックし「New Active Directory Domain Services Connection」を選択。
   - 一覧から DC1 を選んで接続オブジェクトを作成。
   - 作成した接続オブジェクトを開き「Schedule」をクリック。
   - すべての時間帯を「Not available」にした後、毎日 20:00～06:00 のみ「Available」に設定して OK。
5. 手順 4 を DC2 についても繰り返す（DC2 から DC3 への接続オブジェクトを作成し、同じスケジュール 20:00～06:00 を設定）。
6. 必要に応じて、Seattle サイトと DC1/DC2 が属するサイトを結ぶサイト リンクのスケジュールも 20:00～06:00 にそろえる（Inter-Site Transports > IP > 対象のサイト リンクのプロパティ > Change Schedule）。

これで、DC3 は DC1 と DC2 からのみ、指定した時間帯にレプリケーションします。

## 解説107
- AD DS のレプリケーションは、サイト間では「サイト リンク」と各 DC の「接続オブジェクト（Connection Object）」により制御できます。接続オブジェクトは手動で作成でき、スケジュールを設定することで特定の時間帯のみレプリケーションを許可できます。加えて、不要な接続オブジェクトを削除すれば、レプリケーション パートナーを限定できます。
  - 根拠: Active Directory Replication Topologies（接続オブジェクトは KCC により自動生成されるが、管理者が手動で作成・管理でき、スケジュール設定が可能）https://learn.microsoft.com/windows-server/identity/ad-ds/plan/active-directory-replication-topologies
- サイト間のレプリケーション時間はサイト リンクのスケジュールでも制御できます。必要に応じて接続オブジェクトの設定と整合させることで、時間帯制御を確実にできます。
  - 根拠: Understanding Active Directory Sites（サイト リンクはレプリケーションのコストとスケジュールを持つ）https://learn.microsoft.com/windows-server/identity/ad-ds/plan/understanding-active-directory-sites

## 問題108
あなたのネットワークには、contoso.com と fabrikam.com という 2 つの Active Directory Domain Services (AD DS) フォレストがあります。
Contoso.com には、amer.contoso.com、apac.contoso.com、emea.contoso.com という 3 つの子ドメインがあります。
Fabrikam.com には、apac.fabrikam.com という子ドメインがあります。
contoso.com と fabrikam.com の間には双方向のフォレスト トラストが存在します。

contoso.com フォレストのユーザーに、fabrikam.com フォレスト内のリソースへのアクセスを提供する必要があります。
ソリューションは次の要件を満たさなければなりません。

• contoso.com のユーザーは、contoso.com フォレスト内のグループにのみ直接追加されること。
• fabrikam.com のリソースへのアクセス許可は、fabrikam.com フォレスト内のグループにのみ直接付与されること。
• グループの数は最小限に抑えること。

どの種類のグループを使用してユーザーを整理し、アクセス許可を割り当てるべきですか。
回答するには、適切なグループの種類を正しい要件にドラッグします。
各グループは 1 回以上使用しても、使用しなくてもかまいません。
コンテンツを表示するには、ペイン間の分割バーをドラッグするか、スクロールする必要がある場合があります。

注: 各正解は 1 点の価値があります。
Answer Area
選択肢
- Domain global
- Domain local
- Universal

Organize users:
Assign permissions:

## 解答108
|要件|選ぶグループの種類|
|---|---|
|Organize users|Universal|
|Assign permissions|Domain local|

## 解説108
- ユニバーサル グループは、同一フォレスト内の複数ドメインからメンバー（ユーザー/グローバル/ユニバーサル）を集約でき、フォレスト全体で利用できます。contoso.com には 3 ドメインがあるため、ユーザーをユニバーサル グループに集約することでグループ数を最小化できます。
  - 公式: Understand security groups — Universal groups（メンバー/用途）https://learn.microsoft.com/windows-server/identity/ad-ds/manage/understand-security-groups#universal-groups
- ドメイン ローカル グループは、作成されたドメイン内のリソースに対する権限の割り当てに使用します。メンバーには、信頼された他ドメイン/フォレストからのグローバル/ユニバーサル グループやアカウントを含められます。よって fabrikam.com 側でドメイン ローカルに対して権限を直接付与し、そのメンバーとして contoso.com のユニバーサル グループを追加すれば要件を満たします。
  - 公式: Understand security groups — Domain local groups（権限付与の対象ドメイン/許容されるメンバー）https://learn.microsoft.com/windows-server/identity/ad-ds/manage/understand-security-groups#domain-local-groups
- なお、双方向フォレスト トラストがあるため、フォレスト間の認証とリソース共有が可能です。
  - 公式: Active Directory trusts — Forest trust（フォレスト間でのリソース共有）https://learn.microsoft.com/windows-server/identity/ad-ds/plan/security-best-practices/appendix-c--active-directory-trusts#forest-trust

## 問題109
あなたのネットワークには、contoso.com という名前の Active Directory Domain Services (AD DS) フォレストが含まれています。
フォレストには east.contoso.com という子ドメインがあり、次の表に示すサーバーがあります。
|Name|Domain|Description|
|---|---|---|
|DC1|contoso.com|スキーマ マスター、インフラストラクチャ マスター、ドメイン名前付けマスターの役割を保持|
|DC2|east.contoso.com|PDC エミュレーターと RID マスターの役割を保持し、グローバル カタログ サーバー|
|Server1|contoso.com|ファイル サーバー、DFS 名前空間、DFS レプリケーションのサーバー ロールを保持|

フォレスト全体のグループ ポリシー テンプレート ファイルを管理するための Central Store 用フォルダーを作成する必要があります。

フォルダーの名前は何にし、どのサーバーに作成すべきですか。
解答するには、解答エリアで適切なオプションを選択してください。

注: それぞれの正解は 1 ポイントです。
解答エリア
Name:「CentralDefinitions」または「PolicyDefinitions」または「TemplateDefinitions」
Server:「DC1 only」または「DC2 only」または「Server1 only」または「DC1 and DC2 only」または「DC1, DC2, and Server1」

## 解答109
Name: PolicyDefinitions
Server: DC1 and DC2 only

## 解説109
- Central Store はドメイン コントローラー上の SYSVOL 内「…\SYSVOL\<ドメイン名>\Policies\PolicyDefinitions」に作成します。フォルダー名は必ず「PolicyDefinitions」です。これにより GPO 編集時に中央ストアの ADMX/ADML が使用されます。
  - 根拠: Microsoft Learn「Central Store の作成と管理」(中央ストアの場所は %systemroot%\SYSVOL\domain\Policies\PolicyDefinitions)
    https://learn.microsoft.com/en-us/troubleshoot/windows-client/group-policy/create-and-manage-central-store
- 中央ストアは「ドメイン単位」であり、複数ドメインがある場合は各ドメインに中央ストアを作成します。したがって、contoso.com と east.contoso.com の各ドメインの DC（DC1 と DC2）に作成します。メンバー サーバー (Server1) は対象外です。
  - 根拠: Microsoft Learn「Administrative Templates in Group Policy」(Central Store は domain-wide、複数ドメインでは各ドメインに作成)
    https://learn.microsoft.com/en-us/windows/client-management/administrative-templates-in-group-policy#configure-the-central-store

## 問題110
あなたのネットワークには contoso.com という Active Directory ドメインがあります。
このドメインにはグループ管理サービス アカウント (gMSA) が存在します。
Windows Server を実行しておりワークグループに参加している Server1 というサーバーがあります。
Server1 は Windows コンテナーをホストしています。

Windows コンテナーが contoso.com に対して認証できるようにする必要があります。

どの 3 つの操作を順番に実行するべきですか。
回答するには、次の操作の一覧から適切な操作を解答エリアに移動し、正しい順序に並べ替えてください。

選択肢：
- On Server1, install and run ccg.exe.
- On Server1, run New-CredentialSpec.
- In contoso.com, generate a Key Distribution Service (KDS) root key.
- In contoso.com, create a gMSA and a standard user account.
- From a domain-joined computer, create a credential spec file and copy the file to Server1.

## 解答110
1. In contoso.com, create a gMSA and a standard user account.
2. From a domain-joined computer, create a credential spec file and copy the file to Server1.
3. On Server1, install and run ccg.exe.

## 解説110
- ワークグループ ホストでコンテナーに gMSA を使わせるには、ドメイン側で gMSA を用意し、ドメイン参加済みのコンピューターで Credential Spec (JSON) を作成してホストへ配布し、ホスト側では Container Credential Guard (ccg.exe) を用いてその Credential Spec を利用させます。これにより、ホストがドメインに参加していなくてもコンテナーは Kerberos を用いたドメイン認証を行えます。
  - 公式: Manage service accounts for Windows containers（Credential Spec の作成とコンテナーでの使用、ワークグループ ホストでは Container Credential Guard を使用）
    https://learn.microsoft.com/virtualization/windowscontainers/manage-containers/manage-serviceaccounts
  - 公式: New-CredentialSpec（Credential Spec ファイルを作成する PowerShell コマンドレット）
    https://learn.microsoft.com/powershell/module/credentialspec/new-credentialspec
- 問題文では「ドメインには gMSA が存在」とあるため、KDS ルート キーは既にある前提で追加作成は不要です（gMSA の前提として KDS ルート キーが必要）。
  - 公式: Group Managed Service Accounts overview（gMSA と KDS ルート キーの要件）
    https://learn.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview
- 「On Server1, run New-CredentialSpec」はホストがドメイン未参加のため不適切です。Credential Spec の作成はドメインにアクセス可能なドメイン参加コンピューターで実施します。

## 問題111
あなたのネットワークには Active Directory Domain Services (AD DS) ドメインがあります。
ドメインには次のドメイン コントローラーが含まれます。
|Name|Description|
|---|---|
|DC1|PDC emulator, RID master, and global catalog server|
|DC2|Infrastructure master and domain naming master|
|DC3|Schema master|
|RODC1|Read-only domain controller (RODC)|

攻撃者が RODC1 のコンピューター アカウントを侵害した場合でも、Employee-Number という AD DS 属性を閲覧できないようにする必要があります。

どのパーティションを変更する必要がありますか？

A. configuration
B. global catalog
C. domain
D. schema

## 解答111
D. schema

## 解説111
- RODC に複製させたくない機微属性は「RODC フィルター済み属性セット (RODC filtered attribute set)」に追加します。これは属性スキーマの設定であり、スキーマ パーティション内の属性定義を変更して構成します。よって変更対象は schema パーティションです（実運用ではスキーマ マスターで実施）。
- グローバル カタログやドメイン/構成パーティションでは、特定属性を RODC へ複製しない設定は行いません。

根拠（Microsoft 公式ドキュメント）:
- RODC フィルター済み属性セットの概要（RODC へ複製しない機微属性をスキーマで指定）
  https://learn.microsoft.com/windows-server/identity/rodc/rodc-planning#rodc-filtered-attribute-set
- スキーマ パーティションは属性定義を保持し、変更はスキーマ マスターで行う
  https://learn.microsoft.com/windows-server/identity/ad-ds/plan/active-directory-logical-structure#schema-partition

## 問題112
SRV1 上で、mcr.microsoft.com/windows/servercore/iis イメージを使用するコンテナーを実行する必要があります。
コンテナーのポート 60 を SRV1 のポート 5001 に公開し、コンテナーはバックグラウンドで実行されていなければなりません。

## 解答112
docker run -d -p 5001:60 --name iis_container mcr.microsoft.com/windows/servercore/iis

## 解説112
- -d はコンテナーをバックグラウンド（detached）で実行します。
- -p 5001:60 は「ホスト(SRV1)側の 5001」から「コンテナー側の 60」へのポート公開（ポート マッピング）です。
- mcr.microsoft.com/windows/servercore/iis は Microsoft Container Registry 提供の IIS を含む Windows Server Core ベース イメージです。
- 必要に応じて事前にイメージを取得します: docker pull mcr.microsoft.com/windows/servercore/iis

根拠（Microsoft 公式ドキュメント）:
- Windows コンテナーのクイック スタート（IIS コンテナー実行例と -p によるポート公開）
  https://learn.microsoft.com/virtualization/windowscontainers/quick-start/run-your-first-container?tabs=Windows-Server
- Windows コンテナーのネットワーク（ポート公開/マッピングの概要）
  https://learn.microsoft.com/virtualization/windowscontainers/container-networking/
- Windows コンテナーのベース イメージ（Server Core と IIS イメージの解説）
  https://learn.microsoft.com/virtualization/windowscontainers/manage-containers/container-base-images

## 問題113
あなたは、Site-to-Site VPN を使用して Azure 仮想ネットワークに接続されているオンプレミス ネットワークを保有しています。
各ネットワークには、同一の IP アドレス空間を持つサブネットが含まれています。
オンプレミス側のサブネットには仮想マシンが 1 台あります。
この仮想マシンを Azure 側のサブネットへ移行する予定です。
IP アドレスを変更することなく、このオンプレミスの仮想マシンを Azure に移行する必要があります。
ソリューションは管理上の手間を最小限に抑えなければなりません。
移行を実施する前に、どれを実装すべきですか？
A. Azure Extended Network
B. Azure Virtual Network NAT
C. Azure Application Gateway
D. Azure 仮想ネットワーク ピアリング

## 解答113
A. Azure Extended Network

## 解説113
- Azure Extended Network は、オンプレミスのサブネット/アドレス空間を Azure に拡張し、ワークロード移行時に同じプライベート IP アドレスを維持できるため（＝再アドレス指定不要）、管理作業を最小化できます。参考: Azure Extended Network の概要（プレビュー）: https://learn.microsoft.com/azure/virtual-network/azure-extended-network-overview
- 他の選択肢が不適な理由（要点）:

|選択肢|理由（Microsoft Docs 根拠）|
|---|---|
|B. Azure Virtual Network NAT|NAT ゲートウェイは仮想ネットワークのアウトバウンド（送信）インターネット接続を提供する機能で、移行時の同一 IP 維持や重複アドレス空間の解決は目的外。https://learn.microsoft.com/azure/virtual-network/nat-gateway/nat-overview|
|C. Azure Application Gateway|アプリケーション レイヤー（L7）の Web トラフィック用ロード バランサーであり、ネットワークの重複解消や IP 維持のための機能ではない。https://learn.microsoft.com/azure/application-gateway/overview|
|D. Azure 仮想ネットワーク ピアリング|ピアリングする仮想ネットワーク間でアドレス空間の重複はサポートされない（重複アドレスは不可）。https://learn.microsoft.com/azure/virtual-network/virtual-network-peering-overview|

## 問題114
Windows Server を実行しているオンプレミス サーバーが 10 台あります。
これらのサーバーを Azure のリソースに接続するために、Azure Network Adapter を使用する予定です。
オンプレミス側と Azure 側で必要となる前提条件はどれですか？
回答するには、解答領域で適切な選択肢を選択してください。
注: 各正答は 1 ポイントの価値があります。

オンプレミス サーバーを構成するには、次のいずれかを使用します: 「Azure CLI」または「Routing and Remote Access」または「Server Manager」または「Windows Admin Center」
Azure リソースと Azure Network Server Manager Adapter を接続するには、次のいずれかを使用します: 「Azure Bastion」または「Azure Firewall」または「Azure 仮想ネットワーク ゲートウェイ (Virtual network gateway)」または「プライベート エンドポイント (Private endpoint)」または「パブリック Azure Load Balancer」

## 解答114
|設問|正答|
|---|---|
|オンプレミス サーバーを構成する際に使用するもの|Windows Admin Center|
|Azure リソースと Azure Network Server Manager Adapter を接続するために使用するもの|Azure 仮想ネットワーク ゲートウェイ|

## 解説114
- オンプレミス側: Azure Network Adapter は Windows Admin Center の機能で、オンプレミスの Windows Server から Azure 仮想ネットワークへのポイント対サイト (P2S) VPN 接続の作成を簡素化します。したがって構成には Windows Admin Center を使用します。（Microsoft Docs: Windows Admin Center の Azure Network Adapter）https://learn.microsoft.com/windows-server/manage/windows-admin-center/azure/azure-network-adapter
- Azure側: P2S VPN では、VNet に対して VPN ゲートウェイ（Virtual network gateway）が必要です。VPN ゲートウェイは、オンプレミスと Azure VNet 間で暗号化されたトラフィックを転送するために使用されます。（Microsoft Docs: Point-to-site について）https://learn.microsoft.com/azure/vpn-gateway/point-to-site-about （VPN ゲートウェイの概要）https://learn.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways
- なお、Azure Bastion はブラウザ経由のRDP/SSH、Azure Firewall はL3/L7のファイアウォール、プライベート エンドポイントはPaaS へのプライベート接続、パブリック Load Balancer は受信分散用途であり、P2S 接続の必須要件ではありません。

## 問題115
あなたのネットワークには、contoso.com という名前の Active Directory Domain Services (AD DS) ドメインが存在します。
ドメインには、次の表に示すユーザーが含まれています。
|Name|Located in|
|---|---|
|User1|Contoso\Users|
|User2|Contoso\OU1|
|User3|Contoso\OU1\OU2|

ドメインには、次の表に示すグループ ポリシー オブジェクト (GPO) が存在します。
|Name|Linked to|Enforcement|
|---|---|---|
|GPO1|Contoso.com|GPO リンクに対して Enforced (強制) が有効です。|
|GPO2|OU1|None|
|GPO3|OU2|OU2 で Block inheritance (継承のブロック) が有効です。|

これらの GPO は、次の表に示すとおり、H という名前のドライブをマップするように構成されています。
|Name|Configuration|
|---|---|
|GPO1|ドライブ H は \\server1\share にマップされます。|
|GPO2|ドライブ H は \\server2\share にマップされます。|
|GPO3|ドライブ H は \\server3\share にマップされます。|

次の各記述について、記述が正しければ Yes、正しくなければ No を選択してください。
注: 各正答は 1 ポイントの価値があります。

Answer Area
|Statements|Yes|No|
|---|---|---|
|For User1, \\server2\share maps to drive H.| | |
|For User2, \\server1\share maps to drive H.| | |
|For User3, \\server3\share maps to drive H.| | |

## 解答115
|Statements|Yes|No|
|---|---|---|
|For User1, \\server2\share maps to drive H.| |No|
|For User2, \\server1\share maps to drive H.|Yes| |
|For User3, \\server3\share maps to drive H.| |No|

## 解説115
- GPO は LSDOU（Local → Site → Domain → OU）の順に処理され、通常は後段が前段を上書きします。ただし、GPO リンクで Enforced (強制) を有効にすると、子コンテナーにリンクされた GPO による上書きができなくなり、さらに子 OU で Block inheritance (継承のブロック) を有効にしていても、その Enforced GPO は適用されます。
  - 参考: Microsoft Learn「グループ ポリシーの概要（処理順序、継承、Enforced/Block inheritance）」https://learn.microsoft.com/windows-server/identity/ad-ds/manage/understand-group-policy
- 本設問では、ドメインにリンクされた GPO1 が Enforced のため、OU 側の GPO2 や GPO3 でのドライブ マップ設定は上書きできません。結果として、User1/2/3 いずれも最終的に H ドライブは \\server1\share となります。よって設問の判定は、User1 の記述は No、User2 は Yes、User3 は No です。

## 問題116
あなたは sub1 という名前の Azure サブスクリプションと、Windows Server を実行するオンプレミスの仮想マシン 500 台を保有しています。
Azure Arc のデプロイ スクリプトを実行して、オンプレミスの仮想マシンを Azure Arc にオンボードする予定です。
スクリプトが sub1 へのアクセスを認証するために使用する ID を作成する必要があります。
ソリューションは最小特権の原則に従う必要があります。
コマンドをどのように完成させるべきですか？ 
回答するには、解答領域で適切な選択肢を選択してください。
注: 各正答は 1 ポイントの価値があります。

Answer Area
```
<「New-AzADAppCredential」or「New-AzADServicePrincipal」or「New-AzUserAssignedIdentity」> -DisplayName 'Arc-for-servers' -Role <「'Azure Connected Machine Onboarding'」or「'Virtual Machine Contributor'」or「Virtual Machine User Login'」>
```

## 解答116
|項目|正答|
|---|---|
|Cmdlet|New-AzADServicePrincipal|
|Role|'Azure Connected Machine Onboarding'|

完成コマンド:
```
New-AzADServicePrincipal -DisplayName 'Arc-for-servers' -Role 'Azure Connected Machine Onboarding'
```

## 解説116
- Azure Arc でサーバーをスクリプトで一括オンボードする場合、最小特権である組み込みロール「Azure Connected Machine Onboarding」を割り当てたサービス プリンシパルの利用が推奨されます。これにより Arc 対象リソースの登録に必要な最小権限のみを付与できます。
  - 参考: Microsoft Learn「Azure の組み込みロール」(Azure Connected Machine Onboarding) https://learn.microsoft.com/azure/role-based-access-control/built-in-roles#azure-connected-machine-onboarding
  - 参考: Microsoft Learn「Azure Arc 対応サーバー: サービス プリンシパルでのオンボード (スケール)」(サービス プリンシパルと最小権限の推奨) https://learn.microsoft.com/azure/azure-arc/servers/onboard-service-principal
- New-AzADServicePrincipal はサービス プリンシパルを作成し、-Role で RBAC ロールを割り当て可能です。
  - 参考: Microsoft Learn「New-AzADServicePrincipal」https://learn.microsoft.com/powershell/module/az.resources/new-azadserviceprincipal
- 誤りの選択肢の要点:
  - New-AzADAppCredential は既存アプリへの資格情報追加であり、ID作成や RBAC 付与の主用途ではありません。https://learn.microsoft.com/powershell/module/az.resources/new-azadappcredential
  - New-AzUserAssignedIdentity はマネージド ID（Azure リソース専用）で、オンプレミスのスクリプト実行からの認証には使用できません。https://learn.microsoft.com/entra/identity/managed-identities-azure-resources/overview

## 問題117
あなたのネットワークには、contoso.com という名前の Active Directory Domain Services (AD DS) ドメインが存在します。
ドメインには、Server1 と Server2 という名前の 2 台のサーバーが含まれています。
Server1 には Disk2 という名前のディスクがあり、Disk2 には UserData という名前のフォルダーが含まれています。
UserData は Domain Users グループに共有されています。Disk2 は重複除去 (Data Deduplication) に構成されています。
Server1 は Azure Backup によって保護されています。
Server1 が障害で停止しました。
あなたは Disk2 を Server2 に接続しました。
できるだけ早く Disk2 上のすべてのファイルにアクセスできるようにする必要があります。
何をすべきですか？
A. 記憶域プールを作成する。
B. Azure Backup からファイルを復元する。
C. File Server Resource Manager (FSRM) サーバー役割をインストールする。
D. Data Deduplication サーバー役割をインストールする。

## 解答117
D. Data Deduplication サーバー役割をインストールする。

## 解説117
- Disk2 は重複除去で最適化されているため、別サーバーで内容にアクセスするには、そのサーバー側に Data Deduplication の役割（フィルター ドライバー）をインストールしておく必要があります。役割がないと、最適化済みファイルは透過的に読み出せません。参照: Microsoft Learn「Data Deduplication の概要」https://learn.microsoft.com/windows-server/storage/data-deduplication/overview
- 最短でアクセスを復旧する手段は、Server2 に Data Deduplication をインストールすることです。Azure Backup からの復元は可能ですが時間を要し、「できるだけ早く」という要件に適しません。参照: Microsoft Learn「Data Deduplication のインストールと有効化」https://learn.microsoft.com/windows-server/storage/data-deduplication/install-enable

## 問題118
あなたは Windows コンテナーをホストする Server1 という名前のサーバーを保有しています。
複数のコンテナーで構成されるアプリケーションをデプロイする予定です。
各コンテナーは物理（オンプレミス）ネットワーク上の固有の IP アドレスを持ち、外部から直接到達可能である必要があります。
アプリケーションのデプロイをサポートする Docker ネットワークを作成する必要があります。どの種類のネットワークを作成すべきですか？
A. transparent
B. L2bridge
C. NAT
D. L2tunnel

## 解答118
A. transparent

## 解説118
- Transparent ネットワークは、コンテナーを基盤となる物理ネットワークに直接接続し、各コンテナーに外部ネットワーク上の IP を割り当て可能です。これにより、外部からコンテナーへ直接到達（ポート マッピング不要）が実現します。
  - 参考: Microsoft Learn「Windows コンテナーのネットワーク アーキテクチャ（NAT/Transparent/L2bridge/L2tunnel）」https://learn.microsoft.com/virtualization/windowscontainers/container-networking/architecture
  - 参考: Microsoft Learn「Windows コンテナーのネットワーク管理（Transparent: 物理ネットワークへ直接接続）」https://learn.microsoft.com/virtualization/windowscontainers/manage-containers/container-networking
- 誤りの選択肢の要点:
  - NAT: 既定ドライバー。外部到達にはポート公開が必要で、各コンテナーを物理ネットワーク上で直接到達可能にはしない。
  - L2bridge/L2tunnel: 特定のオーケストレーションやネットワーク設計で用いられるが、本設問の「各コンテナーが外部から直接到達可能」という要件には transparent が最も適合する。

## 問題119
あなたのネットワークには Active Directory Domain Services (AD DS) フォレストがあります。
フォレストには Site1、Site2、Site3 という名前の 3 つの Active Directory サイトが含まれています。
各サイトには 2 台のドメイン コントローラーが含まれています。
サイトは DEFAULTIPSITELINK を使用して接続されています。
新しい支社オフィスを開設しました。このオフィスにはクライアント コンピューターのみが含まれます。
新しいオフィス内のクライアント コンピューターが、主として Site1 のドメイン コントローラーによって認証されるようにする必要があります。
解決策: Site1 に関連付けられた新しいサブネット オブジェクトを作成します。
これは目標を満たしますか?
A. No
B. Yes

## 解答119
B. Yes

## 解説119
- クライアントは自分の IP アドレスが Active Directory のサブネット オブジェクトにどのようにマップされているかで「所属サイト」を判定し、そのサイト内のドメイン コントローラーを優先的に使用します。したがって、新しい支社の IP サブネットを表すサブネット オブジェクトを作成し、それを Site1 に関連付ければ、当該クライアントは Site1 所属と見なされ、Site1 の DC によって主として認証されます。
  - 参考: Microsoft Learn「ドメイン コントローラーが検出される方法 (DC Locator)」https://learn.microsoft.com/troubleshoot/windows-server/identity/how-domain-controllers-are-located
  - 参考: Microsoft Learn「AD DS サイトとサービスの概要 (サイト/サブネットの役割)」https://learn.microsoft.com/windows-server/identity/ad-ds/plan/understanding-active-directory-sites
  - 参考: Microsoft Learn「New-ADReplicationSubnet (サブネット オブジェクトの作成とサイトへの関連付け)」https://learn.microsoft.com/powershell/module/addsadministration/new-adreplicationsubnet

## 問題120
あなたは、オンプレミスの Active Directory Domain Services (AD DS) ドメインを所有しており、これが Microsoft Entra テナントと同期しています。
ドメインにカスタム属性を追加するアプリを展開しました。
Azure Cloud Shell から、ユーザーのカスタム属性をクエリできないことに気付きました。
カスタム属性が Microsoft Entra ID で利用できるようにする必要があります。
最初に Microsoft Entra Connect から実行すべき作業はどれですか？

A. 同期のオプションをカスタマイズする  
B. デバイスのオプションを構成する  
C. ディレクトリ スキーマを更新する（Refresh directory schema）  
D. フェデレーションを管理する

## 解答120
C. ディレクトリ スキーマを更新する（Refresh directory schema）

## 解説120
- オンプレミス AD DS に新しい（カスタム）属性を追加した直後は、Microsoft Entra Connect 側のスキーマにそれが読み込まれていないため、同期対象として選択できず、結果として Microsoft Entra ID（Azure Cloud Shell/Graph）でその属性を参照できません。最初に「Refresh directory schema（ディレクトリ スキーマの更新）」を実行して新属性を取り込み、その後に Directory extensions で同期対象の属性として選択します。
- 公式ドキュメントの根拠:
  - Microsoft Entra Connect Sync: Directory extensions では、Active Directory に新しい属性を作成したのに一覧に表示されない場合は「Refresh directory schema」を実行して属性リストを更新してから選択するよう案内されています。  
    https://learn.microsoft.com/entra/identity/hybrid/connect/how-to-connect-sync-feature-directory-extensions
  - 既定で同期される属性は限定されており、追加の属性を同期するには Directory extensions を使用する必要がある旨が記載されています。  
    https://learn.microsoft.com/entra/identity/hybrid/connect/reference-connect-sync-attributes-synchronized

## 問題121
あなたのネットワークには Active Directory Domain Services (AD DS) ドメインがあります。
ドメインには、次のサーバーが含まれます。
|Name|Server role|DHCP scope|
|---|---|---|
|Server1|DHCP Server|Scope1, Scope2|
|Server2|DHCP Server|Scope3, Scope4|

User1 というユーザーがいます。
User1 が Scope1 と Scope3 のみを管理できるようにする必要があります。
どうすべきですか？
A. Server1 と Server2 の DHCP Administrators グループに User1 を追加する。
B. IP Address Management (IPAM) を実装する。
C. Windows Admin Center を実装し、Server1 と Server2 への接続を追加する。
D. ドメイン ローカル グループの DHCP Administrators に User1 を追加する。

## 解答121
B. Implement IP Address Management (IPAM).

## 解説121
- DHCP サーバーの「DHCP Administrators」グループ（ローカル/ドメイン）に追加すると、そのサーバー上のすべてのスコープを管理できるようになります。スコープ単位のきめ細かな委任はできません（A, D は不適）。
- Windows Admin Center は管理ポータルであり、基盤の権限モデルを置き換えないため、スコープ単位の RBAC は提供しません（C は不適）。
- IPAM は DHCP/DNS を集中管理でき、RBAC と「アクセス スコープ」によるきめ細かな委任を提供します。これにより、User1 に Scope1 と Scope3 のみの管理権限を割り当てることが可能です。

根拠（Microsoft 公式ドキュメント）:
- IPAM の概要と RBAC/アクセス スコープによる委任: https://learn.microsoft.com/windows-server/networking/technologies/ipam/ipam-top
- IPAM のロールベース アクセス制御（特定の DHCP スコープやサーバーに対するアクセスの制限）: https://learn.microsoft.com/windows-server/networking/technologies/ipam/ipam-role-based-access-control
- DHCP サーバーのセキュリティ グループ（DHCP Administrators はサーバー全体の管理権限を付与）: https://learn.microsoft.com/windows-server/networking/technologies/dhcp/dhcp-security-groups

## 問題122
あなたは、Windows Server を実行し Hyper-V サーバーの役割がインストールされている 
Server1 という名前のサーバーを所有しています。
Server1 は VM1 という名前の仮想マシンをホストしています。
Server1 には NVMe ストレージ デバイスがあります。
このデバイスは現在、離散デバイス割り当て (Discrete Device Assignment; DDA) を使用して VM1 に割り当てられています。
あなたはこのデバイスを Server1 で利用できるようにする必要があります。
順序通りに実行すべき 4 つの操作はどれですか？
回答するには、次の操作の一覧から適切な操作を解答領域に移動し、正しい順序に並べ替えてください。

選択肢:
- Server1 から、VM1 を停止する。
- Server1 から、Remove-VMAssignableDevice コマンドレットを実行する。
- Server1 から、Mount-VMHostAssignableDevice コマンドレットを実行する。
- Server1 から、デバイス マネージャーを使用してデバイスを有効にする。
- VM1 から、デバイス マネージャーを使用してデバイスを無効にする。

## 解答122
1. VM1 から、デバイス マネージャーを使用してデバイスを無効にする。
2. Server1 から、VM1 を停止する。
3. Server1 から、Remove-VMAssignableDevice コマンドレットを実行する。
4. Server1 から、Mount-VMHostAssignableDevice コマンドレットを実行する。

## 解説122
- DDA により VM に割り当てたデバイスをホストへ戻す基本手順は、(1) ゲスト OS 側でデバイスを無効化 → (2) VM を停止 → (3) Remove-VMAssignableDevice で割り当て解除 → (4) Mount-VMHostAssignableDevice でホストへ再マウントの順です。DDA のデバイス追加/削除は VM の電源オフが前提です。
- 公式ドキュメントの根拠:
  - Discrete Device Assignment の概要と手順（VM の電源をオフにして追加/削除を行う旨を含む）: https://learn.microsoft.com/windows-server/virtualization/hyper-v/deploy/discrete-device-assignment
  - ゲストからホストへデバイスを戻す代表的な手順（Disable → Shutdown → Remove-VMAssignableDevice → Mount-VMHostAssignableDevice）: https://learn.microsoft.com/windows-server/virtualization/hyper-v/deploy/deploying-graphics-devices-using-dda
  - Remove-VMAssignableDevice コマンドレット: https://learn.microsoft.com/powershell/module/hyper-v/remove-vmassignabledevice
  - Mount-VMHostAssignableDevice コマンドレット: https://learn.microsoft.com/powershell/module/hyper-v/mount-vmhostassignabledevice

## 問題123
Server1 という名前のサーバーがあります。
Storage Spaces を使用して Server1 の利用可能なストレージを拡張する計画です。
Server1 には 8 台の物理ディスクを接続しました。4 台は HDD、4 台は SSD です。
新しいすべてのディスク上のストレージを使用するボリュームを Server1 上に作成する必要があります。
さらに、よく使用されるファイルの読み取り性能を最速にする必要があります。
次の操作から、実行する 3 つの操作を正しい順序で選択してください。
選択肢:
- Create a virtual disk.
- Convert each new disk into a dynamic disk.
- Create a storage pool.
- Create a spanned volume.
- Convert each new disk into a GPT disk.
- Create a simple volume

## 解答123
1. Create a storage pool.
2. Create a virtual disk.
3. Create a simple volume.

## 解説123
- Storage Spaces では「物理ディスク → ストレージ プール → 仮想ディスク → ボリューム」という順で構成します。まず HDD と SSD を同一プールにまとめ、次に仮想ディスクを作成します。
- 仮想ディスク作成時にストレージ階層 (Storage Tiers: SSD と HDD) を有効化することで、頻繁にアクセスされるデータが自動的に SSD 階層に配置され、読み取り性能が向上します。最後に仮想ディスク上にボリューム（Simple Volume）を作成してフォーマットします。
- 動的ディスクやスパン ボリュームは Storage Spaces を用いない旧来の方式で、本要件（全ディスクの活用とホット データの高速化）を満たしません。

根拠（Microsoft 公式ドキュメント）:
- Storage Spaces の概要（ストレージ プール／仮想ディスク、ストレージ階層により頻繁にアクセスされるデータを高速メディアへ移動）
  https://learn.microsoft.com/windows-server/storage/storage-spaces/overview
- スタンドアロン サーバーでの Storage Spaces の展開手順（プール作成 → 仮想ディスク作成 → ボリューム作成）
  https://learn.microsoft.com/windows-server/storage/storage-spaces/deploy-standalone-storage-spaces

## 問題124
あなたは、Windows Server を実行し、Hyper-V サーバーの役割がインストールされている Served という名前のサーバーを所有しています。
あなたは Just Enough Administration (JEA) のロール機能 (role capabilities) とセッション構成ファイルを作成しました。
ヘルプデスク ユーザーが Server1 をリモートで管理する際に使用できる Hyper-V モジュールのコマンドレットを制限する必要があります。
PowerShell コマンドをどのように完成させるべきですか？
回答するには、解答エリアで適切なオプションを選択してください。
注: 各正しい選択は 1 点の価値があります。

<「Enter-PSSession」or「New-PSSessionConfigurationFile」or「Register-PSSessionConfiguration」> -Path .\HyperVJeaConfig <「.ps1」or「.psm1」or「.psrc」or「.pssc」> -Name 'HyperVJeaHelpDesk' -Force

## 解答124
Register-PSSessionConfiguration -Path .\HyperVJeaConfig.pssc -Name 'HyperVJeaHelpDesk' -Force

## 解説124
- JEA のエンドポイントは、セッション構成ファイル (.pssc) を Register-PSSessionConfiguration で登録して作成します。これにより、接続ユーザーは JEA で定義された制限付きの実行環境に入ります。
  - 参考: Register-PSSessionConfiguration（セッション構成の登録）
    https://learn.microsoft.com/powershell/module/microsoft.powershell.core/register-pssessionconfiguration
  - 参考: about_Session_Configuration_Files（.pssc の定義と登録）
    https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_Session_Configuration_Files
- どのコマンドレットを使えるかの制御はロール機能ファイル (.psrc) の VisibleCmdlets 等で行い、.pssc の RoleDefinitions でユーザー/グループに割り当てます。Hyper-V モジュールの許可コマンドをロール機能に定義し、登録した JEA エンドポイントに接続させることで、ヘルプデスクの使用可能コマンドレットを制限できます。
  - 参考: about_Role_Capabilities（.psrc と VisibleCmdlets）
    https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_Role_Capabilities
  - 参考: JEA の概要
    https://learn.microsoft.com/windows-server/management/jea/jea-overview

## 問題125
あなたは Windows Server を実行する Server1 という名前のサーバーを所有しており、Server1 には just-a-bunch-of-disks (JBOD) エンクロージャーが接続されています。
あなたは Server1 上に記憶域プールを作成し、ミラー レイアウトを使用する仮想ディスクを作成する予定です。
現在、2 方向ミラー (two-way mirror) と 3 方向ミラー (three-way mirror) のどちらを使用するか検討しています。
各ミラー レイアウトに必要な最小ディスク数はどれですか？
回答するには、解答エリアで適切なオプションを選択してください。
注: 各正答は 1 点の価値があります。

Answer Area
Two-way mirror: 1 or 2 or 3 or 4 or 5 or 6
Three-way mirror: 1 or 2 or 3 or 4 or 5 or 6

## 解答125
Two-way mirror: 2
Three-way mirror: 5

## 解説125
- Windows Server の Storage Spaces におけるミラーの最小ディスク要件は、Two-way mirror は最小 2 台、Three-way mirror は最小 5 台です。Three-way mirror は 2 台の同時ディスク障害に耐えられる設計であり、そのために 5 台以上が必要となります。
- 公式ドキュメントの根拠:
  - Storage Spaces の概要（Resiliency types の表に最小ディスク本数が記載）: https://learn.microsoft.com/windows-server/storage/storage-spaces/overview#resiliency-types
  - 設計/展開ガイダンス（ミラーの特性と耐障害性の説明）: https://learn.microsoft.com/windows-server/storage/storage-spaces/understand-storage-spaces

## 問題126
あなたは、単一のディスクを搭載した Windows Server コンテナー ホストである Server1 を所有しています。
Server1 上で、次の表に示すコンテナーを起動する予定です。

|名前|説明|
|---|---|
|Container1|Container1 は開発中の Web アプリを含む Windows コンテナーです。このコンテナーは他のコンテナーとカーネルを共有してはなりません。|
|Container2|Container2 は Web アプリを実行する Linux コンテナーです。このコンテナーには 2 つの静的 IP アドレスが必要です。|
|Container3|Container3 はデータベースを実行する Windows コンテナーです。このコンテナーには静的 IP アドレスが必要です。|

各コンテナーで使用できる分離モードはどれですか？
回答するには、解答エリアで適切なオプションを選択してください。
注: 各正答は 1 点の価値があります。

Answer Area
Container1:「Hyper-V isolation only」or「Process isolation only」or「Hyper-V isolation or process isolation」
Container2:「Hyper-V isolation only」or「Process isolation only」or「Hyper-V isolation or process isolation」
Container3:「Hyper-V isolation only」or「Process isolation only」or「Hyper-V isolation or process isolation」

## 解答126
- Container1: Hyper-V isolation only
- Container2: Hyper-V isolation only
- Container3: Hyper-V isolation or process isolation

## 解説126
- Windows コンテナーの分離モードには「プロセス分離（Windows Server コンテナー）」と「Hyper-V 分離」があり、プロセス分離はホストとカーネルを共有する一方、Hyper-V 分離は各コンテナーが軽量 VM 内で独自のカーネルを持ちます。よって「他のコンテナーとカーネルを共有しない」要件の Container1 は Hyper-V 分離が必須です。
  - 参考: Windows コンテナーの概要（プロセス分離と Hyper-V 分離）
    https://learn.microsoft.com/virtualization/windowscontainers/about/
- Linux コンテナーは Windows のカーネルを共有できないため、Windows 上で実行する場合は Hyper-V 分離（ユーティリティ VM）などにより Linux カーネルを提供する必要があります。従って Container2 は Hyper-V 分離のみ利用可能です。
  - 参考: Windows 上での Linux コンテナーの実行（Linux コンテナーは Hyper-V 分離/ユーティリティ VM を使用）
    https://learn.microsoft.com/virtualization/windowscontainers/deploy-containers/linux-containers
- 静的 IP の要件はネットワーク ドライバー（例: Transparent ネットワーク）で満たすもので、分離モード（プロセス/Hyper-V）自体の可否を左右しません。よって Windows コンテナーの Container3 はどちらの分離モードでも構いません。
  - 参考: Windows コンテナー ネットワーキング（Transparent などのネットワーク ドライバーと静的 IP）
    https://learn.microsoft.com/virtualization/windowscontainers/container-networking/

## 問題127
オンプレミス ネットワークには単一ドメインの Active Directory Domain Services (AD DS) フォレストがあり、Azure AD テナント contoso.com と Azure AD Connect で同期しています。
フォレスト内のユーザーのうち、カスタム属性に NoSync を持つユーザーを同期対象から除外する必要があります。
Azure AD Connect の cloudFiltered 属性はどのように設定し、どのツールを使用すべきですか。
回答エリアで適切なオプションを選択してください。
注意: 各正解は 1 ポイントの価値があります。
Answer Area:
Attribute:「False」or「Null」or「True」
Tool:「ADSI Edit」or「Synchronization Rules Editor」or「The Microsoft Azure AD Connect wizard」

## 解答127
- Attribute: True
- Tool: Synchronization Rules Editor

## 解説127
- Azure AD Connect では、オブジェクトを Azure AD へ送らないようにする一般的な方法は、cloudFiltered を True に設定する同期ルールを作成することです。今回の要件では「カスタム属性 = NoSync」というスコープ条件で一致したユーザーに対して、Inbound ルールで cloudFiltered を True に変換する同期ルールを追加します（既定ルールより高い優先度）。
- この種の属性ベース フィルタリングは Azure AD Connect の Synchronization Rules Editor を用いて構成します。ADSI Edit はスキーマ/オブジェクトを直接編集するツールで目的外、ウィザードは OU/ドメイン/グループ単位のフィルタリング中心で、任意属性の条件で cloudFiltered を設定することはできません。

根拠（Microsoft 公式ドキュメント）:
- Azure AD Connect のフィルタリング（属性ベース フィルタリングは Sync Rules Editor で構成）:
  https://learn.microsoft.com/azure/active-directory/hybrid/how-to-connect-sync-configure-filtering#filtering-based-on-attributes
- Azure AD Connect 同期のカスタマイズ（宣言型プロビジョニングと cloudFiltered の概念）:
  https://learn.microsoft.com/azure/active-directory/hybrid/concept-azure-ad-connect-sync-declarative-provisioning
- Synchronization Rules Editor の使用:
  https://learn.microsoft.com/azure/active-directory/hybrid/how-to-connect-sync-change-the-configuration

## 問題128
あなたは Windows Server を実行し、DHCP Server の役割がインストールされているサーバーを所有しています。
サーバーには Scope1 という名前のスコープがあり、次の構成があります:
* アドレス範囲: 192.168.0.2 から 192.168.1.200。マスク 255.255.254.0
* ルーター: 192.168.0.1
* リース期間: 3 日
* DNS サーバー: 172.16.0.254

同一ベンダーの Microsoft Teams Phone デバイスを 50 台所有しています。すべてのデバイスの MAC アドレスは同一の範囲内にあります。

Scope1 からリースを受け取るすべての Teams Phone デバイスが、192.168.1.100 から 192.168.1.200 の範囲の IP アドレスを取得できるようにする必要があります。
解決策は、Scope1 から IP 構成を受け取る他の DHCP クライアントに影響を与えてはなりません。

何を作成する必要がありますか?
A. ポリシー
B. スコープ
C. フィッター
D. スコープ オプション

## 解答128
A. ポリシー

## 解説128
- 要件は「同一ベンダーの端末群（MAC アドレス範囲が共通）だけに、同一スコープ内の特定アドレス範囲（192.168.1.100–192.168.1.200）を割り当て、他クライアントへ影響しない」ことです。これはスコープ レベルの DHCP ポリシー（ポリシーベース割り当て）で実現できます。MAC アドレスやベンダー クラス等の条件で一致したクライアントに対し、ポリシー専用の IP アドレス範囲を指定して割り当てられます。
- 公式ドキュメントでは、DHCP ポリシーにより条件に一致したクライアントへ異なるオプションや「ポリシー専用の IP アドレス範囲」を適用できると説明されています（DHCP Policies Overview）。また、フィルターはリースの許可/拒否を制御する機能であり、特定サブレンジからの割り当て指定には使いません。
  - DHCP Policies Overview: https://learn.microsoft.com/en-us/windows-server/networking/technologies/dhcp/dhcp-policies/dhcp-policies-overview
  - Add or Remove Filters: https://learn.microsoft.com/en-us/windows-server/networking/technologies/dhcp/dhcp-add-remove-filter

|選択肢|概要|要件への適合|
|---|---|---|
|ポリシー|条件一致クライアントにポリシー専用の IP 範囲やオプションを適用|◯|
|フィルター|MAC 等で許可/拒否のみ（割り当て範囲の制御は不可）|×|
|スコープ オプション|オプション値の配布（アドレス範囲の選別は不可）|×|
|スコープ|別スコープを作ると他クライアントや設計に影響（要件に反する）|×|

## 問題129
Windows Server を実行する、Server1 という名前のオンプレミス サーバーがあります。
Server1 には、App1 というアプリと、Firewall1 というファイアウォールがあります。

Azure サブスクリプションがあります。

社内ユーザーは WebSockets を使用して App1 に接続しています。

インターネット上のユーザーが App1 を利用できるようにする必要があります。
解決策では、Firewall 1 で開く受信ポートの数を最小限に抑えなければなりません。

解決策には何を含めるべきですか?
A. Web Application Proxy
B. Azure Application Gateway
C. Azure Relay
D. Microsoft Application Request Routing (ARR) Version 2

## 解答129
C. Azure Relay

## 解説129
- Azure Relay（特に Hybrid Connections）は、オンプレミス内のサービスをパブリックに安全に公開でき、社内側では受信ポートを開放せずに済みます（アウトバウンド 443 のみ）。したがって「Firewall 1 で開く受信ポート数を最小化」という要件に最も適合します。
  - 公式ドキュメント: Azure Relay の概要（"ファイアウォールでポートを開けなくても、企業ネットワーク上のサービスを安全に公開"）https://learn.microsoft.com/en-us/azure/azure-relay/relay-what-is-azure-relay
- Hybrid Connections は WebSockets ベースの接続をサポートしており、WebSockets を使用して App1 に接続する要件を満たします。
  - 公式ドキュメント: Hybrid Connections の概要（"HTTP(S) と WebSockets に基づく接続"）https://learn.microsoft.com/en-us/azure/azure-relay/relay-hybrid-connections-overview
- 他の選択肢（WAP、Application Gateway、ARR）はいずれも、インターネットからの受信経路（ポート開放）を前提としたリバース プロキシ/ゲートウェイ構成であり、受信ポート最小化という要件に劣ります。

|選択肢|要点|
|---|---|
|Azure Relay|受信ポート開放不要（送信 443 のみ）、WebSockets 対応|
|WAP/ARR/App Gateway|公開のための受信ポート開放が前提、要件に不利|

## 問題130
あなたのネットワークには contoso.com という名前の Active Directory Domain Services (AD DS) ドメインがあります。
ドメインには、次の表に示す VPN サーバーが含まれています。

|Name|IP address|
|---|---|
|VPN1|172.16.0.254|
|VPN2|131.10.15.254|
|VPN3|10.10.0.254|

NPS (Network Policy Server) がインストールされた NPS1 という名前のサーバーがあります。NPS1 には次の RADIUS クライアントが構成されています: 
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/b6141443-2035-4140-9511-8bbc7c9034b8.png)


VPN1、VPN2、および VPN3 は RADIUS 認証に NPS1 を使用します。
contoso.com のすべてのユーザーは VPN 接続を確立することが許可されています。
次の各記述について、記述が正しければ Yes を、正しくなければ No を選択してください。
注: 各正答は 1 点の価値があります。

Answer Area
|Statements|Yes|No|
|---|---|---|
|The contoso.com users can authenticate successfully when they establish a VPN connection to VPN1.| | |
|The contoso.com users can authenticate successfully when they establish a VPN connection to VPN2.| | |
|The contoso.com users can authenticate successfully when they establish a VPN connection to VPN3.| | |

## 解答130
|Statements|Yes|No|
|---|---|---|
|The contoso.com users can authenticate successfully when they establish a VPN connection to VPN1.| |No|
|The contoso.com users can authenticate successfully when they establish a VPN connection to VPN2.|Yes| |
|The contoso.com users can authenticate successfully when they establish a VPN connection to VPN3.| |No|

## 解説130
- NPS を RADIUS サーバーとして使用する場合、各ネットワーク アクセス サーバー (NAS: ここでは各 VPN サーバー) は NPS 上で “RADIUS クライアント”として正しい IP アドレスまたは FQDN と共有シークレットで登録され、かつ有効化されている必要があります。登録されていない、または無効化されているクライアントからの要求は処理されません。
  - 根拠: 「RADIUS クライアントの追加/構成 (NPS)」には、NPS で RADIUS クライアントを作成し、IP/FQDN を指定して有効化する必要がある旨が記載されています。https://learn.microsoft.com/windows-server/networking/technologies/nps/nps-radius-clients-configure および https://learn.microsoft.com/windows-server/networking/technologies/nps/nps-radius-clients
- 画像より、172.16.0.254 (VPN1) は RADIUS クライアントとして構成されているものの Enabled=False のため、NPS は要求を受け付けず認証は失敗 (No)。
- 131.10.15.254 (VPN2) は Enabled=True の RADIUS クライアントとして登録されているため、ポリシーで許可されているユーザーは認証に成功 (Yes)。
- 10.10.0.254 (VPN3) は NPS 上の RADIUS クライアント一覧に一致がなく、未登録扱いとなるため要求は破棄され認証は失敗 (No)。

## 問題131
Azure サブスクリプションがあります。
サブスクリプションには、Windows Server を実行する VM1 という名前の仮想マシンが含まれています。
VM1 には 128 GB のオペレーティング システム ディスクが含まれています。
VM1 上のボリューム C のサイズを 250 GB に増やす必要があります。
順序どおりに実行する必要がある 4 つの操作はどれですか。
回答するには、操作の一覧から適切な操作を回答領域に移動し、正しい順序に並べ替えてください。

回答領域（操作の一覧）
- Resize VM1.
- Redeploy VM1.
- Stop VM1.
- Resize the operating system disk.
- Start VM1.
- Resize volume C.

## 解答131
| 手順 | 操作 |
|---|---|
| 1 | VM1 を停止する。 |
| 2 | オペレーティング システム ディスクのサイズを変更する。 |
| 3 | VM1 を起動する。 |
| 4 | ボリューム C のサイズを変更する。 |

## 解説131
- Azure の Windows VM で OS ディスクを拡張して C ドライブを大きくする一般的な流れは、VM の停止（割り当て解除）→ OS ディスクのサイズ変更 → VM の起動 → ゲスト OS 内で C ボリュームを拡張、の順です。VM のサイズ変更（コンピュート SKU 変更）や再デプロイは不要です。

根拠（Microsoft 公式ドキュメント）
- Windows VM の OS ディスク拡張（停止→OS ディスクのサイズ変更→起動→OS 内で拡張）: https://learn.microsoft.com/azure/virtual-machines/windows/expand-os-disk?tabs=azure-portal
- Windows のディスクの管理でボリュームを拡張（C ドライブを OS 内で拡張）: https://learn.microsoft.com/windows-server/storage/disk-management/extend-a-basic-volume

## 問題132
あなたは Azure サブスクリプションと、Windows 11 を実行する Computer1 という名前のコンピューターを所有しています。
Azure ポータルから、Windows Server を実行する VM1 という名前の仮想マシンをデプロイしました。
VM1 は既定の設定を使用するように構成されています。
VM1 に対して PowerShell リモーティングで接続できるようにする必要があります。
どのコマンドレットを実行し、またそのコマンドレットをどこから実行すべきですか。
回答するには、解答エリアで適切なオプションを選択してください。
注意: 各正解は 1 ポイントの価値があります。
Answer Area
Run from:「A PowerShell session on VM1」or「A PowerShell session on Computer1」or「Azure Cloud Shell」
Cmdlet:「Enable-AzVMPSRemoting」or「Enable-PSRemoting -Force」or「Enable-PSSessionConfiguration」

## 解答132
- Run from: Azure Cloud Shell
- Cmdlet: Enable-AzVMPSRemoting

## 解説132
- Enable-AzVMPSRemoting は対象の Azure VM で PowerShell リモーティング（WinRM）を有効化し、必要な Windows ファイアウォール設定とネットワーク セキュリティ グループ(NSG) ルールを自動構成します。既定設定で作成した VM に対しても、追加作業なく接続可能になります。
- Azure Cloud Shell には Az PowerShell モジュールが事前インストールされており、直ちに上記コマンドレットを実行できます。
- Enable-PSRemoting -Force はローカル マシン上で WinRM を有効化するだけで、NSG など Azure 側の設定は行いません。Enable-PSSessionConfiguration はセッション構成管理用で本要件には該当しません。

根拠（Microsoft 公式ドキュメント）:
- Enable-AzVMPSRemoting（Azure VM への PowerShell リモーティング有効化）: https://learn.microsoft.com/powershell/module/az.compute/enable-azvmpsremoting
- Azure Cloud Shell の概要（Az モジュール利用可）: https://learn.microsoft.com/azure/cloud-shell/overview
- Enable-PSRemoting（ローカルでの WinRM 有効化）: https://learn.microsoft.com/powershell/module/microsoft.powershell.core/enable-psremoting

## 問題133
次の表に示すストレージ アカウントを含む Azure サブスクリプションがあります。
|名前|場所|ファイル共有|BLOB コンテナー|
|---|---|---|---|
|storage1|West US|share1|なし|
|storage2|West US|share2|container2|
|storage3|Central US|share3|なし|
|storage4|Central US|share4|container4|

West US の Azure リージョンで、SyncA という名前の Storage Sync Service を作成しました。
GroupA という名前の同期グループ（sync group）を作成する予定です。
GroupA で使用できるクラウド エンドポイントの最大数は何ですか?
A. 4
B. 2
C. 1
D. 3

## 解答133
C. 1

## 解説133
- Azure File Sync の同期グループは「1 つのクラウド エンドポイント（Azure ファイル共有）と、1 つ以上のサーバー エンドポイント」で構成されます。したがって、同期グループで利用できるクラウド エンドポイントは最大 1 つです。
  - 根拠（公式ドキュメント）: "A sync group contains one cloud endpoint and one or more server endpoints." https://learn.microsoft.com/en-us/azure/storage/files/storage-sync-files-planning
- 参考: クラウド エンドポイントに使う Azure ファイル共有は、Storage Sync Service と同一リージョンである必要があります（本例では West US）。
  - 根拠（公式ドキュメント）: "The storage account containing the Azure file share for the cloud endpoint must be in the same region as the Storage Sync Service." https://learn.microsoft.com/en-us/azure/storage/files/storage-sync-files-deployment-guide

## 問題134
あなたのネットワークには、contoso.com と fabrikam.com という 2 つの Active Directory Domain Services (AD DS) フォレストが含まれています。
フォレスト間には双方向のフォレスト信頼があります。
各フォレストには 1 つのドメインが含まれます。
ドメインには、次の表に示すサーバーが含まれています。
|Name|Domain|Description|
|---|---|---|
|Server1|contoso.com|Windows Admin Center ゲートウェイをホストする|
|Server2|fabrikam.com|Server1 上の Windows Admin Center を使用してリモートで管理されるリソースをホストする|

contoso.com のユーザーが Server1 上の Windows Admin Center を使用して Server2 に接続できるように、リソースベースの制約付き委任 (resource-based constrained delegation; RBCD) を構成する必要があります。
コマンドをどのように完成させるべきですか。
回答するには、回答エリアで適切なオプションを選択してください。
注: 各正解は 1 ポイントの価値があります。

Answer Area
```
Set-ADComputer -Identity <「Get-ADComputer server1.contoso.com」or「Get-ADComputer server2.fabrikam.com」or「Get-ADGroup 'Contoso\Domain Users'」or「Get-ADGroup 'Fabrikam\Domain Users'」> -PrincipalsAllowedToDelegateToAccount
```

## 解答134
次のように指定します（正答のみ）。

```
Set-ADComputer -Identity (Get-ADComputer server2.fabrikam.com) -PrincipalsAllowedToDelegateToAccount (Get-ADComputer server1.contoso.com)
```

|パラメーター|選択|
|---|---|
|-Identity|(Get-ADComputer server2.fabrikam.com)|
|-PrincipalsAllowedToDelegateToAccount|(Get-ADComputer server1.contoso.com)|

## 解説134
- RBCD は「リソース側（アクセスを受ける側）」のコンピューター アカウントに対して設定します。今回は Server2（fabrikam.com）のアカウントに、「委任を許可する主体」として Server1（contoso.com）のコンピューター アカウントを登録します。
- PowerShell の Set-ADComputer コマンドレットの -PrincipalsAllowedToDelegateToAccount パラメーターで、RBCD の許可対象プリンシパルを指定できます。
- Windows Admin Center でゲートウェイ（Server1）からターゲット（Server2）へユーザー資格情報を委任して管理するには、Kerberos 制約付き委任（RBCD）の構成が必要です。

根拠（Microsoft 公式ドキュメント）
- Kerberos 制約付き委任の概要（RBCD の説明）: https://learn.microsoft.com/windows-server/security/kerberos/kerberos-constrained-delegation-overview#resource-based-constrained-delegation
- Set-ADComputer（-PrincipalsAllowedToDelegateToAccount の説明）: https://learn.microsoft.com/powershell/module/activedirectory/set-adcomputer
- Windows Admin Center の準備（Kerberos 制約付き委任の構成）: https://learn.microsoft.com/windows-server/manage/windows-admin-center/deploy/prepare-environment#configure-kerberos-constrained-delegation

## 問題135
オンプレミスの Active Directory Domain Services (AD DS) ドメイン contoso.com があり、Azure AD Connect を使用して Azure AD と同期しています。
contoso.com に対してパスワード保護を有効化しました。
ユーザーが自分のパスワードの一部として「Contoso」という単語を含めることを防ぐ必要があります。
何を使用する必要がありますか?
A. Windows Admin Center
B. Active Directory Users and Computers
C. the Azure Active Directory admin center
D. Synchronization Service Manager

## 解答135
C. the Azure Active Directory admin center

## 解説135
- Azure AD（Microsoft Entra ID）のパスワード保護では、カスタム禁止パスワード（例: “Contoso”）を Azure ポータル（Azure Active Directory 管理センター）で構成します。オンプレミス AD DS に対してパスワード保護を有効化している場合、ドメイン コントローラー エージェントが Azure からこのポリシーを取得し、パスワード変更時に強制します。
  - 根拠（公式ドキュメント）: Custom banned passwords はポータルで構成可能（"Custom banned passwords"）https://learn.microsoft.com/entra/identity/authentication/concept-password-ban-bad#custom-banned-passwords
  - 根拠（公式ドキュメント）: Windows Server Active Directory 向けパスワード保護（エージェントがポリシーをダウンロードして適用）https://learn.microsoft.com/entra/identity/authentication/concept-password-ban-bad-on-premises

|選択肢|可否|理由|
|---|---|---|
|Azure AD 管理センター|◯|カスタム禁止パスワード（“Contoso” など）の設定場所|
|Active Directory Users and Computers|×|禁止語リストの構成は不可（複雑性/履歴/長さのみ）|
|Windows Admin Center|×|対象外の管理ツール|
|Synchronization Service Manager|×|同期の管理ツールであり、パスワード保護ポリシーの構成ではない|

## 長文問題1
ケーススタディ1 - Fabrikam, Inc

概要
Fabrikam, Inc は、ニューヨーク本社とシアトル支社を持つ製造企業です。

既存環境
オンプレミス サーバー
オンプレミス ネットワークには、次の表のとおり Windows Server を実行するサーバーがあります。

|Name|Configuration|Office|
|---|---|---|
|AADC1|Azure AD Connect|New York|
|APP1|Application Server|New York|
|APP2|Application Server|Seattle|
|DC1|Domain Controller|New York|
|DC2|Domain Controller|Seattle|
|DHCP1|DHCP server|New York|
|DHCP2|DHCP server|Seattle|
|FS1|File server|New York|
|FS2|File server|Seattle|
|VM1|None|New York|
|VM2|None|Seattle|
|WEB1|Web server|New York|
|WEB2|Web server|Seattle|

DC1 はすべての操作マスター ロールをホストしています。
WEB1 と WEB2 では、Webapp1 という IIS Web アプリが稼働しています。

オンプレミス ネットワーク
ニューヨークとシアトルのオフィスは、冗長化された WAN リンクで接続されています。
各オフィスのクライアント コンピューターは、ローカルの DHCP サーバーから IP アドレスを取得します。
DHCP1 にはニューヨーク向けのアドレスを持つ Scope1 が、DHCP2 にはシアトル向けのアドレスを持つ Scope2 が構成されています。

ID基盤
ネットワークには、corp.falbrikam.com という単一のオンプレミス Active Directory Domain Services (AD DS) ドメインがあります（原文の綴り）。現在、すべてのサービス アカウントは個別のドメイン ユーザー アカウントを使用しています。
すべてのドメイン コントローラーには DNS Server ロールがインストールされており、corp.fabrikam.com の Active Directory 統合 DNS ゾーンのコピーをホストしています。
corp.fabrikam.com の AD DS ドメインは Azure Active Directory (Azure AD) テナントと同期しています。

Group Policy Objects (GPOs)
corp.fabrikam.com ドメインには、次の OU とカスタム GPO が存在します。

|OU name|Linked GPO|Description|
|---|---|---|
|AllUsers|GPO1|Contains all the user accounts in the domain|
|AllComputers|GPO2|Cotains all the computer accounts for the client computers in the domain|
|AllServers|GPO3|Contains all the computer accounts for Windows servers|
|VirtualDesktops|GPO4|A new OU that will contain the computers account for AzureVirtual Desktop session hosts|

Requirements
Planned Changes
Fabrikam は次の計画変更を特定しています:
* Sub1 という単一の Azure サブスクリプションを作成し、その中に Vnet1 という単一の Azure 仮想ネットワークを作成する。
* シアトルとニューヨーク間の WAN リンクを Azure Virtual WAN と ExpressRoute で置き換える。両オンプレミス拠点は ExpressRoute で Vnet1 に接続する。
* newyorkfiles、seattlefiles、companyfiles の 3 つの Azure ファイル共有を作成する。
* Vnet1 に dc3.corp.fabrikam.com というドメイン コントローラーを作成する。
* Vnet1 に Azure Virtual Desktop ホスト プールを展開する。Azure Virtual Desktop セッション ホストはハイブリッド Azure AD 参加とする。
* すべてのサーバーに Microsoft Defender for Servers のライセンスを付与する。
* Azure Policy を使用して、Azure 上およびオンプレミスのサーバーに構成管理ポリシーを適用する。

Networking Requirements
次のネットワーク要件があります:
* Virtual WAN を実装し、サイト間のすべてのネットワーク トラフィックが Virtual WAN を経由するようにする。すべての通信は ExpressRoute 上で行うこと。
* いずれかの DHCP サーバーが障害になっても、クライアント コンピューターが動的 IP アドレスの取得と既存リースの更新を継続できるようにする。
* Vnet1 のリソースが、corp.fabrikam.com ドメイン内のオンプレミス サーバー名を解決できるようにする。

Security Requirements
次のセキュリティ要件があります:
* GPO4 を Azure Virtual Desktop セッション ホストに適用する。Azure Virtual Desktop のユーザー セッションは 10 分間アイドル状態が続いたらロックされること。ユーザーは自分のクライアント コンピューターからロックアウト時間を手動で調整できること。
* サーバー管理者が Azure 仮想マシンへ RDP 接続を確立する前に承認を要求させること。承認された場合、2 時間以内に接続が確立される必要がある。
* ユーザー パスワードに、Fab、f@br1kAm、fabr!| など会社名に基づく単語の全部または一部を含めないようにすること。
* すべての Webapp1 のインスタンスで同じサービス アカウントを使用すること。そのサービス アカウントのパスワードは 30 日ごとに自動で変更すること。
* ドメイン コントローラーが直接インターネット上のホストへ接続しないようにすること。

File Sharing Requirements
Azure Files の同期構成で次を満たす必要があります:
* seattlefiles は FS2 と同期すること。
* newyorkfiles は FS1 と同期すること。
* companyfiles は FS1 と FS2 の両方と同期すること。

## 問題1
Azure Virtual Desktop セッション ホストがセキュリティ要件を満たすように、グループ ポリシー設定を構成する必要があります。
何を構成すべきですか？

A. GPO4 のリンクに対するセキュリティ フィルターリング  
B. GPO1 のリンクに対するセキュリティ フィルターリング  
C. GPO4 でのループバック処理の構成  
D. GPO1 のリンクに対する「強制」の設定  
E. GPO1 でのループバック処理の構成  
F. GPO4 のリンクに対する「強制」の設定

## 解答1
C. GPO4 でのループバック処理の構成

## 解説1
- ユーザー構成（例: アイドル 10 分でのロック）を、特定のコンピューター（AVD セッション ホスト）にログオンしたユーザーへ適用するには、そのコンピューターが属する OU にリンクされた GPO で「ユーザーのグループ ポリシーのループバック処理」を有効化します。これにより、ユーザーがどの OU にいても、対象セッション ホストにログオンしたときに GPO4 のユーザー設定が適用されます。  
  - 参考: User Group Policy loopback processing mode（Microsoft Learn）  
    https://learn.microsoft.com/windows/security/threat-protection/security-policy-settings/user-group-policy-loopback-processing-mode  
  - 参考: ループバック処理の詳細（トラブルシューティング記事）  
    https://learn.microsoft.com/troubleshoot/windows-server/group-policy/loopback-processing-of-group-policy

## 問題2
ファイル共有要件を満たすように Azure File Sync を構成する必要があります。
何を行うべきですか？
該当する選択肢を回答エリアから選んでください。注: 各正解は 1 ポイントです。

Answer Area
Minimum number of sync groups to create: 「1」 or 「2」 or 「3」 or 「4」
Minimum number of Storage Sync Services to create: 「1」 or 「2」 or 「3」 or 「4」

## 解答2
- Minimum number of sync groups to create: 3
- Minimum number of Storage Sync Services to create: 1

## 解説2
- 同期は「同期グループ（sync group）」単位で定義され、各同期グループは必ず 1 つのクラウド エンドポイント（= 1 つの Azure ファイル共有）を持ちます。newyorkfiles／seattlefiles／companyfiles はそれぞれ別の共有であるため、最小でも 3 つの同期グループが必要です。companyfiles の同期グループには FS1 と FS2 のサーバー エンドポイントを両方追加します。
  - 参考: Azure File Sync の概要／計画（Sync group・エンドポイントの説明）
    - https://learn.microsoft.com/azure/storage/file-sync/file-sync-introduction
    - https://learn.microsoft.com/azure/storage/file-sync/file-sync-planning#sync-group
- Storage Sync Service は Azure File Sync の管理単位で、同一リージョン内で複数の同期グループと登録サーバーを 1 つにまとめて管理できます。本シナリオの最小構成では 1 つで足ります（ストレージ アカウントと同一リージョンである必要があります）。
  - 参考: Storage Sync Service の設計指針・要件
    - https://learn.microsoft.com/azure/storage/file-sync/file-sync-planning#storage-sync-service

## 問題3
シアトル オフィスとニューヨーク オフィス間のネットワーク通信を構成する必要があります。ソリューションはネットワーク要件を満たさなければなりません。
何を構成するべきですか。回答するには、解答エリアで適切なオプションを選択してください。
注: 正解 1 つにつき 1 点です。

On a Virtual WAN hub:
「An ExpressRoute gateway」または「A virtual network gateway」または「An ExpressRoute circuit connection」

In the offices:
「An ExpressRoute circuit connection」または「A Site to-Site VPN」または「An Azure application gateway」または「An on premises data gateway」

## 解答3
- On a Virtual WAN hub: An ExpressRoute gateway
- In the offices: An ExpressRoute circuit connection

## 解説3
- すべてのサイト間通信を Virtual WAN 経由かつ ExpressRoute のみで行う要件より、Virtual WAN ハブには ExpressRoute 接続を終端するための「ExpressRoute ゲートウェイ」をデプロイします。Virtual WAN では vNet 用の「Virtual network gateway」ではなく、ハブの「ExpressRoute gateway」を使用します。
  - 根拠（Microsoft 公式ドキュメント）: Virtual WAN のハブに ExpressRoute ゲートウェイをデプロイし、ハブ経由で接続する https://learn.microsoft.com/azure/virtual-wan/virtual-wan-about
- 各オフィス側はキャリア提供の ExpressRoute 回線（回線キー）を用意し、ハブに対して「ExpressRoute circuit connection（ER 回線接続）」を構成して紐付けます。これにより、両サイト間のトラフィックは Microsoft グローバル ネットワーク上の Virtual WAN ハブを経由して相互到達し、要件（ExpressRoute のみ、Virtual WAN 経由）を満たします。Site-to-Site VPN は「ExpressRoute のみ」の要件に反します。
  - 根拠（Microsoft 公式ドキュメント）: Virtual WAN で ExpressRoute 回線を接続（ハブの ER ゲートウェイに回線接続を作成） https://learn.microsoft.com/azure/virtual-wan/virtual-wan-expressroute-portal

## 問題4
ネットワーク要件を満たす名前解決ソリューションを実装する必要があります。
どの 2 つのアクションを実行する必要がありますか？
正しい回答は解の一部を構成します。
注意: 各正解は 1 ポイントの価値があります。

選択肢：
A. corp.fabhkam.com という名前の Azure プライベート DNS ゾーンを作成する。
B. coip.fabnkam.com の Azure プライベート DNS ゾーンで仮想ネットワーク リンクを作成する。
C. corp.fabrikam.com という名前の Azure DNS ゾーンを作成する。
D. Vnet1 の DNS サーバー設定を構成する。
E. corp.fabnkam.com の Azure プライベート DNS ゾーンで自動登録を有効にする。
F. DC3 に DNS Server ロールをインストールする。
G. DC3 に条件付きフォワーダーを構成する。

## 解答4
D、F

## 解説4
- 目的は「Vnet1 のリソースがオンプレミスの AD DS ドメイン corp.fabrikam.com 内のサーバー名を解決できるようにする」ことです。これには、VNet の DNS サーバーを既定の Azure 提供 DNS ではなく、ドメイン内の DNS（AD 統合 DNS をホストする DC）に切り替えるのが推奨手法です（D）。
- Vnet1 に配置する DC3 を DNS サーバーとして構成（DNS Server ロールを追加）すると、AD 統合ゾーン corp.fabrikam.com が DC どうしでレプリケートされ、Vnet1 内の VM は DC3 を参照してオンプレミス名を解決できます（F）。
- パブリックの Azure DNS ゾーン（C）は社内プライベート ドメインの解決には不適切で、プライベート DNS ゾーン関連（A/B/E）はこの要件には不要です。DC3 がゾーンをホストするため、条件付きフォワーダー（G）も必須ではありません。

根拠（Microsoft 公式ドキュメント）
- Azure VM の名前解決（独自の DNS サーバーを使用）: https://learn.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances#name-resolution-using-your-own-dns-server
- Active Directory 統合 DNS ゾーンとレプリケーション: https://learn.microsoft.com/windows-server/networking/dns/deploy/active-directory-integrated-zones

## 長文問題2
ケーススタディ2 - Contoso, Ltd

Overview
Contoso, Ltd は、シアトルに本社、ロサンゼルスとモントリオールに 2 つの支社を持つ企業です。

Existing Environment
AD DS Environment
ネットワークには、オンプレミスの Active Directory Domain Services (AD DS) フォレスト contoso.com があります。フォレストには contoso.com と canada.contoso.com の 2 つのドメインが含まれます。

フォレストには次のドメイン コントローラーが含まれます。
|Name|Domain|Active Directory site|
|---|---|---|
|DC1|contoso.com|Seattle|
|DC2|contoso.com|Los Angeles|
|DC3|canada.contoso.com|Montreal|
|DC4|contoso.com|Montreal|
|DC5|canada.contoso.com|Seattle|

すべてのドメイン コントローラーはグローバル カタログ サーバーです。

Server infrastructure
ネットワークには次のサーバーが含まれます。
|Name|Organizational Unit(OU)|Server role|Domain|Active Directory site|
|---|---|---|---|---|
|Server1|Member Servers|None|canada.contoso.com|Montreal|
|Server2|Member Servers|Hyper-V|canada.contoso.com|Montreal|
|Server3|Member Servers|None|canada.contoso.com|Montreal|

Server4 という名前のサーバーは Windows Server を実行しており、ワークグループに所属しています。Server4 の Windows ファイアウォールはプライベート プロファイルを使用しています。

Server2 には VM1、VM2、VM3 という 3 台の仮想マシンがホストされています。

VM3 はファイル サーバーで、次のボリュームにデータを保存しています。
|Name|File system|
|---|---|
|C|NTFS|
|D|NTFS|
|E|ReFS|
|F|ExFat|

Group Policies
contoso.com ドメインには次のグループ ポリシー オブジェクト (GPO) があります。
|Name|Minimum password length|Linked to|
|---|---|---|
|GPO1|14|OU1|
|GPO2|8|Member Servers|
|Default Domain Policy|10|contoso.com|

Existing Identities
フォレストには次のユーザーが含まれます。
|Name|In OU|Member of|
|---|---|---|
|Contoso\Admin1|Contoso\OU1|Contoso\Enterprise Admins|
|Contoso\Admin2|Contoso\OU1|Contoso\Domain Admins|
|Canada\Admin3|Canada\OU2|Canada\Domain Admins|
|Contoso\User1|Contoso\OU3|Contoso\Domain Users|

フォレストには次のグループが含まれます。
|Name|Domain|Type|
|---|---|---|
|Group1|contoso.com|Universal security group|
|Group2|contoso.com|Global security group|
|Group3|contoso.com|Domain local security group|
|Group4|canada.contoso.com|Global distribution group|
|Group5|canada.contoso.com|Global distribution group|
|Group6|canada.contoso.com|Domain local distribution group|

Current Problems
管理者が Virtual Machine Connection を使用して VM2 のコンソールにサインインし、サインアウトせずにセッションを切断すると、別の管理者が現在サインインしているユーザーとしてそのコンソール セッションに接続できてしまいます。

Requirements
Technical Requirements
Contoso は次の技術要件を特定しています。
* すべてのサイト間リンクのレプリケーション スケジュールを 30 分に変更する。
* Server1 を canada.contoso.com のドメイン コントローラーに昇格させる。
* Server3 を DHCP サーバーとしてインストールして承認する。
* User1 が Contoso\OU3 内のすべてのグループのメンバーシップを管理できるようにする。
* Server1 から PowerShell リモーティングを使用して Server4 を管理できるようにする。
* VM1 で仮想マシンを実行できるようにする。
* ユーザーが VM2 に接続する際に、資格情報の入力を必須にする。
* VM3 上のすべてのボリュームで重複除去 (Data Deduplication) を使用可能にする。

## 問題1
VM2 に関する技術要件を満たす必要があります。何を実行すべきですか？
A. シールドされた仮想マシンを実装する。
B. ゲスト サービス統合サービスを有効にする。
C. Credential Guard を実装する。
D. 拡張セッション モードを有効にする。

## 解答1
D. 拡張セッション モードを有効にする。

## 解説1
- 目的は「VM2 に接続する際に常に資格情報の入力を要求し、前回のコンソール セッションにそのまま入れてしまう挙動を防ぐ」ことです。Hyper‑V の拡張セッション モードは VMConnect が RDP ベースでゲストに接続する仕組みを提供し、接続時に資格情報の入力（サインイン）が求められます。これにより、誰かがコンソールでサインインしっぱなしでも、接続側は自分の資格情報でサインインする必要があり、要件を満たします。
- 設定の目安: Hyper‑V マネージャー > Hyper‑V の設定 > 拡張セッション モード ポリシー「許可する」、およびユーザーごとの「拡張セッション モードを使用する」を有効化。
- Microsoft 公式ドキュメント（根拠）:
  - Virtual Machine Connection (VMConnect) は、拡張セッション モードが利用可能な場合に RDP を用いてゲストに接続し、接続時にサインインが必要になります（資格情報の入力を促されます）。https://learn.microsoft.com/windows-server/virtualization/hyper-v/manage/virtual-machine-connection-vmconnect
  - 拡張セッション モードでは、接続時のダイアログやリソースリダイレクトを含む RDP 体験が提供され、ゲスト OS へのサインインを行います。https://learn.microsoft.com/windows-server/virtualization/hyper-v/learn-more/use-local-resources-on-hyper-v-virtual-machine-with-virtual-machine-connection
- 他の選択肢が不適切な理由（要点）:
  - シールドされた仮想マシン（A）はファブリック管理者からの保護や暗号化に関する機能で、VMConnect の接続時に資格情報を必須化する要件には直接関係しません。
  - ゲスト サービス（B）はホストからゲストへのファイルコピー等の機能で、サインイン要求の挙動は変えません。
  - Credential Guard（C）は OS 内部の資格情報保護機能であり、VMConnect の接続プロンプト要件とは別問題です。

## 問題3
次の各記述について、正しい場合は Yes、そうでない場合は No を選択してください。
注: 各正解は 1 点の価値があります。

解答エリア
|Statements|Yes|No|
|---|---|---|
|Admin1 must use a password that has a least 14 characters.| | |
|User1 must use a password that has at least 10 characters.| | |
|If Admin1 creates a new local user on Server1, the password for the new user must be at least eight characters.| | |

## 解答3
|Statements|Yes|No|
|---|---|---|
|Admin1 must use a password that has a least 14 characters.| |No|
|User1 must use a password that has at least 10 characters.|Yes| |
|If Admin1 creates a new local user on Server1, the password for the new user must be at least eight characters.| |No|

## 解説3
- ドメイン アカウントのパスワード ポリシーは、ドメインのルートにリンクされた GPO（通常は Default Domain Policy）で定義されたアカウント ポリシーが適用されます。OU にリンクした GPO のアカウント ポリシーはドメイン アカウントには影響せず、その OU に属するコンピューターのローカル アカウントにのみ影響します。本ケースでは contoso.com の Default Domain Policy が最小 10 文字のため、Admin1 に「14 文字以上」は不要で、User1 は「10 文字以上」が必要です。
  - 参考: Microsoft Learn「Fine-Grained Password Policies」https://learn.microsoft.com/windows-server/identity/ad-ds/manage/component-updates/fine-grained-password-policies
  - 参考: Microsoft Learn「Group Policy の概要」https://learn.microsoft.com/windows-server/identity/ad-ds/get-started/group-policy/group-policy-overview
- Server1 は canada.contoso.com ドメインの Member Servers OU にあり、contoso.com ドメインの GPO2（最小 8 文字）は適用されません。ローカル アカウントの「最小パスワード長」の既定値は 0 のため、「少なくとも 8 文字でなければならない」とは言えません。
  - 参考: Microsoft Learn「Minimum password length」https://learn.microsoft.com/windows/security/threat-protection/security-policy-settings/minimum-password-length

## 問題4
VM3 が技術要件を満たすようにする必要があります。最初に何をインストールするべきですか？
A. iSNS サーバー サービス
B. File Server Resource Manager (FSRM)
C. Enhanced Storage
D. Windows Standards-Based Storage Management

## 解答4
B

## 解説4
- データ重複除去 (Data Deduplication) を使用するには、Windows Server の「ファイルおよび記憶域サービス」に含まれる役割サービスとして機能を追加する必要があります。まずファイル サービス周辺の役割群を有効化してから重複除去を追加するのが前提となります。選択肢の中では、FSRM がファイル サービス役割群の導入に直結するため最初のインストール対象として適切です（その後に「データ重複除去」役割サービスを追加します）。
- なお、重複除去は NTFS（および Windows Server 2019 以降の ReFS）のみでサポートされ、FAT/ExFAT はサポートされません。したがって VM3 の F:（ExFAT）は NTFS への再フォーマットが必要であり、E:（ReFS）は OS が Windows Server 2019 以降であれば対応可能です。

参考（Microsoft 公式ドキュメント）:
- Data Deduplication 概要: https://learn.microsoft.com/windows-server/storage/data-deduplication/overview
  - 「Data Deduplication は Windows Server の機能で、… サポート対象は NTFS（および Windows Server 2019 以降の ReFS）。FAT/ExFAT は非サポート」
- Data Deduplication のインストール/有効化: https://learn.microsoft.com/windows-server/storage/data-deduplication/install-enable
  - 「[ファイルおよび記憶域サービス] の役割サービスとして [データ重複除去] を追加して有効化する」

## 問題5
Server1 に関する技術要件を満たす必要があります。現在、必要なタスクを実行できるユーザーはどれですか？
A. Admin1 and Admin3 only
B. Admin3 only
C. Admin1 Admin2. and Admm3
D. Admin1 only

## 解答5
A

## 解説5
Server1 に関する技術要件は「Server1 を canada.contoso.com のドメイン コントローラーに昇格させる」ことです。Microsoft 公式ドキュメントでは、既存ドメインに追加のドメイン コントローラーをインストールするには、対象ドメインの Domain Admins または Enterprise Admins のメンバー シップ（または同等の委任権限）が必要とされています。したがって、
- Enterprise Admins である Admin1
- canada.contoso.com の Domain Admins である Admin3
が現在この作業を実行できます。contoso.com の Domain Admins である Admin2 は canada.contoso.com ドメインに対する必要権限を持たないため該当しません。

根拠（Microsoft 公式ドキュメント）:
- 「Add a domain controller to an existing domain（既存ドメインにドメイン コントローラーを追加）」の前提資格情報: Domain Admins（対象ドメイン）または Enterprise Admins が必要。
  https://learn.microsoft.com/windows-server/identity/ad-ds/deploy/adds-install-overview
- 既定のセキュリティ グループ「Enterprise Admins」: フォレスト全体の管理権限を持ち、すべてのドメインに対する広範な管理操作が可能。
  https://learn.microsoft.com/windows-server/identity/ad-ds/manage/understand-security-groups#enterprise-admins

## 問題6
Group3 と Group5 に追加できるグループはどれですか？回答するには、解答領域で適切なオプションを選択してください。注意: 各正解は 1 ポイントの価値があります。

解答領域
Group3:「Group6 only」or「Group1 and Group2 only」or「Group1 and Group4 only」or「Group1, Group2, Group4 and Group5 only」or「Group1, Group2, Group4, Group5, and Group6」
Group5:「Group1 only」or「Group4 only」or「Group6 only」or「Group2 and Group4 only」or「Group4 and Group6 only」

## 解答6
|項目|選択肢|
|---|---|
|Group3|Group1 and Group2 only|
|Group5|Group4 only|

## 解説6
- Group3 は「Domain local security group（contoso.com）」です。ドメイン ローカル（セキュリティ）グループは、任意ドメインのグローバル/ユニバーサル セキュリティ グループおよびアカウントをメンバーにできます。一方、配布（Distribution）グループはセキュリティが有効ではないため、権限付与を目的としたセキュリティ グループのメンバーとして適切ではありません。よって Group1（ユニバーサル セキュリティ）、Group2（グローバル セキュリティ）は可、Group4/5/6（いずれも配布）は不可です。
- Group5 は「Global distribution group（canada.contoso.com）」です。グローバル グループは同一ドメイン内のアカウントとグローバル グループのみをメンバーにできます。したがって、同じドメイン（canada.contoso.com）のグローバル配布グループである Group4 は追加可能ですが、別ドメインの Group2（contoso.com）や、ユニバーサル/ドメイン ローカル（Group1/Group6）は追加できません。

根拠（Microsoft 公式ドキュメント）
- グループ スコープ（メンバーシップ規則: グローバル/ドメイン ローカル/ユニバーサル）: https://learn.microsoft.com/windows-server/identity/ad-ds/plan/security-groups#group-scope
- セキュリティ グループと配布グループの違い（配布グループは権限付与には使用できない/セキュリティが必要な場合はセキュリティ グループを使用）: https://learn.microsoft.com/windows-server/identity/ad-ds/manage/understand-security-groups#security-and-distribution-groups

## 長文問題3

Case Study 3 - ADatum Corporation（ケーススタディ3 – ADatum社）

概要（Overview）

会社情報（Company Information）
- ADatum社は製造業の企業で、シアトルに本社、ロサンゼルスとモントリオールに支社があります。

Fabrikam社とのパートナーシップ（Fabrikam Partnership）
- ADatumは最近、Fabrikam社（2社）と提携しました。
- Fabrikam社は製造業の企業で、ボストンに本社、オーランドに支社があります。
- 両社はいくつかの共同プロジェクトでコラボレーションする予定です。

既存環境（Existing Environment）

ADatum の AD DS 環境（ADatum AD DS Environment）
- ADatum のオンプレミスネットワークには、adatum.com という名前の Active Directory Domain Services（AD DS）フォレストがあります。
- フォレストには adatum.com と east.adatum.com の2つのドメインがあり、次のドメイン コントローラー（DC）が存在します。

|Name|Domain|Operations master roles|
|---|---|---|
|DC1|adatum.com|Schema master|
|DC2|adatum.com|None|
|DC3|east.adatum.com|PDC emulator, RID master|

Fabrikam の AD DS 環境（Fabrikam AD DS Environment）
- Fabrikam のオンプレミスネットワークには、fabrikam.com という AD DS フォレストがあります。
- フォレストには fabrikam.com と south.fabrikam.com の2つのドメインがあります。
- fabrikam.com ドメインには Marketing という組織単位（OU）があります。

サーバー基盤（Server Infrastructure）
- adatum.com ドメインには次のサーバーがあります。

|Name|Role|
|---|---|
|HyperV1|Hyper-V|
|SSPace1|File and Storage Services|

- HyperV1 には次の仮想マシン（VM）が存在します。

|Name|Operating system|Description|
|---|---|---|
|VM1|Windows Server 2022 Datacenter|adatum.com ドメイン参加／Data1 というファイル共有あり／ローカルユーザー User1|
|VM2|Red Hat Enterprise Linux (RHEL)|ローカルユーザー User2|
|VM3|Windows Server 2022 Standard|adatum.com ドメイン参加／File and Storage Services 役割インストール済み|

- HyperV1 上のすべての仮想マシンには既定の管理ツールのみがインストールされています。

- SSPace1 には次の Storage Spaces 仮想ディスクがあります。

|Name|Number of pysical disks|Redundancy|
|---|---|---|
|Disk1|8|Three-way mirror|
|Disk2|12|Parity|

Azure リソース（Azure Resources）
- ADatum は Azure サブスクリプションと Azure AD テナントを持ち、Azure AD Connect により adatum.com フォレストと Azure AD を同期しています。
- サブスクリプションには次の仮想ネットワーク（VNet）があります。

|Name|Location|Subnet|
|---|---|---|
|VNet1|West US|Subnet1, Subnet2|
|VNet2|West US|SubnetA, SubnetB|

- 次の Azure Private DNS ゾーンがあります。

|Name|Virtual network link|
|---|---|
|Zone1.com|VNet1|
|Zone2.com|VNet2|
|Zone3.com|None|

- サブスクリプションには次の仮想マシンがあります（いずれもワークグループ所属）。

|Name|Operating system|Security type|
|---|---|---|
|Server1|Windows Server 2022 Datacenter: Azure Edition|Trusted launch|
|Server2|Windows Server 2022 Datacenter: Azure Edition|Standard|
|Server3|Windows Server 2022 Datacenter|Standard|
|Server4|Windows Server 2019 Datacenter|Trusted launch|

- ストレージアカウント storage1 があり、share1 というファイル共有があります。

要件（Requirements）

計画されている変更（Planned Changes）
- Data1 を share1 と同期する。
- Task1 という名前の Azure Runbook を構成する。
- Azure AD ユーザーが Server1 にサインインできるようにする。
- 次の構成で Azure DNS Private Resolver を作成する：
  - Name: Private1
  - Region: West US
  - Virtual network: VNet1
  - Inbound endpoint: SubnetB
- adatum.com ドメインのユーザーが south.fabrikam.com ドメインのリソースにアクセスできるようにする。

技術要件（Technical Requirements）
- SSPace1 のデータは常に利用可能でなければならない。
- DC1 が障害した場合、DC2 はスキーママスターになる必要がある。
- VM3 はフォルダー単位のクォータ（per-folder quotas）を有効化するよう構成する必要がある。
- 信頼（トラスト）は必要なリソースにのみアクセスを許可する形で構成する必要がある。
- Marketing OU のユーザーは storage1 にアクセスできる必要がある。
- すべてのサポート対象の Azure 仮想マシンに Azure Automanage を使用する必要がある。
- HyperV1 上のサポート対象のすべての仮想マシンを、直接の SSH セッションで管理する必要がある。

## 問題1
Task1 に使用できる言語はどれですか。正解を2つ選んでください（Each correct answer presents a complete solution）。
- A. Bicep
- B. Python
- C. Java
- D. PowerShell
- E. JavaScript


## 解答1
- B. Python
- D. PowerShell


## 解説1
- Azure Automation の Runbook がサポートする言語は、PowerShell（および PowerShell 7）と Python（Python 3）です。Bicep、Java、JavaScript は Runbook の実行言語としてはサポートされていません。
- 公式ドキュメントの根拠：
  - Runbook の種類（Types of runbooks）として「PowerShell」「PowerShell Workflow（非推奨）」「Python」が挙げられています。
    - Microsoft Learn: Runbook types in Azure Automation
      https://learn.microsoft.com/azure/automation/automation-runbook-types
  - Python 3 Runbook のサポート（Azure Automation で Python 3 Runbook を作成・実行できる）
    - Microsoft Learn: Azure Automation Python 3 runbooks
      https://learn.microsoft.com/azure/automation/automation-python-3-runbooks

したがって、Task1（Azure Runbook）で使用できる言語は Python と PowerShell です。

## 問題2
HyperV1 の技術要件を満たす必要があります。
どのコマンドを実行すべきですか？（回答エリアの適切な選択肢を選んでください）
注意: 各正解は 1 ポイントです。

Answer Area
```
<「Connect-PSSession」 or 「Connect-WSMan」 or 「hvc.exe」 or 「mstsc.exe」> ssh <「User1@VM1」 or 「User1＠VM2」 or 「User2@VM2」 or 「VM1」>
```

## 解答2
hvc.exe ssh User2@VM2

## 解説2
- 技術要件「HyperV1 上のすべての対応 VM を管理する際、ダイレクト SSH セッションを使用する」に合致させるには、Hyper‑V ホストからネットワークを介さずにゲストへ SSH 接続できる「SSH Direct（hvc.exe、Hyper‑V Sockets/VSOCK を使用）」を用います。Linux ゲスト（VM2 は RHEL）かつローカル ユーザー User2 が存在するため、形式は「hvc.exe ssh User2@VM2」となります。
  - 参考: OpenSSH on Windows 概要（Windows における SSH の利用）
    - https://learn.microsoft.com/windows-server/administration/openssh/openssh_overview
  - 参考: PowerShell Direct（Windows ゲスト向けのダイレクト接続の対比。Linux には SSH Direct/hvc.exe を使用）
    - https://learn.microsoft.com/windows-server/virtualization/hyper-v/learn-more/powershell-direct

## 問題3
SSPace1 上のデータ可用性が技術要件を満たすようにする必要があります。
各ディスクで同時に障害が発生しても許容できる物理ディスクの最大数はいくつですか？
回答するには、回答領域で適切な選択肢を選択してください。
注: 正解の選択肢はそれぞれ 1 点の価値があります。

Answer Area
Disk1:「1」or「2」or「3」or「4」or「5」
Disk2:「1」or「2」or「3」or「4」or「5」

## 解答3
Disk1: 2
Disk2: 1

## 解説3
- Storage Spaces の冗長性で、Three-way mirror は同時に 2 台までの物理ディスク障害に耐えられます。一方、Parity（単一パリティ）は同時に 1 台の物理ディスク障害に耐えます。
- したがって、Disk1（Three-way mirror）は 2、Disk2（Parity）は 1 が最大許容故障数です。
- 公式ドキュメント（根拠）:
  - Storage Spaces の「Resiliency types」では、three-way mirror は 2 台のドライブ障害を許容し、parity は 1 台のドライブ障害を許容すると説明されています。
    https://learn.microsoft.com/windows-server/storage/storage-spaces/overview#resiliency-types

## 問題4
VM3 が技術要件を満たすように構成する必要があります。最初にインストールすべきものはどれですか。
A. Enhanced Storage
B. iSNS Server サービス
C. File Server Resource Manager (FSRM)
D. Windows Standards-Based Storage Management

## 解答4
C. File Server Resource Manager (FSRM)

## 解説4
- 技術要件は「VM3 でフォルダー単位のクォータを有効化できるようにする」ことです。フォルダー単位（per-folder）のクォータは FSRM のクォータ管理機能で提供され、特定フォルダーに制限を設定できます。まず FSRM をインストールする必要があります。
  - 公式ドキュメント: FSRM 概要（クォータ管理はボリュームまたはフォルダーにサイズ制限を適用可能）
    - https://learn.microsoft.com/windows-server/storage/fsrm/fsrm-overview#quota-management
  - 公式ドキュメント: クォータの作成（フォルダーへのクォータ設定手順）
    - https://learn.microsoft.com/windows-server/storage/fsrm/create-quota
- 誤答の要点:
  - A: Enhanced Storage はクォータ機能ではありません。
  - B: iSNS Server は iSCSI 名称サービスで、クォータと無関係です。
  - D: Standards-Based Storage Management はストレージ管理 API/SMI-S 向けで、フォルダー クォータは提供しません。

## 問題5
Server4 の技術要件を満たす必要があります。
Server1 と Server4 で実行すべき cmdlet はどれですか。
解答するには、解答エリアで適切な選択肢を選んでください。

注意: 各正解は 1 ポイントの価値があります。

解答エリア
- Server1:「Enable-PSRemoting」または「Enable-ServerManagerStandardUserRemoting」または「Set-Item」または「Start-Service」
- Server4:「Enable-PSRemoting」または「Enable-ServerManagerStandardUserRemoting」または「Set-Item」または「Start-Service」

## 解答5
- Server1: Set-Item
- Server4: Enable-PSRemoting

## 解説5
- いずれのサーバーもワークグループのため、Server1 から Server4 をリモート管理するには、管理側（Server1）の WinRM クライアントで TrustedHosts に対象（Server4）を登録し、対象側（Server4）で PowerShell リモーティング（WinRM リスナーとファイアウォール例外）を有効化するのが基本手順です。これにより資格情報指定による接続が可能になります。
  - Server1（管理側）: Set-Item WSMan:\localhost\Client\TrustedHosts -Value Server4 -Force
  - Server4（対象側）: Enable-PSRemoting -Force（WinRM サービスの起動/自動化と必要なファイアウォール規則の有効化を実施）
- 根拠（Microsoft 公式ドキュメント）：
  - about_Remote_Troubleshooting（信頼されていないドメイン/ワークグループ間での接続と TrustedHosts の構成。Set-Item による WSMan:\localhost\Client\TrustedHosts の設定）
    https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_remote_troubleshooting
  - Enable-PSRemoting（WinRM リスナー、ファイアウォール規則、サービス起動を自動構成）
    https://learn.microsoft.com/powershell/module/microsoft.powershell.core/enable-psremoting

## 問題6
VM1 の技術要件を満たす必要があります。
まずどのコマンドレットを実行すべきですか。
回答するには、解答エリアで適切なオプションを選択してください。
注: 各正解は 1 点の配点です。
```
<「Set-VM」or「Set-VMBios」or「Set-VMHost」or「Set-VMFirmware」or「Set-VMProcessor」> VM1 <「-NewVMName」or「GuestControlledCacheTypes」or「-EnableHostResourceProtection」or「-ExposeVirtualizationExtensions」> $true
```

## 解答6
Set-VMProcessor VM1 -ExposeVirtualizationExtensions $true

## 解説6
- VM に入れ子の仮想化（Nested Virtualization）を有効化するには、まず対象 VM のプロセッサ設定で「仮想化拡張機能をゲストに露出」させます。これは Set-VMProcessor の -ExposeVirtualizationExtensions で行います。
- Microsoft 公式ドキュメントでも「Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true」を手順として示しています。
参考: https://learn.microsoft.com/virtualization/hyper-v-on-windows/user-guide/nested-virtualization
- Set-VMProcessor のリファレンスでも、-ExposeVirtualizationExtensions はゲストに仮想化拡張機能を露出する設定であると記載されています。
参考: https://learn.microsoft.com/powershell/module/hyper-v/set-vmprocessor?view=windowsserver2022-ps
