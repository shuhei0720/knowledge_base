今回は、AzureでVPNを導入する流れを説明します。AzureのVPNサービスはサイト間VPNとExpressRouteがありますが、今回はサイト間VPNについて説明します。
## VNet Gatewayとは
VNet Gatewayは、VNetをVPNやExpressRouteに接続するためのゲートウェイです。
VPNに接続するVNet Gatewayを**VPN Gateway**、ExpressRouteに接続するVNet Gatewayを**ER Gateway**と呼びます。今回はVPN Gatewayに焦点を当てていきます。
- VNetには、VPN GatewayとER Gatewayをそれぞれ1つだけ作成できます

## VNet Gatewayの料金
VPN Gatewayは稼働した時間に対して課金が発生します。SKUは以下の2つの要件を踏まえて選びます。
- 可用性ゾーンが必要か
- 必要な帯域幅
[Vnet GatewayのSKU](https://learn.microsoft.com/ja-jp/azure/vpn-gateway/about-gateway-skus#benchmark)

## 導入の流れ
それではVPNを使用したサイト間接続導入の流れを説明していきます。
| Step | 作業者 | 作業場所 | 作業概要 |
| :---: | :---: | :---: | :---: |
| 0 | ユーザー | - | VPN導入に向けて情報収集する |
| 1 | ユーザー | Azure | VPN接続に必要となるリソースを構築する |
| 2 | ユーザー | オンプレミス | VPNデバイスを設定する |
| 3 | ユーザー | オンプレミス | VPN接続を確認する |

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/5004916f-0586-4000-a5e3-007afe7ac0d3.png)

## Step0. VPN導入に向けて情報収集する
この手順は重要であり、このステップ次第でStep1以降の手順が変わります。
以下に重要なポイントを示します。
- VPN ゲートウェイのSKU
    - 必要となるサイト間接続の数、スループットを踏まえて検討する
    - 基本的に**Generation2**のSKUを選択する
    - 同じ世代内であればSKUのサイズ変更が可能
    - Generation2はBGPをサポートしており、ExpressRouteとの共存が可能
- オンプレミス側のネットワーク構成の確認
    - Azureと接続するVPNデバイスが検証済みのVPNデバイスであれば、Azureとの接続がサポートされている。[検証済みのVPNデバイスとデバイス構成ガイド](https://learn.microsoft.com/ja-jp/azure/vpn-gateway/vpn-gateway-abount-vpn-devices#devicetable)
    - 外部接続用の固定パブリックIPアドレスがあるか
- Azure側のネットワーク構成の検討
    - Hub&Spoke構成が推奨
    - 大規模で複雑な接続シナリオの場合、Virtual WANも検討する
    - オンプレミスと重複したIPアドレスは利用できない

## Step1. VPN接続に必要となるAzureリソースを構築する
まず、以下のリソースを作成します。
1. VNet及びゲートウェイサブネット
1. VPN Gateway(パブリックIPアドレス含む)
1. ローカルネットワークゲートウェイ
1. 接続

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/a6c6652a-f2c0-4435-bc3e-2766df48d1c8.png)

### 1.VNet及びゲートウェイサブネット
1つのVNet内にデプロイできるVPN Gatewayは1つなので、複数のゲートウェイが必要な場合は注意してください。
また、ゲートウェイサブネットに関しては、以下のように制約事項があります。
- VPN Gatewayのみが存在するサブネットであり、VMなどほかのリソースは作成してはいけない
- NSGを適用してはいけない
- サブネットのプライベートIP範囲は/27以上が推奨されている
    - ExpressRouteと共存させる場合や、大規模な構成である場合、/27以上が必要
    - サブネット内にリソースが無ければIP範囲は変更可能
    - 割り当てたIPアドレスは内部的にゲートウェイVMとゲートウェイサービスに利用される
    - サブネットの名前は「GatewaySubnet」とする

VNet作成時にサブネットも同時に作成できますが、以下のように「サブネットテンプレート」から[Virtual Network Gateway]を選択すると上記の制約事項を満たしたサブネットが設定できます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/390391cb-4138-4865-a4ca-f7de496f6f12.png)

### 2.VPN Gateway
サブネット作成後、以下のようにVPN用の仮想ネットワークゲートウェイを、必要項目を入力してデプロイいします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/86bfb78c-ca29-4c28-af5c-63340147cb1c.png)
- **ゲートウェイの種類**：VPNを選択
- **アクティブ/アクティブモードの有効化**：高可用性を必要とする場合に有効化する。このパラメータはデプロイ後でも変更できるため、特に高可用性構成に関心が無ければ、いったん[無効]で作成して問題ない
- **BGPの構成**：BGPによるダイナミックルーティングを利用する場合に有効化する。アクティブ/アクティブモードを有効化する場合は、フェールオーバー時の制御を動的に行うためにBGPもあわせて有効化するのが一般的

