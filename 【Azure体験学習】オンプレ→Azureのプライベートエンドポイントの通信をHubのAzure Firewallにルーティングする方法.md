# はじめに
今回は**オンプレ→Azureのプライベートエンドポイントの通信をHubのAzure Firewallにルーティングする方法**をご紹介します。
現場でMicrosoftのサポートも駆使した上で、解決に時間を要した問題ですので、今後誰かが同じ問題に当たった時の役に立てれば幸いです。
 また、体験学習コーナーでは以下のような環境を作成します。クラウド技術の学習では自分で手を動かして学習することが非常に大事ですので、是非実施していただければと思います。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/63157c62-74b7-e1ad-7720-8f2ad5c8e5af.png)

# 対象読者

- Azureを使用する現場に従事している方
- Azureの資格勉強をしている方
- Azureに興味がある方
- クラウド技術に興味がある方

# 免責事項
Azureの従量課金アカウントが作成されていることが前提です。
ハンズオンにあたって料金がかかります。**今回の体験学習ではVPN GatewayやAzure Firewallなど放置しておくととんでもなく高額になるサービス**が含まれます。記事の最後に**リソースの削除の仕方**も解説しておりますので、ハンズオンが終了したらただちに**削除を実施**してもらうようお願いします。
また、Azureの画面に関しては、**2024年11月30日時点のもの**を使用しております。


# 使用サービス・技術

### ・Virtual Network(以下、VNet)
Azure内で仮想マシン（VM）やその他のリソース間で安全な通信を可能にするための論理的な分離ネットワークです。

### ・サブネット
VNet内でIPアドレス範囲をさらに細分化した論理区画です。サブネットを使うことで、VNet内のリソースをグループ化し、異なるセキュリティおよびネットワークポリシーを適用することが可能になります。

### ・ルートテーブル
ネットワークトラフィックのルーティングを制御するためのカスタマイズ可能なルーティングルールを定義するために使用されます

### ・VPN Gateway
Azure内の仮想ネットワーク（VNet）をオンプレミスのネットワークや他のVNetに接続するためのリソースです。これにより、安全なトンネルを通じてインターネットを介してデータを暗号化し、リモートネットワーク間で通信することが可能になります。

### ・ローカルネットワークゲートウェイ
オンプレミスのネットワーク環境（または他のクラウドサービスプロバイダーのネットワーク）とAzureの仮想ネットワーク（VNet）を接続する際に使用されるリソースです。ローカルネットワークゲートウェイは、オンプレミスネットワークのIPアドレス範囲やネットワークゲートウェイの情報（例えば、VPNデバイスのパブリックIPアドレス）をAzureに認識させる役割を担います。

### ・プライベートDNSゾーン
Azure仮想ネットワーク内のリソースに対してプライベートなDNS解決機能を提供するためのサービスです。この機能により、Azure VNet内のリソースがインターネットに公開されることなく、内部的に隔離された状態でDNS解決を実行できます。

### ・プライベートDNSリゾルバー
Azure仮想ネットワーク（VNet）内でプライベートDNSゾーンを利用して名前解決を行うためのサービスです。これにより、VNet内外のリソース間で安全で効率的なDNS解決を提供することができます。

### ・プライベートエンドポイント
Azureのパブリックサービス（例：Azure Storage、Azure SQL Database、Azure Cosmos DB など）に対してプライベートな接続を提供するためのネットワークインターフェースです。プライベートエンドポイントを使用することで、これらのサービスにインターネット経由ではなく、Azureの仮想ネットワーク（VNet）内のプライベートIPアドレスを介して安全にアクセスできるようになります。

### ・Azure Firewall
Azureの仮想ネットワークでネットワークトラフィックを管理し、セキュリティを強化するためのクラウド型のセキュリティサービスです。Azure Firewallは、統合されたステートフル（状態保存型）ネットワークファイアウォールとして機能し、トラフィックのフィルタリング、ログの取得、セキュリティポリシーの管理を一元的に行うことができます。

### ・ストレージアカウント
クラウドでデータを格納、管理、およびアクセスするためのAzure Storageサービスを提供する基本単位です。ストレージアカウントを作成することで、Azureが提供するさまざまなストレージサービスやデータサービスを利用するためのコンテナやデータベースをホストできます。

