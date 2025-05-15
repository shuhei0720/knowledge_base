今回は、AzureでExpressRouteを導入する流れを説明します。
AzureのVPNサービスはサイト間VPNとExpressRouteがありますが、今回はExpressRouteについて説明します。
## VNet Gatewayとは
VNet Gatewayは、VNetをVPNやExpressRouteに接続するためのゲートウェイです。
VPNに接続するVNet Gatewayを**VPN Gateway**、ExpressRouteに接続するVNet Gatewayを**ER Gateway**と呼びます。
今回はER Gatewayに焦点を当てていきます。
- VNetには、VPN GatewayとER Gatewayをそれぞれ1つだけ作成できます

## VNet Gatewayの料金
VPN Gatewayは稼働した時間に対して課金が発生します。SKUは以下の2つの要件を踏まえて選びます。
- 可用性ゾーンが必要か
- 必要な帯域幅
[Vnet GatewayのSKU](https://learn.microsoft.com/ja-jp/azure/vpn-gateway/about-gateway-skus#benchmark)

## 導入の流れ
それではExpressRoute導入の流れを説明していきます。
| Step | 作業者(L2接続の場合) | 作業者(L3接続の場合) | 作業概要 |
| :---: | :---: | :---: | :---: |
| 0 | ユーザー/接続プロバイダー | ユーザー/接続プロバイダー | ExpressRoute導入に向けて情報収集する。接続プロバイダーとの調整や契約を行う |
| 1 | ユーザー/接続プロバイダー | ユーザー/接続プロバイダー | ユーザー/接続プロバイダー拠点間を接続する |
| 2 | ユーザー | ユーザー | Azure上でExpressRoute Circuitを作成する |
| 3 | ユーザー | ユーザー | ExpressRoute Circuitのサービスキーを接続プロパイダーへ連絡する |
| 4 | 接続プロバイダー | 接続プロバイダー | ExpressRoute Circuitとの接続に必要な各種設定を行う |
| 5 | ユーザー | 接続プロバイダー | ユーザー自身で用意したオンプレミス設置BGPルーターの設定を行う |
| 6 | ユーザー | 接続プロバイダー | Private Peeringの構成を行う |
| 7 | ユーザー | ユーザー | ExpressRoute CircuitとER Gatewayを接続する |
| 8 | ユーザー | ユーザー | ExpressRoute接続を確認する |

ExpressRouteは単独では利用できず、接続プロバイダーのサービスと組み合わせて利用することが前提になります。
接続プロバイダーは、L3接続プロパイダーとL2接続プロパイダーに大別されます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/fc54e4f9-726b-4726-8f76-bda99b1728af.png)

- **L3接続の場合**：ルーティングやルーターの管理等も含めて接続プロパイダーへ任せることができ、導入・運用負荷が少なくなる
- **L2接続の場合**：ルーティングやルーターの管理などを自分たちで行う必要があるが、より柔軟なネットワーク構成が可能になる

今回は、L2接続における導入の流れを紹介します。
L3接続の場合は、Step4～Step6の作業を接続プロパイダーに任せられるため不要です。

## Step0. ExpressRoute導入に向けて情報収集する
この手順は重要であり、このステップ次第でStep1以降の手順が変わります。
以下に重要なポイントを示します。
- 契約する接続プロパイダーの検討
    - それぞれの接続プロパイダーによって提供サービス内容や納期が異なり、また複数の接続プロパイダーと契約することも可能
    - ExpressRoute導入検討時のできるだけ早いタイミングで、接続プロパイダーに相談する(ポートの空き状況により、利用開始に半年程度かかる場合もある)
- ExpressRouteを介して利用したいAzureサービスの整理
    - Azure(IaaS,PaaS)なのかM365(SaaS)なのかでピアリングの種類が変わる
- ピアリングの種類
    - ExpressRouteのピアリングの種類には、**Private Peering**と**Microsoft Peering**の2種類がある
    - Azure(IaaS,PaaS)と接続する場合Private Peering、M365(SaaS)と接続する場合Microsoft Peeringを選択する
- ExpressRouteの本数、場所(ロケーション)やSKUやオプションの選択
    - 場所は東日本で3つ(Tokyo/Tokyo2/Tokyo3)、西日本では1つ(Osaka)提供されており、接続プロパイダーによって利用可能な場所が異なる

