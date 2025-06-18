今回は、Azure(Microsoft)のDNSについて深堀します。
ハンズオンで体験しながら見ていきましょう。

## AzureのプライベートDNS
AzureのVNet上のVM等がマネージドサービスを使って名前解決を行う方法は以下のパターンが存在します。
１．VNetの既定のDNS(168.63.129.16)
２．AzureプライベートDNSゾーン
３．Azure DNS Private Resolver(アウトバウンドエンドポイント)
４．Azure DNS Private Resolver(インバウンドエンドポイント)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/6ab6667c-065e-482a-b9d8-efcc13bcc88c.png)


これらをハンズオンで見ていきたいと思います。

## １．VNetの既定のDNS(168.63.129.16)
VNetでは規定で、Azureが基盤として提供するDNSサーバー(168.63.129.16)をDNSサーバーとして指定されています。以下は新規作成した状態のVNetのDNSサーバーの設定です。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/1e6616f0-7b2d-4110-a13b-696d2242f662.png)

規定のDNSのIPアドレスである、168.63.129.16はMicrosoftが所有する特殊なパブリックIPアドレスです。VMなどのリソースが正常稼働するためのエンドポイントとしても利用されています。

VNet内の仮想マシンは既定のDNSを使ってインターネットへ名前解決を行うことができます。
インターネットへの名前解決は既定のDNSからAzureが管理する再帰リゾルバーに転送されます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/5e6d6f77-6627-4ffd-bad4-95f2d0a6189a.png)

### １．ハンズオン ~VNetの既定のDNS~
仮想マシンのDNSがどのように動作するか体験してみましょう。
AzureポータルからVMの作成画面に移動します。
「基本」タブで以下のパラメーターを入力し、「確認及び作成」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/92467e1c-a1aa-42b9-8c49-f8f754a6b284.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f2a13c90-f801-4ab5-a22d-27970ac67fae.png)

「作成」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/7884a96f-aabf-4d6e-8684-4ee7650976a8.png)

「リソースに移動」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/0ae3b24e-0d53-44ce-9ca5-d48d197534e5.png)

「接続」→「Bastion」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/68e4f361-2991-491a-a80a-20bc041f1230.png)

「Bastionのデプロイ」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/01828d5b-18ea-4cd4-ae45-5968fcfa716b.png)

Bastionのデプロイが完了すると、以下のような画面になるので、ユーザー名とパスワードを入力し、「接続」押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/7d13d6f3-c8d4-418c-a899-6be308da3de1.png)

新しいブラウザが開き、VMの接続画面が表示されます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/036cd892-3b3f-43ae-9faa-fc98a198b2ab.png)

サーバーマネージャーを閉じて、コマンドプロンプトを開き、以下のコマンドを実行します。
```
ipconfig /all
```
DNS Serversを確認すると「168.63.129.16」となっていることが確認できます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/4bf6289c-b687-4cea-8722-ab8067cc56a9.png)

では、以下のコマンドを実行しインターネットに名前解決してみましょう。
```
nslookup google.com
```
すると、「168.63.129.16」が名前解決をしてくれているのがわかります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e7d5c17d-df73-463a-91e3-f87878d6f99b.png)

さて、次はVMのDNSサーバーを変更してみましょう。
VMのDNSサーバーはVNetからDHCPで配布されています。**Microsoftから動作が保証されていないので、VMの中からDNSサーバーの設定を変更するのはやめましょう。次から変更の手順を説明していきます。

Azureポータルに戻り、VMの画面の「概要」を押下し、概要画面の「仮想ネットワーク/サブネット」欄に表示されている「vm-dns-test-vnet/default」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/abf99462-637c-4c97-a151-f73a7aa8b429.png)

「DNSサーバーを押下」
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/5d5e08a9-f49c-41c3-9a1c-a9c6f9154c06.png)

「カスタム」を選択し、DNSサーバーのIPアドレスとして、「8.8.8.8」を入力し、「保存」を押下します。
ここのVNetのDNSサーバーを設定すると、AzureのDHCPでVNet内のVM等にDNSサーバーの設定が配布されます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/dc170c8e-6bc4-44ac-8750-b4dbfe0e75d0.png)