### ・VirtualMachine(以下、VM)
クラウド環境でのコンピューティングリソースを提供するサービスです。これは仮想サーバーとも呼ばれ、ユーザーが仮想環境で物理サーバーを使うようにコンピューティングリソースを利用できます。AzureのVMは、WindowsやLinuxのオペレーティングシステムを実行でき、Webサーバー、データベース、アプリケーションのホスト、開発テスト環境など、さまざまな用途に使用されます。

### ・Azure Bastion
Azure仮想ネットワーク内の仮想マシン（VM）にセキュアで無操作なRDP（リモートデスクトッププロトコル）およびSSH（セキュアシェル）接続を提供するマネージドサービスです。Bastionを使用することで、仮想マシンに直接パブリックIPアドレスを公開することなく、安全に管理できます。

# 1. 問題

### オンプレ→プライベートエンドポイントの通信をAzure Firewallにルーティングするルートテーブルを使っても、デフォルトの設定ではAzure Firewallにルーティングしてくれない。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/1e2c0b19-24a3-6c0a-f9ce-180c4671c865.png)




# 2. 解決策

### **プライベートエンドポイントがあるサブネットの「ネットワークポリシー」の設定でルートテーブルを有効化する**
上述の設定が無効になっている場合、プライベート エンドポイントが作成された際に自動で作られる内部的な経路が優先され、ルート テーブルで指定している経路ではなく、直接通信してしまうことが原因でした。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/cfc5aef2-cde0-621d-e58b-26c98d995728.png)



# 3. 体験学習

それでは実際に環境を構築して体験学習していきます。

### 3-1.環境作成
今回作成するリソースが非常に多いため、私のGitHubリポジトリから以下のようなネットワーク環境を一括で作成します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/a4038043-dc24-5366-c9fe-047722687406.png)

以下のリンクからGitHubに移動してください。
[shuhei0720のGitHubリポジトリへ移動](https://github.com/shuhei0720/Azure-Hub-Spoke)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/a59d5cc0-36ae-942f-1bfc-cf9ddd89bbe5.png)


下にスクロールし、[Deploy to Azure]ボタンをクリックしてください。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f516c616-c375-22d7-18a6-a39c977232d7.png)

すると、Azureのログイン画面に移動するため、読者様個人のアカウントでログインします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/450575cd-ad9b-e367-a7f2-bf95969e59f1.png)


ログインが完了すると、Azureのデプロイ画面に移動します。
最初の「基本」タブで以下のように入力します。「サブスクリプション」は各自のサブスクリプションを選択してください。入力後、[次へ]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/3a8cc62b-31c1-791b-2fd3-2095d5747742.png)

「Hub VNET」タブで以下のように入力し、[次へ]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/8cdb8793-70b9-0020-db16-c29da0084f6d.png)

「Spoke VNET」タブで以下のように入力し、[次へ]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/bc5d6a82-32c7-84f4-4367-9de16b524a93.png)

「オンプレミスのシミュレーション」タブで以下のように入力し、[次へ]をクリックします。
「サイト間共有キー」は任意のパスワードを入力してください。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e80d9b89-288b-b805-d99d-a5a3cc2b1ea9.png)

「仮想マシン」タブで以下のように入力し、[次へ]をクリックします。
仮想マシンのサイズは何でもいいですが、できれば画像のものと合わせてください。
管理者ユーザー名とパスワードは任意のものを入力してください。(VMのパスワードポリシーに引っかからないようにできるだけ複雑な方が良いです。)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/9f51d035-305c-67d6-96df-ac9b0d99629f.png)

「タグ」タブは特に入力せず[次へ]をクリックしてください。

「確認と作成」タブで[作成]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/acc620e1-b4c7-3488-c468-addb85b23e5c.png)

すると、以下のような画面になります。デプロイに1時間程度かかるので1時間ほど待ってから画面を見に来てください。![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ca73ce44-5576-3a1c-3310-37441e83ca0e.png)

以下の画面になればデプロイ成功です。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ebb35f24-978a-8970-883e-f22900c46390.png)

以上で、「3-1.環境作成」は完了です。


