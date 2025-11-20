# はじめに
今回はMicrosoft Entra Private Accessを使用して、自宅PCからオフィスの検証環境に接続する方法について説明させていただきたいと思います。

# Microsoft Entra Private Accessとは
Microsoft Entra Private AccessはVPNの代替となる、リモートサービスです。
インターネットから接続するので危険だと思うかもしれませんが、MFAによる認証/条件付きアクセスによる接続元のデバイスやIP制限/監査ログの収集などゼロトラストセキュリティに基づいた仕組みによりセキュリティは強いです。
むしろVPNよりもセキュリティが強く、VPN装置の脆弱性対策、VPN装置の単一障害点対策、各拠点への高価なVPN装置の購入/運用費用がかからないので注目されている技術です。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/adf6a9ff-d618-4c2c-aa66-26441cc47428.png)

ただ、料金は無料ではなく以下のライセンス料金がかかります。
- Microsoft Entra ID P1以上 (¥899 ユーザー/月相当)
- Microsoft Entra Suite (¥1,799 ユーザー/月相当)


# Windows Server 2022 評価版VMの作成
それでは環境構築していきます。
まずは、リモート接続先として、VMware環境にサーバーを作成します。
- 必要ファイルのダウンロード
    * Windows Server 2022 評価版ISO
    [こちらからダウンロード](https://www.microsoft.com/ja-jp/evalcenter/download-windows-server-2022)
    * VMware Workstation Pro
    [ダウンロードとインストール方法](https://forest.watch.impress.co.jp/docs/review/2002377.html)

- 新規VMの作成
    * VMware Workstation起動→[新規仮想マシンの作成]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/66879ab0-1116-4de1-8321-053b04515a1e.png)
    * [カスタム]を選択して[次へ]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/aac1dd3a-45e2-41c7-b922-2ceab094ea50.png)
    * そのまま[次へ]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f3013506-5e15-48ef-99f6-c82bfe157e59.png)
    * [後でOSをインストール]を選択して[次へ]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/06cc3d9b-8ef1-4c3d-b5d9-8bfbf38330fb.png)
    * そのまま[次へ]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/2a0caf9c-4715-4000-8995-a9bcc1fc885f.png)
    * VM名を入力して[次へ]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/7903d638-ac3b-48fc-8356-8310baa5e99f.png)
    * 「セキュアブート」にチェックを入れて[次へ]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/dfc6eb34-0c4c-488c-ace7-913ec6d039b8.png)
    * そのまま[次へ]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/5c3a1973-cfad-46b5-b1b6-cfe6862de3bb.png)
    * メモリを[4096]に設定し、[次へ]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/fdfdf87b-b9d1-4031-a7ee-effca923db21.png)
    * そのまま[次へ]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/1d9646e8-e27b-4ce8-ae46-d651bee511db.png)
    * そのまま[次へ]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/80cc72ed-23f6-4347-ac00-5e8090f5ee1f.png)
    * そのまま[次へ]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/33ae7755-2b98-4c67-a8c9-b50c19a8b72c.png)
    * そのまま[次へ]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/0984c494-907f-4863-9d80-6d3bfb49c0cc.png)
    * そのまま[次へ]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/7cbd2083-208c-451c-80b4-87bd71f04163.png)
    * そのまま[次へ]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/02bc02c5-7e2f-4283-b016-245a72ab7831.png)
    * [完了]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/fd914b03-65a2-4b40-842d-f77303f93686.png)
    * [仮想マシンの設定を編集する]をクリックする
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/2cda69d1-c030-4d77-81d3-c4083e375051.png)
    * [CD/DVD(SATA]をクリックし、[ISOイメージファイルを使用する]を選択しISOファイルを指定後、[OK]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/cf6bc29c-9c47-419f-8ddd-6f71c5f05289.png)
    * 仮想マシンをパワーオンして接続後、「Press Any Key」と出るので何かKeyを押す
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/55d9cbe2-0cf0-4bb5-b1e4-48b4840921c9.png)
    * Windowsのセットアップ画面に入るので、そのまま[次へ]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/da1ec29a-8d69-4dfc-982f-137810a70f89.png)
    * [今すぐインストール]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/d27ebead-fd92-4bc8-ae1f-2d38be051d94.png)
    * Windows Server 2022 Datacenterのデスクトップ版を選択して[次へ]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/a5204ad2-daac-4bc2-9d66-2b02278cb1ca.png)
    * ライセンス条項に同意して[次へ]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/735dcdf1-a97a-49cc-8b01-eab66937a9c8.png)
    * [カスタム]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/627111fd-04c7-435b-9a19-dcaa03c7d4a3.png)
    * [新規]→[適用]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/aec4d23a-d008-4e53-a59c-970b874085bb.png)
    * [OK]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/bcbbbf45-ff4e-4fe5-a86a-740c800a77d0.png)
    * [次へ]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ee7d6608-ebe1-47b8-8a96-81cd160b270d.png)
    * OSのインストールが開始し、終了すると以下の画面になるのでパスワードを入力し、[完了]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/db8e9365-9612-449d-b141-26c87634e731.png)
    * VMにログイン
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/81d74011-a9c6-4ded-900b-bcb18588c191.png)