先ほど接続したVMの画面に戻り、コマンドプロンプトで以下のコマンドを実行してDNSサーバーの設定を更新します。
```
ipconfig /renew
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/140c74d9-ff03-4b9f-8c5a-8d50fbca86bd.png)


以下のコマンドを実行します。すると、DNS Serversが8.8.8.8に変更されていることが分かります。
```
ipconfig /all
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/515a8b24-d89e-4ff5-8ed3-5720ee56d3ea.png)

以下のコマンドもう一度インターネットへ名前解決してみると、8.8.8.8が返してきたことが分かります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/2d551e88-cf6c-49c7-814d-b7fac9381bff.png)

ハンズオンは以上で終了ですが、後のハンズオンでもVNetやVMを使用するのでDNSサーバーの設定を元に戻しておきます。
AzureポータルのVNetのDNSサーバーの設定画面で「規定(Azure提供」を選択し、「保存」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/4035bfa4-986d-4fc6-a7fb-c6f99cc8caa8.png)

接続しているVMの画面に戻り以下のコマンドを実行します。
```
ipconfig /renew
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/d58860d7-bb60-48df-a359-61c02bac4034.png)


以上で「１．ハンズオン ~VNetの既定のDNS~」は終了です。

## ２．AzureプライベートDNSゾーン
AzureプライベートDNSゾーンは、Azureが提供するプライベートの権威DNSサーバーです。
プライベートDNSゾーンをVNetにリンクすることで、Azure既定のDNSがそのゾーンを参照して名前解決できるようになります。
プライベートDNSゾーンの名前解決は基本はVNet内部のみですが、後に紹介するAzure DNS Private Resolverを使用するとオンプレミス等からも名前解決できるようになります。
PaaSへのプライベート接続を行うための、「プライベートエンドポイント」と組み合わせて利用することが多いです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ef3df427-f9e4-404b-8885-6144a5f91567.png)

### ２．ハンズオン ~AzureプライベートDNSゾーン~
それでは、AzureプライベートDNSゾーンを実装してみましょう。

AzureポータルでプライベートDNSゾーンの画面に移動し、「作成」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ee43d38f-856f-444d-b2f9-3de281867992.png)

以下のようにパラメーターを入力し、「Virtual Network Links」タブに移動します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/7a4535db-af13-48f6-8c58-f8938e2cae75.png)

「仮想ネットワークリンクを追加します」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/bde04171-19b5-433e-a10d-7b5825faebe7.png)

開いたタブで以下のようにパラメーターを入力し、「保存」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/a532f441-226c-4a18-8f5e-4e87a9fdb8d7.png)

「確認及び作成」を押下し、「作成」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e6a8bad9-9f24-4522-9113-c7e5a5955cc2.png)

以上で、プライベートDNSゾーンと仮想ネットワークへのリンクは作成完了です。
作成が完了したら、「リソースに移動」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/0fa29e56-e32b-465f-87cd-c16717ccdf5c.png)

「設定」→「DNSの管理」を押下し、「追加」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/06949a40-f027-4cba-8356-027eeb6dbece.png)

「レコードセットの追加」画面で以下のように入力し、「追加」を押下します。
※IPアドレスはなんでもいいです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/b9d47ece-33b7-400d-a9d2-428167cd9bc8.png)

レコードが作成されました。このゾーンが参照できれば、「www.test.com」は「192.168.1.1」に解決されるはずです。先ほど作成したVMから確認していきます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/9e3fce43-3484-4f00-92f9-80683e8ad1c3.png)

接続しているVMの画面に移動し、コマンドプロンプトで以下のコマンドを実行します。
すると、先ほどプライベートDNSゾーンに登録した「www.test.com」が「192.168.1.1」に解決されていることがわかります。
```
nslookup www.test.com
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/cede26c6-7d14-402a-b2c7-e50e072d6c08.png)

次に、PaaSのプライベートエンドポイントでプライベートDNSゾーンが使われるパターンも試していきましょう。
Azureポータルで「ストレージアカウント」と検索し、ストレージアカウントの画面で「作成」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/d08aa8bc-c183-434d-9222-a0ad93f8b646.png)