### 3-2.診断設定有効化
Azure Firewallで通信ログを取るために、診断設定を有効化します。
最初にログを格納する箱を作成します。
ポータル画面で「Log」と検索し[LogAnalyticsワークスペース]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/2ae36c0f-b687-c7dc-1f5f-cb8ae4f2cabf.png)

[作成]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/fa29a590-299a-3b42-d01a-714f0e160c15.png)

以下のように入力し、[確認と作成]→[作成]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/b32ac5a7-287b-c6a1-8580-5ee168142f39.png)

ログを格納する箱は作成完了です。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/874ca5a1-8743-c789-ff91-547467c9aa06.png)

次にAzure Firewallの診断設定を有効化していきます。
ポータル画面で「ファイアウォール」と検索し、[ファイアウォール]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e965371b-2a7c-fcc3-2c62-392d78cc1c94.png)

[Firewall-Hub]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/5ccffa92-7f8e-e8aa-53a7-7e7379010251.png)

左ペインから[診断設定]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/772f01b6-87ed-4d5a-2fe0-94a12261c66c.png)

[診断設定を追加]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/64d4730b-ea66-8b39-dc16-c485eb6a8c11.png)

以下のように入力し、［保存］をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/31815887-53a4-6558-fea5-17e754680e6c.png)

以上で、「3-2.診断設定有効化」は完了です。

### 3-3.ストレージアカウントの設定

Azureポータルで「ストレージ」と検索し、[ストレージアカウント]を選択します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/fef5bcd5-3c3d-65b6-97fe-f37935479003.png)

[作成]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/66c6139d-f5e2-88c0-5bad-6ad5ce6f4c26.png)

「基本」タブで以下のように入力し、[次へ]をクリックします。
※ストレージアカウント名は世界で一意になる必要があります。適宜変更してください。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/c3550e1d-9e37-992d-cb65-9e1c86303b0c.png)

「詳細」タブはそのままで[次へ]をクリックします。

「ネットワークタブ」で以下のように入力し、[プライベートエンドポイントの追加」をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/8ce4453f-d0b8-9d26-93ce-616c53422a44.png)

「プライベートエンドポイントの作成」画面で以下のように入力し、[OK]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/7bbe75c4-8b85-a762-238a-01a2ffcea3d4.png)

元の画面に戻るので[次へ]をクリックします。

「データ保護」タブで以下のように入力し、[次へ]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/0e30eb2b-c347-46a4-aa1b-84426af433d5.png)

その他のタブはデフォルトのままで[確認と作成]を押してください。ここで検証に失敗した場合、ストレージアカウント名が世界ですでに使用されている可能性がありますので、変更してください。
検証に成功したら、[作成]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/47d4cce8-c23a-7918-3f33-2c8e8da27271.png)

デプロイが完了したら[リソースに移動]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/fdcdb46d-e2b3-aef5-7baf-55b6dd1059c5.png)

「概要」画面で[機能]タブをクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/9dff35a9-3729-e4ee-5204-1aa03ffa48a3.png)

[静的なWebサイト]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/28fddc17-7315-8cb1-c396-2b96a809cb66.png)

以下のように入力し、[保存]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/a52037a4-d89b-db77-117b-2545852b65bb.png)

「プライマリエンドポイント」のURLをメモ帳にコピーしておきましょう。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ec966d04-cd83-2761-72d6-2b4a8968fc8a.png)


[概要]画面に戻ります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/069474b7-8792-dabe-e52f-87436a780803.png)

「概要」画面で[アップロード]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/0f67f878-3b7b-3e27-61fd-5791e50c0322.png)

「BLOBのアップロード」画面で[ファイルの参照]をクリックし、この記事に添付している[index.html]ファイル保存し、それを選択します。
※この記事に添付しているindex.htmlは私がテキトウに作成したhtmlファイルです。htmlファイルで名前がindex.htmlであればなんでもいいです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/4246cf34-064e-cc60-e72c-fe7b83bb92fb.png)

「BLOBのアップロード」画面に戻り、「既存のコンテナーを選択する」で[$web]を選択し、[アップロード]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/de67c05e-de0b-a330-9e85-7f4793dda8f7.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/93484b34-d6f2-727e-279a-2f89c8559db2.png)