### 3.ローカルネットワークゲートウェイ
VPN Gatewayの作成後、ローカルネットワークゲートウェイを作成します。ローカルネットワークゲートウェイは、オンプレミスのVPNデバイスをAzure上で認識させるために必要なリソースです。そのため、VPN デバイスに設定されているパブリックIPアドレスとオンプレミスのアドレス空間をローカルネットワークゲートウェイとして指定します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/1a65f5d2-9aab-48ca-afde-d5be01ad0a28.png)
**IPアドレス**：VPN デバイスのパブリックIPアドレス
**アドレス空間**：VPN デバイスからAzureのVPN Gatewayに広報するIPアドレス範囲(BGPでピアリングする場合は不要)

### 接続
接続は、作成したVPN Gatewayとローカルネットワークゲートウェイを紐づけるために必要な、独立した1つのAzureリソースです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/3d267ab8-7939-434b-8213-09094779aa7f.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/fe1497a2-8c18-428f-878d-1969228e737a.png)

## Step2. VPNデバイスを設定する
続いて、オンプレミスのVPN デバイスの設定を行います。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/1486bbb6-9eef-43fb-90c4-1e10595f2f6e.png)
VPN デバイスの設定は、「接続」画面から[構成のダウンロード]を利用すると、各デバイスのファームウェアなどに合わせた設定のサンプルがダウンロードできます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/730b30e6-2b0a-4163-a658-e78c7df800b5.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/2049004a-06bf-49e7-8d09-ac9bccc413e5.png)