「基本」タブで以下のようにパラメーターを入力し、「ネットワーク」タブに移動します。
※ストレージアカウント名は全国で一意にする必要があるので、末尾に好きな数字を付け加えてください。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/c815a502-2485-48d4-bb76-4ce8df5b0155.png)

「ネットワーク」タブで以下のようにパラメーターを入力し、「プライベートエンドポイントの追加」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/5f096551-f68a-491b-abb3-12035425a570.png)

「プライベートエンドポイントの作成」タブで以下のようにパラメーターを入力し、「OK」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/522764bf-7bcb-4665-bbfc-67ceb6d2a45c.png)

「レビューと作成」→「作成」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/99430494-b0de-4e74-8375-19280a84b97c.png)

作成が完了したら「リソースグループ」の「rg-dns-test」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/6934e74f-5d2b-446e-936a-ab14e12005ff.png)

プライベートDNSゾーン「privatelink.blob.core.windows.net」が作成されているのが分かります。
「privatelink.blob.core.windows.net」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/76d6d294-113c-42a1-8055-54a2c4d9fb03.png)

「DNSの管理」→「レコードセット」を確認すると、自動的にAレコードが作成されていることを確認します。
※レコードの名前は作成したストレージアカウント名になっています。
これにより、プライベートDNSゾーンを使って「<ストレージアカウント名.blob.core.windows.net」をプライベートエンドポイントのIPアドレスに解決できるようになります。VMから見てみましょう。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/9c529e98-1d48-4145-8f3e-b16551d42038.png)