- VM側の準備
    * VMにログインすると、サーバーマネージャーが開くので、[リモートデスクトップ]の[無効]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/95ee2e61-65ff-48dc-bed3-f86666ccf230.png)
    * リモートデスクトップを有効にし、[OK]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/516a505a-34b0-4fb1-8933-a11ce91d6e54.png)
    * IEセキュリティ強化の構成を無効にする
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f722ba1e-64e2-425b-80c2-7bd610623acd.png)
    * PowerShellを開き、以下のコマンドでVMのIPアドレスとインターネット接続を確認する
    ```Powershell
    ipconfig
    Test-NetConnection www.microsoft.com -Port 443
    ```
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/6f452971-7629-4998-9112-af0d3a174055.png)
    * ホストマシンから先ほど確認したIPアドレスにリモートデスクトップで接続する
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/48c632e2-17ac-4376-b78f-99cbe9333a09.png)
    * PowerShellで以下のコマンドを実行する([Entra Private Accessの前提条件としてTLSv1.2の有効化が必要](https://learn.microsoft.com/ja-jp/entra/global-secure-access/how-to-configure-connectors))
```Powershell
If (-Not (Test-Path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client'))
{
    New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -Force | Out-Null
}
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -Name 'Enabled' -Value '1' -PropertyType 'DWord' -Force | Out-Null
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -Name 'DisabledByDefault' -Value '0' -PropertyType 'DWord' -Force | Out-Null

If (-Not (Test-Path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server'))
{
    New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -Force | Out-Null
}
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -Name 'Enabled' -Value '1' -PropertyType 'DWord' -Force | Out-Null
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -Name 'DisabledByDefault' -Value '0' -PropertyType 'DWord' -Force | Out-Null

If (-Not (Test-Path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319'))
{
    New-Item 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -Force | Out-Null
}
New-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -Name 'SystemDefaultTlsVersions' -Value '1' -PropertyType 'DWord' -Force | Out-Null
New-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -Name 'SchUseStrongCrypto' -Value '1' -PropertyType 'DWord' -Force | Out-Null

Write-Host 'TLS 1.2 has been enabled. You must restart the Windows Server for the changes to take effect.' -ForegroundColor Cyan
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/aeed239d-9ca0-4ea4-8f4b-d3401623d9ca.png)

# Entra Private Accessの設定
次に、Entra Private Accessの設定をしていきます。
今回は、一か月の無料試用版を使用します。

- Entra Private Accessの有効化
    * Entra 管理センター（entra.microsoft.com）にグローバル管理者でサインイン
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/2f31b0a2-a7b7-44cc-9ba2-8d12b85fc717.png)

    * 左ペインの[グローバル セキュア アクセス]→[ダッシュボード]をクリックして[無料で試す]をクリック
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/0fef085f-6647-4b87-932c-4eedd1b4d690.png)

    * アクティブ化タブが開くので、[無料試用版]→[アクティブ化]をクリック
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/340ee5d3-17df-442c-b6bb-40088c040a9f.png)

    * Microsoft Entra Suite -試用版画面が開くので、アカウント・支払いの設定などを行う
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/597b3961-300e-4a33-aebc-9843320c0749.png)

    * 完了画面
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/775d2dc8-399a-4f99-8647-6d67e3595d02.png)

    * Entra 管理センターに戻り、グローバル セキュア アクセスの[アクティブ化]をクリック
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/cf51633d-dc9b-45f4-8fa4-a37183e461a4.png)

- 接続先サーバーへのコネクタインストール
グローバル セキュア アクセスから接続先サーバーへ接続するためのコネクタを構成します。
    * アクティブ化が完了したら、左ペインの[グローバル セキュア アクセス]→[接続]→[コネクタとセンサー]をクリックし、[コネクタ サービスのダウンロード]→[規約に同意してダウンロード]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/70ab4ae5-e13f-47d5-8b47-a9004ffd002a.png)

    * ダウンロードしたインストーラーをVMにコピーし、実行→インストールを開始
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/eb8c2027-5193-4d4f-9caf-2716323a278e.png)

    * テナントの管理アカウントでログインする
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/540069b4-78a4-4211-8e21-0eab2641ebca.png)

    * インストールが終了すると以下の画面になるので、閉じる
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/6a08e35b-b3dd-47c4-9a0b-a895c576a981.png)

    * Entra 管理センターに戻り、コネクタが登録されていることを確認する
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/173d3f34-3f79-480c-a711-b41aa89b224d.png)

- トラフィック転送プロファイルの有効化
クライアントがどのIP(クイックアクセス）にアクセスしたときにトラフィックを転送するのかを設定します。
    * 左ペインの[グローバル セキュア アクセス]→[接続]→[トラフィック転送]をクリックし、[プライベートアクセス プロファイル]→を有効にする
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e7ce68fc-24b9-48db-922f-62508b58d7ce.png)
    * 「ユーザーおよびグループの割り当て」の[表示]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f4f0d157-f618-42e4-83fd-7ff5f801d0af.png)
    * [すべてのユーザーに割り当てる]をオンにする
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/bdd298b1-0dfc-4405-9903-e3dd4903fbc2.png)

- クイックアクセスの構成
接続先サーバーのIPアドレス/ポートと接続できるユーザーを指定します。
    * 左ペインの[グローバル セキュア アクセス]→[アプリケーション]→[クイックアクセス]をクリックし、「名前」と「コネクタグループ」を選択して、[＋クイック アクセスのアプリケーション セグメントを追加]をクリックする
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/672f0ee7-6b73-48f6-9559-4d79821bb1a5.png)
    * 「IPアドレス」に接続先サーバーのプライベートIPアドレス、「ポート」に[3389]を入力し、[適用]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/620b978b-4c9b-4d3b-a817-bc4fe47f89fb.png)
    * [保存]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/57e4c359-673b-47d6-b463-1aa210c312df.png)
    * [ユーザーとグループ]→[＋ユーザーまたはグループの追加]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/5ffffa50-9703-4aa7-8b92-a1935bba634b.png)
    * 接続を許可するユーザーを選択して[選択]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f1359d83-87f2-436d-adaf-ed9df0d7c0d6.png)
    * [割り当て]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/8127fc09-ae27-4ce5-b182-e4c604705e95.png)

- 条件付きアクセスの設定
接続できるユーザーや接続元のIP、デバイスなど接続時のセキュリティ設定を行います。
    * [条件付きアクセス]→[＋新しいポリシー]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ff07257d-4e66-4ffa-98e8-da355d08a1a3.png)
    * 条件付きアクセスを設定し、[保存]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/b81f1cb1-9b44-4677-909e-8ececa55607f.png)


# 自宅PCに Global Secure Access クライアントを導入
リモートサーバーへの接続を転送するためのクライアントを自宅PCにインストールします。

- 自宅PCをEntra参加する
接続元端末の条件としてEntra参加またはハイブリッドEntra参加が必須になります。

    * Windowsの設定画面を開き、[アカウント]→[職場または学校へのアクセス]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/d49e3601-595e-499a-8017-708008effedb.png)
    * [接続]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/49c6798b-1474-4388-b4cd-3f34fa0544c7.png)
    * [このデバイスを Microsoft EntraIDに参加させる]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/54b8514b-b817-4d45-b975-fd156cb81296.png)
    * クイックアクセスで許可してMicrosoftアカウントにログインする
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/3b1948f4-304c-435a-9fc3-26acd754fc58.png)
    * [参加する]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/c0c8bc5e-11ba-4494-aed0-f28c75dec8d8.png)
    * Entra参加は完了です
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e509ed77-2f6a-40e7-9f6b-cfb6ce8d11f6.png)

- Global Secure Accessクライアントのインストール

    * [グローバル セキュア アクセス]→[接続]→[クライアントのダウンロード]をクリックし、[クライアントのダウンロード]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/7ce966b2-168b-4f07-8d74-96e431efd7b3.png)

    * ダウンロードしたインストーラーを実行してインストールを開始
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/cb364a81-2255-4158-abd5-df6cdc3336b8.png)

    * インストールが完了したら閉じる
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/48a3f106-cbc5-49a0-a37b-6a700e4bc207.png)
    * 右下に以下のようなポップアップが出るので、[Sign in]をクリックしてMicrosoftアカウントにログイン
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/71e959cb-54b7-4435-8a95-17cbeebc6341.png)
    * 右下のタスクトレイに、GlobalSecureAccessClientのアイコンが出て「Connected」になっていればインストール完了です

# 動作確認
- RDPでサーバーのプライベートIPを指定して接続してみると、プライベートネットワークにつないでいなくてもインターネットにさえ繋がっていれば、接続することができます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/2eb4a21a-c0e6-4072-ae03-f885206c9605.png)

- 右下のタスクトレイのGlobalSecureAccessClientのアイコンを右クリックして[Advanced diagnostics]をクリック
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/0283fef9-b7ef-4452-a2ef-5a4dc78dde12.png)

- 開いた画面で[Fowarding profile]タブを開くと、指定したIPとポートが転送される設定が入っていることが確認できる
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/c428c6fd-6983-4f7e-8f79-e40278b1a8df.png)

- [Traffic]タブで、接続情報を確認できる
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/cb7b76d7-422f-4dc2-b320-d8fc9f73a9dd.png)

# （おまけ)Chrome リモートデスクトップで無料でリモート環境を構築
セキュリティは落ちますが、「Chrome リモートデスクトップ」で無料でリモート環境を構築することができるので、その方法をご紹介します。

- リモート接続先の設定
    * 下記のURLにアクセスし、googleアカウントでログイン後、Chromeリモートデスクトップのダウンロードボタンをクリック
    https://remotedesktop.google.com/access
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/1686df24-347f-423f-8bcb-61812929cec3.png)

    * [インストール]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/260ab77e-9b17-4867-a809-ae76a5de36fa.png)
    * [同意してインストール]をクリックして、インストーラーを起動
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/1f9bf0ee-78c5-4e37-8c1b-0ffbaac6a4b6.png)
    * 接続先のPC名を入力し、[次へ]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/43ee0cbf-b883-478b-84cb-38f0a4f8a9c0.png)
    * PINを入力し、[起動]をクリック
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/57228237-1879-45eb-aac9-13cfbc23a200.png)

- リモート接続元の設定
    * 接続元でも以下のURLに接続し、googleアカウントにログインする
    https://remotedesktop.google.com/access
    * リモート接続先のデバイスが表示されるので、クリックし、PINを入力する
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f1c949b1-dbd2-40e9-919e-7b8a48498f2b.png)
    * リモート先に接続できる。右側のボタンからctrl+alt+deleteを押してログインする(右shiftキーが反応しないため注意)
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/fcbbfdb6-7871-4c22-b318-6c9e700acccc.png)
    * 接続完了
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/9915cfa0-2f21-41be-8ba1-b727832f23ee.png)


    