| Azureリージョン | 場所(Location) | 接続ポイント | 接続プロバイダー |
| :---: | :---: | :---: | :---: |
| 東日本 | Tokyo | Equinix TY4 | Aryaka Networks, AT&T NetBond, BBIX, British Telecom, CenturyLink Cloud Connect, Colt, Equinix, Intercloud,Internet Initiative Japan Inc., Megaport, NTT Communications, NTT EAST, オレンジ, Softbank, Telehouse(KDDI), Verizon |
| 東日本 | Tokyo2 | アット東京 | アット東京, China Unicom Global, Colt, Equinix, IX Reach, Megaport, PCCW Global Limited, Tokai Communications |
| 東日本 | Tokyo3 | NEC | NEC, SCSK |
| 西日本 | Osaka | Equinix OS1 | アット東京, BBIX, Colt, Equinix, Internet Initiative Japan Inc., Megaport, NTT Communications, NTT SmartConnect, Softband, Tokai Communications |
- オンプレミス側のネットワーク構成の確認
    - オンプレミス側のネットワーク構成を変更する必要があるか確認
- Azure側のネットワーク構成の検討
    - 推奨はHub&Spoke構成
    - 大規模で複雑な接続シナリオを実現したい場合、Virtual WANも検討
    - オンプレミスと重複したIPアドレスは利用できない

## Step1. ユーザー/接続プロパイダー拠点間を接続する
通常は、既存の接続プロパイダーWAN回線網にExpressRouteを接続することが多いため、この手順はスキップすることもあります。
しかし、新規契約の接続プロバイダーの場合、オンプレミス環境への物理配線の引き込みなどの作業が発生します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/5ca32140-18ee-4c2e-afab-b790b05f7a8a.png)

## Step2. ExpressRoute Circuitの作成
続いて、ExpressRoute Circuitを作成します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/fde02d25-e18c-46d5-a59a-e597f51da37b.png)
作成する際は、「接続に使う場所」と「接続プロバイダー」を選択します。ここでの場所とは、**ExpressRoute回線の物理ルーターが敷設されている施設のことで、TokyoやTokyo2などの値を選択します。

## Step3. ExpressRoute Circuitのサービスキーを接続プロバイダーへ連絡する
作成されたExpressRoute Circuitのサービスキーを接続プロバイダーへ連絡します。
連絡方法は、ポータル画面から入力する形式や所定の申請書に記載して送付する形式など、接続プロバイダーによって異なるため、接続プロバイダーに確認してください。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f5e3e0ac-955d-4217-b1a5-b81e4bdfb1b1.png)

## Step4. ExpressRoute Circuitとの接続に必要な各種設定作業を行う　※L3接続の場合不要
接続プロバイダー側で、ExpressRoute Circuitとの接続に必要な各種設定作業が行われます。
なお、ここでは「接続プロバイダーの作業範囲としましたが、接続プロバイダーのポータル画面などから、ユーザー自身が各種設定パラメータなどの入力を求められる場合もあります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/1c732dee-7eeb-40db-94b5-02535bea30dd.png)
この接続プロバイダーの作業が完了すれば、Azure Portalから確認できるExpressRoute Circuitが、[プロバイダーの状態：プロビジョニング済み]となります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/2c716ed4-1c01-4603-b87c-45bbccf5f49e.png)

## Step5. オンプレミス設置BGPルーターの設定を行う　※L3接続の場合不要
ユーザー側で、オンプレミス設置BGPルーターの設定を行います。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/0d811960-1234-45e1-9d46-8e578587a2bf.png)

接続プロバイダーとの接続に関する必要な設定など、接続プロバイダーから通知された内容に従い、ユーザー側でオンプレミス設置BGPルーターの設定を行います。
- L2接続プロバイダーの場合、接続プロバイダーからピアリングのための**VLAN ID**が指定される
- それぞれのVLANリンクに[/30]のプライベートサブネットを割り当てる必要がある。なお、Vnetでは利用していないサブネットを使う。図では、172.16.0.0/30と172.16.0.4/30を割り当てており、オンプレミス設置スイッチが若番のIPアドレスである、172.16.0.1と172.16.0.5になる
- AS番号については、Azure側は[12076]で固定。オンプレミス側のAS番号(65515～65520までの番号を除く)を用意し、オンプレミス設置BGPルーターに設定する