接続していたVMの画面に戻り、コマンドプロンプトで以下のコマンドを実行します。
※ストレージアカウント名は各自作成したストレージアカウント名で置き換えてください。
すると、プライベートエンドポイントのIPアドレスが名前解決されています。
これにより、プライベートIPアドレスでPaaSに接続できるようになります。
```
nslookup <ストレージアカウント名.blob.core.windows.net
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/733f6f1e-555c-4ab2-b2b8-9fdb1f419cdf.png)

以上で、「２．ハンズオン ~AzureプライベートDNSゾーン~」は終了です。

## ３．Azure DNS Private Resolver(アウトバウンドエンドポイント)
Azure DNS Private Resolverはサーバーを持たずに、オンプレミスとAzureとの相互の名前解決を実装できるサービスです。
Azure DNS Private Resolverは2つのコンポーネントがあり、その一つがアウトバウンドエンドポイントです。
アウトバウンドエンドポイントは、Azure内のリソースからオンプレミスのゾーンに対するクエリを受信し、指定したDNSサーバーに転送することができます。
具体的には転送ルールセットというリソースに、転送するドメインと転送先のDNSサーバーのIPアドレスを設定し、転送対象のVNetにリンクすることで名前解決を転送します。

以下に、アウトバウンドエンドポイントを利用する際の名前解決の順番を解説します。
３-１.DNSクライアントとなる仮想マシンからアウトバウンドエンドポイントに名前解決が送られます。
３-２.名前解決要求が既定のDNSに送られます。
３-３.DNSクエリと一致するプライベートDNSゾーンがVNetにリンクしている場合、そのゾーンを参照します。
３-４.DNSクエリと一致するプライベートDNSゾーンがVNetにリンクしていない場合、VNetにリンクされた転送ルールセットを参照します。そこにDNSクエリに一致するゾーンと宛先が存在する場合、その宛先に名前解決を転送します。
３-５.転送ルールセットがリンクされていない、またはクエリに該当するゾーンが転送ルールセットに存在しない場合、再帰的リゾルバに転送して名前解決を行います。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/70ec3de1-3c5e-43b7-bc15-1e00a2d633be.png)

### ３．ハンズオン ~Azure DNS Private Resolver(アウトバウンドエンドポイント)~
それでは、アウトバウンドエンドポイントをハンズオンで体験しましょう。
まず、疑似的にオンプレミスの環境を再現していきます。
今回は、オンプレミスを模倣したVNetを作成し、その中にADドメインコントローラーを作成します。※ドメインコントローラーはDNSサーバーとして動作します。
そして、オンプレミスを模倣したVNetと先程作成したVNetをピアリングします。

AzureポータルからVMの作成画面に移動します。
「基本」タブで以下のパラメーターを入力し、「次へ：ディスク>」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/102f2a6a-f311-435d-9f8a-ac530169a016.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e1b6d3a6-4d1e-4a30-b95b-283be55f5ed4.png)

「ディスク」タブでそのまま「次へ：ネットワーク>」を押下します。

「仮想ネットワーク」の「新規作成」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/765f0d64-dc40-45fc-ae47-15737b8e653f.png)

「仮想ネットワークの作成」画面で以下のように入力し、「OK」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/bd21e74c-8830-4194-9631-cae9f7f54e69.png)

「確認及び作成」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/88326470-2a0c-4338-9185-6c867338e832.png)

「作成」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/c4950e8b-df28-4c3d-ac12-be9df7ea1479.png)

「リソースに移動」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/cd0e7553-3d42-487a-b17b-3d16d96d12a2.png)

「仮想ネットワーク/サブネット」の「onpre-vnet/default」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/c17a9fc0-0a79-488a-a7eb-3609caa98187.png)

「設定」配下の「ピアリング」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/23b3f759-1724-4a7c-b844-d26980d21c1e.png)

「追加」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/7f6462fc-92b1-4536-af73-0293cb23f4e4.png)

以下のようにパラメーターを入力し、「追加」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/7be0ae47-1b7c-46d5-910b-4eba25c2e695.png)

次に「設定」配下の「DNSサーバー」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/71e1e816-dfe5-450c-8bc8-18edeefab2c8.png)

以下のように入力し、「保存」を押下します。
※ADDSとしてデプロイしたVMをDNSサーバーとして使うための設定です。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/eff934d3-84b9-40d6-be75-fc5f8122f05b.png)

画面上部の「vm-adds-dns-test」を押下し、仮想マシンの画面に戻ります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/4c2fab2c-b5b7-485d-81d5-62b6b20c3af4.png)

「接続」→「Bastion」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/64efe626-8190-435e-a24f-4aab403c2233.png)

接続情報を入力し、「接続」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/66e6c2bb-7a7d-454a-b122-9289c33a7ccf.png)

接続するとサーバーマネージャーが開いているので、「Manage」→「Add Role and Features」を押下します。
※ここからADDSのセットアップをしていきます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/0b10f56c-46f3-491a-9b98-84cb60ed2dcc.png)

役割と機能の追加ウィザードが起動します。そのまま「Next >」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/b4bdb6da-fbc3-486f-8987-882cdd89aad2.png)

そのまま「Next >」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/cb3e2f6d-a1a2-481e-9e1d-eeca9d1a335d.png)

そのまま「Next >」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ba3e79db-03cd-4b87-b300-be760fd73d79.png)

「Active Directory Domain Services」にチェックを入れます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f17623d7-7c39-4860-8cab-77a77e0082d0.png)

「Include management tools」にチェックを入れ、「Add Features」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/fa6337b4-9bd2-4c80-89ea-1e01bf516f8d.png)

「Active Directoryドメイン サービス」にチェックが入っていることを確認し、「Next >」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/c2517c17-4ea8-4747-82c4-b5d135578a40.png)

そのまま「Next >」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/1603ed94-e125-40ae-befd-a01e5c7b70ce.png)

「Active Directoryドメイン サービス」の画面が表示されます。そのまま「Next >」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/191ead19-19d5-4e03-882a-253fbb54ba28.png)

内容を確認し、「Install」を押下します。機能のインストールが開始されます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/74a47455-68fe-4526-ac8f-2042e6a1095b.png)

5分程待つとインストールが完了します。インストール完了後、「Close」をクリックする。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/fffd6909-bfdb-40b7-a217-e46da9e43034.png)

サーバーマネージャーに戻り、「Promote this server to a domain controller」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/2b898e98-5ff4-40be-8c8b-60ec82111114.png)

「Add a New forest」にチェックを入れ、以下のようにドメイン名を入力し、「Next >」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/3ee7eb05-4aa6-4c67-9723-b6a63c4f8e9e.png)

以下のように入力し、「Next >」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/192ac1c0-9551-493e-a687-50046af36c72.png)

そのまま「Next >」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f5b827bf-4078-4a8b-814a-2beabd78488d.png)

NetBIOS名が入力されていることを確認し、「Next >」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/595ee2ff-7c05-4d82-8b07-39d7d01847ba.png)

「データベースのフォルダー」「ログファイルのフォルダー」「SYSVOLフォルダー」に値が入っていることを確認し、「Next >」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ea1c237e-57e1-4489-a46b-0fed7ce63627.png)

オプションの確認画面に遷移します。そのまま「Next >」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/bbd1e3bb-56f8-4adc-b436-ba0e9abd0ad9.png)

「All prereauisite checks passed successfully.」であることを確認し、「Install」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/aa540d14-713c-440d-ba22-e3107bbfe919.png)

ポップアップが表示されるので、「閉じる」を押下します。すると自動で再起動されるので、少し待ってから仮想マシンに再接続します。

以上で、ADDSの準備は完了です。
それでは、Azure側からオンプレADドメインの「onpre.com」の名前解決をできるようにしていきましょう。

Azureポータルで「DNS private resolvers」と検索し、「DNS private resolvers」の画面に移動し、「作成」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/c28d536c-f887-4e14-9153-4791eddca09f.png)

以下のように入力し、「Next：Inbound Endpoints」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/b2aacb05-dedf-469e-a95c-7dec5db48a99.png)

「Add an endpoint」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/1bc60f65-6a1c-4ef7-93ad-ffde165edae3.png)

「Subnet」の「Create new」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e1972164-10ec-4777-985c-b5f3f5f29733.png)

以下のように入力し、「Create」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/b706d7fa-fc52-4dc4-8b30-4bcb907ca3a4.png)

以下のように入力し、「Save」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ea313c10-2799-49f4-8ba2-fbe5df53962a.png)

「Next：Outbound Endpoints」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/662e3cfd-98b7-4b49-861c-9987610f2961.png)

「Add an endpoint」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/cc8a7b82-ef8f-4775-9ab3-3be6026b1183.png)

「Subnet」の「Create new」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/5162a126-887b-409d-a08a-cdfb1d622314.png)

以下のように入力し、「Create」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/4d578c56-87f1-47c6-b622-baa248b217ca.png)

以下のように入力し、「Save」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/992325e6-9a6a-4b3a-a1d3-8a4525358698.png)

「Next：Ruleset」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/3dda8df7-2574-436e-b9a5-cde44e62354e.png)

「Add a ruleset」にチェックを入れます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/36c4da3f-9f56-4924-b75e-26c2c65e7292.png)

以下のように入力し、「Rules」の「Add」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e4409c36-5587-4ee7-b1a4-4df6982d1254.png)

以下のように入力し、「Add」を押下します。
※「onprem.com」の名前解決を先ほど作成したAD(10.1.0.4)に転送します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/12b5b110-ee7b-4608-af51-5bf2c21ef81c.png)

「Review+create」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/88e50224-0f7c-4b30-b7f0-7d3f2a343ad7.png)

「Create」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/c325de5d-7c92-45b4-8c46-afca3c356f2e.png)

以上で、Private DNS Resolverの準備は完了です。
それでは、Azure内のVMから疑似オンプレのDNSの名前解決を試してみましょう。

最初に作成したVMに接続し、コマンドプロンプトで以下のコマンドを実行します。
```
nslookup onpre.com
```

以下のように「10.1.0.4」が返ってくれば転送ルールセットで転送されていることが分かります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/b4722ff5-2b2f-49da-a2c2-fd174dd4f270.png)

経路としては、以下の「3-4」の経路です。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f097110a-ee5f-487b-8b09-0d5ef204547c.png)

以上でAzureからオンプレへの名前解決は実装できましたが、番外編として一個試してみましょう。
転送ルールで全てのドメインをADに転送するようにして、VNetに紐づけられたプライベートDNSゾーンの名前解決をしてみます。
先ほどの図では、転送ルール(3-4)よりプライベートDNSゾーン(3-3)の方が先に参照され、プライベートDNSゾーンで名前解決できるはずです。
では、Azureポータルから「dns-resolver-test」と検索して、先ほど作成したリゾルバの画面に移動します。
そして、「Settings」配下の「Outbound endpoints」へ移動し、「Associated ruleset」の「ruleset」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/fdffc260-c9d5-497b-be43-7f4355ecaa80.png)

「Settings」配下の「Rules」を押下し、「Add」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/0fabf76f-6cec-43ca-b27b-18ad73b4a840.png)

以下のように入力し、「Add」を押下します。
※転送するドメインは「.」を入力することで全てのドメインをADDSに転送します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/5ef24f6d-6f11-419a-b0ac-6ac96a2b1f08.png)

「vm-dns-test」の接続画面に戻り、以下のコマンドを実行しましょう。
```
nslookup google.com
```
転送ルール設定前はインターネットで名前解決できていましたが、ADに転送されるようになったので名前解決できないはずです。※ADはgoogle.comのドメイン情報を持っていません。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/39c98d8c-3e15-49c1-a0d3-c004f9169840.png)

では、以下のコマンドを実行してみましょう。
※<ストレージアカウント名>はこの記事の2番目のハンズオンで作成したストレージアカウント名です。
```
nslookup <ストレージアカウント名.blob.core.windows.net
```
ADへは転送されず、プライベートDNSゾーンを参照して名前解決できるはずです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e171057c-2087-4aca-bdad-71cfa1682ceb.png)

お疲れ様でした。これで「３．ハンズオン ~Azure DNS Private Resolver(アウトバウンドエンドポイント)」は終了です。


## ４．Azure DNS Private Resolver(インバウンドエンドポイント)
Azure DNS Private Resolverのもう一つのコンポーネントは、インバウンドエンドポイントです。
インバウンドエンドポイントを使用することで、オンプレミス等の別ネットワークからAzureプライベートDNSゾーンに対するクエリを受信し名前解決を行うことができます。

以下に、インバウンドエンドポイントを利用する際の名前解決の順番を解説します。
４-１.オンプレミスのクライアントからオンプレミスのDNSサーバーにAzureプライベートDNSゾーンの名前解決を送信します。
４-２.オンプレミスのDNSサーバーは転送ルールの設定に従って、Azureインバウンドエンドポイントに名前解決を転送します。
４-３.インバウンドエンドポイントが存在するVNetの既定のDNSに名前解決が送られます。
４-４.既定のDNSから、VNetにリンクされているプライベートDNSゾーンを参照して名前解決を行います。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/9dc0dc94-0acd-42f0-b6f4-934584d7eb4c.png)

### ４．ハンズオン ~Azure DNS Private Resolver(インバウンドエンドポイント)~
それでは、インバウンドエンドポイントをハンズオンで体験しましょう。
「３．ハンズオン ~Azure DNS Private Resolver(アウトバウンドエンドポイント)」でインバウンドエンドポイントはすでに作成済みなので、インバウンドエンドポイントのIPアドレスだけ確認しておきます。
Azureポータルで「dns-resolver-test」を検索し、リゾルバーの画面に移動します。
「Settings」配下の「Inbound endpoints」を押下し、インバウンドエンドポイントのIPアドレスを確認します。![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/5f5519ad-0a4b-46dd-b4c5-26413acd3955.png)

次に、ADDSをインストールした「vm-adds-dns-test」の接続画面に移動します。
※条件付きフォワーダーを設定していきます。
サーバーマネージャーの「Tools」→「DNS」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/05e17e6e-2f4e-4ca2-9535-09d57a85347e.png)

「DNS Manager」が開きますので、ADDSの名前を開き、「Conditional Forwarders」を右クリック→「New Conditional Forwarder...」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e4adfa2d-6b70-4bba-b110-3f72d936036a.png)

以下のように入力し、「OK」を押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ba8b7c49-6274-453b-9d10-c8e7360e380e.png)

そのままADDS仮想マシン内でコマンドプロンプトを開き、以下のコマンドを実行します。
```
nslookup sgdnstest.blob.core.windows.net
```
AzureのプライベートDNSゾーンで解決し、プライベートIPが返ってくるはずです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f676d772-f42d-4eb1-8297-0dd99a2a0784.png)

以上で、「Azure DNS Private Resolver(インバウンドエンドポイント)」は終了です。