試しに、先ほどメモ帳にコピーしたURLをブラウザで入力してみましょう。
以下のようにWebサイトにアクセスできるはずです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/6d6b7aea-5020-24f0-f954-c716981c897c.png)

次に、このWebサイトへのアクセスをプライベートエンドポイントからのみアクセスできるように変更します。
ストレージアカウントの画面で左ペインの「セキュリティとネットワーク」を開き、[ネットワーク]をクリックします。![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/cbbc4edb-3717-08b7-067d-a60d61696174.png)

「ファイアウォールと仮想ネットワーク」の画面で「パブリックネットワークアクセス」を[無効]にし、[保存]をクリックします。![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/a49976a3-3f4f-cc86-8312-6f0148530106.png)

保存出来たら、先ほどアクセスしたURLにもう一度アクセスしてみましょう。個人のPCからのアクセスはパブリックインターネットを通るので、拒否されるはずです。
※先ほどと同じように表示された場合キャッシュが残っています。シークレットブラウザでアクセスしてみましょう。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/bd0b2e1c-a805-2b20-0c9f-44a3c848ec02.png)

以上で、「3-3.ストレージアカウントの設定」は完了です！

### 3-4.プライベートDNSの設定


次にプライベートエンドポイントのDNSの設定を行っていきます。
まず、Spoke VnetのDNSサーバーを設定します。
ポータル画面で「vnet」と検索し、[仮想ネットワーク]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/9f06fba6-f2ea-1bc3-7625-77ec6613742b.png)

[VNET-Spoke1]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/d328bccd-93ae-754b-56e9-1a4dc0914f4e.png)

左ペインから[DNSサーバー]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/8607dd4d-b5d1-4d59-1fde-e2e26e0f9ec0.png)

DNSサーバーを[規定(Azure提供)]を選択し、[保存]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/dd7646c2-617f-765a-c021-0a39ffd6c9a2.png)

次に、インバウンドエンドポイント専用のサブネットを作成します。
VNET-Spoke1の画面の左ペインから[サブネット]を選択します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/edd22fa1-0150-5c4a-b537-e16b73c6bbc8.png)

[+ サブネット]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/21df05c6-df5d-28aa-5b9a-8e177aa5698c.png)

以下のように入力し、[追加]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/31751f8e-2129-0c3c-11a9-6699cfd83bf0.png)

サブネット追加は完了です。

次に、プライベートDNSリゾルバーを作成していきます。
ポータル画面で「DNS」と検索し、[DNS Private Resolver]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/10b16fe9-19d1-71d7-418b-85b64228b215.png)

[作成]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e3caef54-5f4d-9ebc-473f-36f89720723d.png)

「基本」タブで以下のように入力し、[次へ]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/68519744-1e38-d128-9887-11419a2690f1.png)

「受信エンドポイント」タブで[+ エンドポイントの追加]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/02759700-6d35-104a-9e3b-22d40f1e8bce.png)

「受信エンドポイントの追加」画面で以下のように入力し、[保存]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/2cc5041a-8718-be94-1095-d3594b05d50b.png)

元の画面に戻るので、[確認および作成]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/fbef70b9-6240-9e31-ff28-9c02939f4261.png)

検証が成功したら、[作成]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/a47c15fb-ca91-abad-fcb7-a352b62b38de.png)

以下の画面のようになれば、DNSリゾルバーの作成は完了です。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f63538ec-cd34-6ca7-4b78-3f4f3e19c92b.png)


### 3-5.疑似オンプレミスのDNSサーバーの設定

まず、疑似オンプレミスのVNetのDNSサーバーを設定します。
ポータル画面から「vnet」で検索し、[仮想ネットワーク]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/2538a96b-460d-8c00-c7b6-b4eaaebaa6c3.png)

[VNET-OnPrem]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/30ca3f5d-bd8f-c41c-f0a8-208c10cc4c7f.png)

左ペインから[DNSサーバー]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/1c0e5a5d-2b27-71f5-fec3-59d2600c9ced.png)

以下のように入力し、[保存]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/bf9f38d2-5546-4358-ff73-04e472dd7aba.png)