## Step6. Private Peeringの構成を行う　※L3接続の場合不要
Azure PortalからPrivate Peeringの構成を行います。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/75a7aa2e-8df0-43e5-9989-ef7f0ba44d41.png)
Azure Portalから確認できるExpressRoute Circuitが、[プロバイダーの状態：プロビジョニング済み]となっていることを確認します。
[ピアリング]→[Azureプライベート]を選択して設定を行います。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/b4bd4b57-59b3-486f-90a1-87304c3496e2.png)
- **ピアASN**：オンプレミス側で設定したAS番号を指定する
- **プライマリサブネット、セカンダリサブネット**：2つのプライベートサブネットを指定する
- **IPv4ピアリング**：有効にする
- **VLAN ID**：プロバイダーから通知されたVLAN IDを指定する

設定を保存すると、Private Peeringが構成され、[状態]が有効となります。
[Azureプライベート]を選択すれば、ルートやARPが確認できるので、正常に構成できているか確認します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/fa921f73-2c32-435c-91c2-520147e06360.png)
このステップで、オンプレミス側の機器も正しく設定できているか確認します。
Ciscoであれば```show ip bgp neighbors xx.xx.xx.xx```や```show bgp summary```などでBGP接続のステータスを確認し、MSEEとBGP接続が確立できているか確認します。

## Step7. ExpressRoute CircuitとER Gatewayを接続する
ExpressRoute CircuitとER Gatewayを接続します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/95253aeb-cb4c-4774-a53b-893fb6ceb244.png)
なお、ExpressRoute CircuitとER Gatewayは以下の条件で、互いに複数で接続可能です。
- ExpressRoute Circuit(Local/Standard SKU)は、最大10個のVnet(ER Gateway)と接続可能。さらに、Premium SKUを利用すれば、回線のサイズに応じて最大100個まで拡張することができる
- ER Gatewayに接続可能なExpressRoute Circuitの数は、接続しようとしているExpressRoute Circuitのロケーションのよって異なる
- 異なるLocationの場合：Gateway SKU次第
    - Standard/ERGw1Az：4つまで
    - High Perf/ERGw2Az：8つまで
    - Ultra Perf/ERGw3Az：16つまで

ER Gatewayは、「GatewaySubnet」という既定の名前の専用サブネットが必要になります。
このサブネットをVNet内に事前作成したうえで、下図のように必要なSKUなどを指定してExpressRoute用のER Gatewayをデプロイします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/b4e228fa-d9fe-4c68-aa3f-275385b9d23b.png)

ER Gatewayのデプロイが完了したら、ExpressRoute Circuitを選択し、[接続]を作成します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/db566f71-79ae-4da6-bd51-5393938d7142.png)
プルダウンから、接続するER Gatewayを選択します。
複数接続を作成する場合は、重みづけの設定もできます。
規定はECMP(Equal-cost multi-path routing)です。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/35e0e7d7-7413-4dd3-9dc7-b5e0da957018.png)

## Step8. ExpressRoute接続を確認する
VNet内にデプロイされている仮想マシンであれば、オンプレミスのクライアントからプライベートIPアドレスでのリモートデスクトップ接続や、Ping疎通をテストしてみましょう。


## Tips. ExpressRouteの可用性について
ExpressRouteは、既定で冗長化されたSLAが提供されています。
VPN Gatewayと異なり、ER Gatewayはアクティブ/アクティブで構成され、MSEEとPE間も標準で2つの回線で構成されています。
つまり、ExpressRouteを1本導入することで、内部的には2回線が冗長化された状態で利用可能となっています。
さらに大規模災害を想定して可用性を高めたい場合は、以下のような構成を検討します。
- ExpressRoute 1本＋VPN
    -ExpressRouteの障害時はVPNにフォールバックする
- ExpressRoute 2本
    - 例えば、東京の被災時は大阪のExpressRouteにフォールバックする
    - セカンダリの場所(大阪)にも接続プロバイダーとの契約ｙオンプレミスの接続点の用意が必要