ダウンロードしたサンプルは、「接続の作成」画面で入力した共有キーやVPN GatewayのパブリックIPなど、VPN接続に必要となるAzureリソースの情報が自動的に入力されています。その他、部分的に編集が必要な項目(アクセスリストなど)があり、それらはデバイスベンダーによって異なるため、詳細な設定手順は各デバイスベンダーに問い合わせてください。以下はCiscoの場合のサンプルです。
```

! Microsoft Corporation
! ------------------------------------------------------------------------------
! Sample VPN tunnel configuration template for Cisco IOS-based devices
!
! ##############################################################################
! !!! Search for "REPLACE" to find the values that require special
! !!! considerations
! !!!
! !!! (1) ACL/access-list rule numbers
! !!! (2) Tunnel interface number
! !!! (3) Tunnel interface IP address
! !!! (4) BGP routes to advertise (if BGP is enabled)
! !!! (5) BGP peer IP address on the device - loopback interface number
! ##############################################################################
!
! [0] Device infomration
!
!   > Device vendor:    Cisco
!   > Device family:    IOS-based (ASR, ISR)
!   > Firmware version: IOS 15.1 or beyond
!   > Test platform:    Cisco ISR 2911, version 15.2
!
! [1] Network parameters
!
!   > Connection name:       Connection01
!   > VPN Gateway name:      7627006d-41d9-416b-9fb7-3af00177ace0
!   > Public IP addresses:   
!     + Public IP 1:         74.176.139.86
!   > Virtual network address space: 
!     + CIDR:10.0.0.0/16, prefix:10.0.0.0, netmask:255.255.0.0, wildcard:0.0.255.255
!   > Local network gateway: LocalNetworkGateway01
!   > On-premises VPN IP:    48.218.158.144
!   > On-premises address prefixes:
!     + CIDR:10.1.0.0/16, prefix:10.1.0.0, netmask:255.255.0.0, wildcard:0.0.255.255
!
! [2] IPsec/IKE parameters
!
!   > IKE version:             IKEv2
!     + Encryption algorithm:  aes-cbc-256
!     + Integrityalgorithm:    sha1
!     + Diffie-Hellman group:  2
!     + SA lifetime (seconds): 3600
!     + Pre-shared key:        SSss07200270
!     + UsePolicyBasedTS:      False
!
!   > IPsec
!     + Encryption algorithm:  esp-gcm 256
!     + Integrity algorithm:   
!     + PFS Group:             none
!     + SA lifetime (seconds): 3600
!     + SA lifetime (KB):      102400000
!
! [3] BGP parameters - Azure VPN gateway
!
!   > Azure virtual network
!     + Enable BGP:            False
!     + Azure BGP ASN:         VNG_ASN
!   > On-premises network / LNG
!     + On premises BGP ASN:   LNG_ASN
!     + On premises BGP IP:    LNG_BGPIP
!
! ==============================================================================
! Cisco IOS 15.x+ IKEv2, route-based (any-to-any) 
! ==============================================================================
!
! ACL rules
! 
! Some VPN devices require explicit ACL rules to allow cross-premises traffic:
!
! 1. Allow traffic between on premises address ranges and VNet address ranges
! 2. Allow IKE traffic (UDP:500) between on premises VPN devices and Azure VPN gateway
! 3. Allow IPsec traffic (Proto:ESP) between on premises VPN devices and Azure VPN gateway
! [REPLACE] access-list number: access-list 101

access-list 101 permit ip 10.1.0.0 0.0.255.255 10.0.0.0 0.0.255.255
access-list 101 permit esp host 74.176.139.86 host 48.218.158.144
access-list 101 permit udp host 74.176.139.86 eq isakmp host 48.218.158.144
access-list 101 permit udp host 74.176.139.86 eq non500-isakmp host 48.218.158.144

! ==============================================================================
! Internet Key Exchange (IKE) configuration
! - IKE Phase 1 / Main mode configuration
! - Encryption/integrity algorithms, Diffie-Hellman group, pre-shared key

crypto ikev2 proposal Connection01-proposal
  encryption aes-cbc-256
  integrity  sha1
  group      2
  exit

crypto ikev2 policy Connection01-policy
  proposal Connection01-proposal
  match address local 48.218.158.144
  exit
  
crypto ikev2 keyring Connection01-keyring
  peer 74.176.139.86
    address 74.176.139.86
    pre-shared-key SSss07200270
    exit
  exit

crypto ikev2 profile Connection01-profile
  match address  local 48.218.158.144
  match identity remote address 74.176.139.86 255.255.255.255
  authentication remote pre-share
  authentication local  pre-share
  lifetime       3600
  dpd 10 5 on-demand
  keyring local  Connection01-keyring
  exit

! ------------------------------------------------------------------------------
! IPsec configuration
! - IPsec (or IKE Phase 2 / Quick Mode) configuration
! - Transform Set: IPsec encryption/integrity algorithms, IPsec ESP mode

crypto ipsec transform-set Connection01-TransformSet esp-gcm 256 
  mode tunnel
  exit

crypto ipsec profile Connection01-IPsecProfile
  set transform-set  Connection01-TransformSet
  set ikev2-profile  Connection01-profile
  set security-association lifetime seconds 3600
  exit

! ------------------------------------------------------------------------------
! Tunnel interface (VTI) configuration
! - Create/configure a tunnel interface
! - Configure an APIPA (169.254.x.x) address that does NOT overlap with any
!   other address on this device. This is not visible from the Azure gateway.
! * REPLACE: Tunnel interface numbers and APIPA IP addresses below
! * Default tunnel interface 11 (169.254.0.1) and 12 (169.254.0.2)

int tunnel 11
  ip address 169.254.0.1 255.255.255.255
  tunnel mode ipsec ipv4
  ip tcp adjust-mss 1350
  tunnel source 48.218.158.144
  tunnel destination 74.176.139.86
  tunnel protection ipsec profile Connection01-IPsecProfile
  exit


! ------------------------------------------------------------------------------
! Static routes
! - Adding the static routes to point the VNet prefixes to the IPsec tunnels
! * REPLACE: Tunnel interface number(s), default tunnel 11 and tunnel 12

ip route 10.0.0.0 255.255.0.0 Tunnel 11

! ==============================================================================
! Cleanup script
! ==============================================================================
!
! [WARNING] This section of the script will cleanup the resources: IPsec/IKE,
! [WARNING] interfaces, routes, access-list. Validate the objects in your
! [WARNING] configuration before applying the script below.
! [REPLACE] Interfaces: Loopback 11, Tunnel 11, Tunnel 12; access-list 101
!
!!
!! no ip route 10.0.0.0 255.255.0.0 Tunnel 11
!!
!!
!! no int tunnel 11
!!
!! no crypto ipsec profile Connection01-IPsecProfile
!! no crypto ipsec transform-set Connection01-TransformSet
!!
!! no crypto ikev2 profile Connection01-profile
!! no crypto ikev2 keyring Connection01-keyring
!! no crypto ikev2 policy Connection01-policy
!! no crypto ikev2 proposal Connection01-proposal
!!
!! no access-list 101 permit ip 10.1.0.0 0.0.255.255 10.0.0.0 0.0.255.255
!! no access-list 101 permit esp host 74.176.139.86 host 48.218.158.144
!! no access-list 101 permit udp host 74.176.139.86 eq isakmp host 48.218.158.144
!! no access-list 101 permit udp host 74.176.139.86 eq non500-isakmp host 48.218.158.144
```
無事にVPNデバイスの設定が完了すると、「接続」画面で[接続済み]のステータスとなり、VPN接続が確立されていることを確認できます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/c05c20e5-ff96-41b1-9936-25ab6b6d1113.png)


## Step3. VPN接続を確認する
VNet内にデプロイされている仮想マシンであれば、オンプレミスのクライアントからプライベートIPアドレスでのリモートデスクトップ接続や、Ping疎通をテストしてみましょう。

## Tips. VPN Gatewayの可用性について
VPN Gatewayを高可用性にしたい場合は以下の組み合わせにすると良いです。
- ゾーン冗長ゲートウェイを使用する(SKUを「～AZ」にする)
[ゾーン冗長ゲートウェイについて](https://learn.microsoft.com/ja-jp/azure/vpn-gateway/about-zone-redundant-vnet-gateways)
- アクティブ/アクティブモードにする(※オンプレミス側のVPN デバイスも2台にする)
[アクティブ/アクティブモードゲートウェイについて](https://learn.microsoft.com/ja-jp/azure/vpn-gateway/about-active-active-gateways)