次に、疑似オンプレミスVNet内のVMにDNSサーバーの役割を追加し、条件付きフォワーダーの設定を行います。
ポータル画面から「virtual」と検索し、[Virtual Machines]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/1415f0a5-b9d7-63df-6929-4f50b4986909.png)

[VM-OnPrem]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/06397f21-fb8a-a288-b53e-3ee2d5bd9246.png)

左ペインから[Bastion]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/4fc956b3-6655-6342-f3ff-8d556b609bb4.png)

環境作成時に指定したユーザーとパスワードを入力し、[接続]をクリックします。
※接続時に「クリップボードを許可しますか」と聞かれた場合、[はい]を選択します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/266b5127-0463-4684-eac4-e006d279e769.png)

接続すると、Server Managerが開いていますので、右上の[Manage]→[Add Roles and Features]を選択します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/d432f1df-5331-5cd3-6586-a630aa0e8dff.png)

[Next >]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/b0f3996d-6fcd-44fa-20fc-3908e2ea0d87.png)

そのまま[Next >]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/d3c5c9b6-6397-196c-54f1-e0c2e39fb0d6.png)

そのまま[Next >]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/b83ce5ea-ba16-dea8-3cf2-7b1406590405.png)

[DNS Server]のチェックボックスを選択します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ea0e1b26-5f07-af4e-5fc0-1a14d1a60eb5.png)

開いた画面でそのまま[Add Features]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/4251b93e-2ef2-a9d8-c718-b18e059dbfd4.png)

開いた画面でそのまま[Continue]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f2111fc8-0001-314e-31aa-7dc7a47318e6.png)

元の画面に戻るので、[Next >]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e2a6ae39-42c6-1f1b-7793-d1a981a546a4.png)

次の画面でそのまま[Next >]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/783f6f95-f05a-17f7-4f64-d607fdd508e7.png)

そのまま[Next >]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/42a80832-6582-35c6-84de-a0caeaab50a8.png)

そのまま[install]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/9a3d0629-2ec5-af46-1128-c40e0cf1bf27.png)

インストールが完了したら[Close]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/c8dea58c-741d-d2a1-946f-5989437df0e9.png)

Server Managerの画面に戻りますので、右上の[Tools]→[DNS]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/5b784f87-7d9c-8be5-d3d5-1c317f1cbc22.png)

DNS Managerの画面で、[VM-OnPrem]→[Conditional Forwarders]→[New Conditional Forwarders]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/b81e0178-552a-e76c-7d8a-41299c2c1d8f.png)

以下のように設定し、[OK]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/2bce0481-efeb-8c81-95bd-436d11efb72d.png)


VMを再起動します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e16be54d-a3df-3494-b7a9-3a619a822971.png)

VMが起動したら、名前解決をテストしてみます。VM内でコマンドプロンプトを開きます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/5fcab721-e453-66f6-3ab5-afc5abb2720f.png)

先ほどメモしたURLからnslookupのコマンドを作成します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/0d17a681-9798-2756-8399-0a1043670f24.png)

VM内でメモ帳を起動し、そこに貼り付けます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/333a69c8-09f9-0cc1-e632-7a57754e2cb2.png)

先ほど作成したnslookupコマンドを実行してみます。
10.0.1.5が返ってきていれば名前解決は成功しています。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/87df7763-05ff-20cf-0c09-f8a90b9efbbd.png)

続いて、VM内でブラウザを開きメモにあるURLを検索してみましょう。
以下のようにプライベートエンドポイント経由でアクセスできました。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/372ded3a-e48a-0da7-0499-5a0c87fe32c3.png)

以上で、「3-5.疑似オンプレミスのDNSサーバーの設定」は完了です。

### 3-6.Azure Firewallのログの確認

さて、環境が整ったところで、オンプレ→SpokeのHTTPS通信がAzure Firewallを経由しているか、ログを確認してみましょう。

Azureポータルから[ファイア]と検索し、[ファイアウォール]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/066a0f39-d672-962c-19c6-e3c9339a68db.png)

[Firewall-Hub]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/fd13129b-62fe-48bd-090a-c43bfca4726f.png)

左ペインから、[ログ]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/91dc610c-956e-fad7-35c7-fa59c186f907.png)

開いた画面で、以下のクエリを実行してください。
※以下の画像と画面の表示が違う場合は、[KQLモード]に切り替えてください。
```
AZFWNetworkRule 
| where DestinationIp contains "10.0.1.5"
```
結果が表示されずログがない（=Azure Firewallを通っていない)ことがわかります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/2dc71204-e5c2-361e-8e39-b3a6a484d1de.png)


### 3-7.サブネットの設定（今回の肝)

それでは、この通信をFirewallを通すための設定を行います。

ポータル画面で「vnet」と検索し、[仮想ネットワーク]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/9f06fba6-f2ea-1bc3-7625-77ec6613742b.png)

[VNET-Spoke1]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/d328bccd-93ae-754b-56e9-1a4dc0914f4e.png)

左ペインから[プライベートエンドポイント]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/2b8064fc-d81c-0760-e8fc-eca67324f15e.png)

「プライベートエンドポイント」画面で「プライベートエンドポイントネットワークポリシー」が[Disabled]になっていると思うので、そちらをクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f9bd3b54-adc5-6431-c7b6-4503d4b220ee.png)

「サブネットの編集」画面で下にスクロールし、「プライベートエンドポイントネットワークポリシー」を[ルートテーブル]に変更し、[保存]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/65ebe8d5-044d-4a95-4062-f86578cdb35f.png)

もとの画面に戻るので、「プライベートエンドポイントネットワークポリシー」が「RouteTableEnabled」に代わっていることを確認してください。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ce1540f7-f8f6-35fd-1fd6-a4493a9d16a9.png)

さて、サブネットの設定が完了したので、先ほどのVMの画面に戻り、新しいプライベートウィンドウからもう一度ストレージアカウントに検索してみましょう。
タスクバーのMicrosoft Edgeのアイコンを右クリックし、[New InPrivate window]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ceed242b-46f3-6ad2-c7b4-e85e7298821b.png)

先ほどと同じURLを検索します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/58d17aee-2b4e-c674-4da6-3ff9499dbd97.png)
接続できました。

次にまた先ほどと同じようにAzure Firewallのログを確認します。(ログに反映されるまで5～10分かかるのでしばらく待ってください。)
すると、ログが出てきます。これでAzure Firewallを通るようになったことが確認できました。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/eda765e3-3f8e-879b-6744-74269a1d934b.png)

以上で、すべてのハンズオンは完了です。

### 3-9.リソースの削除
**※高額なリソースが含まれるので、途中で中断する場合も必ず削除してください。**
ハンズオンが終了したら、課金が発生しないようにリソースを削除しておきましょう。

Azureポータルから[リソースグループ]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e0125b4a-e8d9-927d-3dbd-445dc29f80a2.png)

[rg-hub]、[rg-onprem]、[rg-spoke1]を削除していきます。
まず、[rg-hub]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/a8509db7-53a9-26de-6f92-a8d2a5b62ca1.png)

[リソースグループの削除]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/8f91cf34-7236-1d56-3fb7-b79afe7ed21e.png)

「リソースグループの削除」画面でリソースグループ名を入力し、[削除]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/b9c8d08f-8ead-873a-53c0-2f20edf4b60a.png)

「削除の確認」で[削除]をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f245da77-d675-470b-2dc0-4d700fd9c725.png)

同様に、[rg-onprem]、[rg-spoke1]も削除します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/4b29fbab-f4b2-42b7-9d18-d15b9f04e28a.png)

削除には時間がかかります。
しばらくすると以下のように通知が来るのでリソースグループの画面から[rg-hub]、[rg-onprem]、[rg-spoke1]が削除されたことを確認しましょう。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/a4c86a3b-10ff-9369-c6c8-2f47d56a5fed.png)


以上で、リソースグループの削除は完了です。

## おわりに

今回は、オンプレ→Azureのプライベートエンドポイントの通信をHubのAzure Firewallにルーティングする方法を紹介しました。
複雑なアーキテクチャになってくると原因を特定するのがかなり大変です。
今回もルートテーブル等の設定に注目していたら、なかなか気づかないような点に解決策があるので情報がないと苦労します。
現場で私と同じような機会があれば、是非この記事の設定を参考にしていただければ幸いです。

