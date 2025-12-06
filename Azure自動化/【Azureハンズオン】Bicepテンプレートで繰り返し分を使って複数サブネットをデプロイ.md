# はじめに
　今回はBicepテンプレートを使ってAzureに**仮想ネットワークと複数サブネットをデプロイする際の問題と解決方法**をご紹介します。
読者の方が、ご自分のアカウントでも体験できるように**ハンズオンとして構成**しています。


# 対象読者

- Azureを使用する現場に従事している方
- Azureの資格勉強をしている方
- Azureに興味がある方
- クラウドに興味がある方
- IaCに興味がある方

# 免責事項
- ハンズオンにあたってリソースを放置すると料金がかかる可能性があります。**筆者が記事作成の間リソースを使用した分では0円**でした。記事の最後に**リソースの削除の仕方**も解説しておりますので、ハンズオンが終了したら**削除を実施**してもらうようお願いします。
- Azureの画面に関しては、**2024年10月02日時点のもの**を使用しております。
- ハンズオンにはVSCodeとBicep拡張機能を使用します。👉[拡張機能の使い方](https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/bicep/visual-studio-code?tabs=CLI)
- ハンズオンではコードが長くなるのを避けるために一部のルートテーブルやNSGの記述を省いて作成しています。


# 使用サービス・技術

### Bicepテンプレートとは
Bicepテンプレートとは、Azureリソースのデプロイを行うためのインフラストラクチャ・アズ・コード（IaC）のツールです。Bicepを使用することで、ARMテンプレート(JSONテンプレート)の記述を30%から40%削減できると言われています。
👉[公式ドキュメント](https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/bicep/overview?tabs=bicep)

### Bicepテンプレートの特徴
- **簡潔な構文**: 直感的で簡単に記述できる構文を提供し、より少ないコードでテンプレートを記述できます。個人的な感覚的にはJavascriptとかに似ています。
- **型チェック**: 強力なコード内のエラー検出機能がある。VSCodeの拡張機能を使えば編集している時点でほぼ全ての記述間違いを指摘してくれます。
👉[拡張機能の使い方](https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/bicep/visual-studio-code?tabs=CLI)
- **モジュール化**: 関数やモジュールを使用して、コードの再利用性と整理を向上させます。今回の現場でも何度も行うデプロイを自動化するために作成しました。

### ARMテンプレート(JSON)との比較
- **可読性が高い**:ARMテンプレートと比較すると明らかに読みやすいです。人間が理解しやすい構文を意識して作られています。
- **開発速度**: 簡潔で、ツールサポートが充実しているため、迅速に開発が可能。ARMテンプレートからの可能
👉[ARMテンプレートからBicepへの変換](https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/bicep/decompile?tabs=azure-cli)
- **デバッグとメンテナンス**:シンプルな構文でエラーを見つけやすい。コードの構造の理解がしやすいので、デバッグが早いです。


# 1. 要件

### 要件①：繰り返し手動で行う仮想ネットワーク・サブネット・ルートテーブル・NSGのデプロイ作業を自動化したい。
### 要件②：複数サブネットをBicepでデプロイすると、依存関係のエラーでデプロイできない。
#### エラー内容
```
AnotherOperationInProgress: Another operation on this or dependent resource is in progress. To retrieve status of the operation use uri: https://management.azure.com/subscriptions/~~~~
```

# 2. 解決策

### Bicepテンプレートを使用してデプロイを自動化
Bicepテンプレートを使用することで、デプロイ作業の負荷を軽減します。

### 繰り返し分を使って複数サブネットを作成
繰り返し分を使って複数サブネットを作成すれば、要件②のエラーを解決できました。

# 3. ハンズオン

それでは実際に作成していきます。

### 3-1.Bicepテンプレートファイルの作成
早速ですが、Bicepのテンプレートを作成していきます。
PC上の任意の場所にBicepハンズオンというフォルダを作成します。![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/36643024-e3e1-5605-a25d-62de9f09bc69.png)
<br>
VSCodeを起動し、先ほど作成したフォルダを開きます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ed5b7a0f-ab2d-fae6-a25f-4b374b978c79.png)
<br>
ファイルマークを押して、main.bicepというファイルを作成します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/bed6656c-b2f0-2a9a-2ae5-e6abb2573080.png)
<br>
main.bicepに以下のコードを記述します。
paramデコレータを使用して、コード内で使用するパラメーターを定義します。
このコードのポイントはサブネットの名前とアドレス範囲を配列で定義しているところです。
```
// 変数の定義
@description('仮想ネットワークの名前')
param vnetName string = 'testVnet'

@description('仮想ネットワークのアドレスプレフィックス')
param vnetAddressPrefix string = '10.0.0.0/16'

@description('仮想ネットワークのサブネット')
param Subnets array = [
  {
    // GatewaySubnetの名前とアドレス範囲
    name: 'GatewaySubnet'
    subnetPrefix: '10.0.1.0/24'
  }
  {
    // FWSubnetの名前とアドレス範囲
    name: 'AzureFirewallSubnet'
    subnetPrefix: '10.0.2.0/24'
  }
  {
    // PrivateSubnetの名前とアドレス範囲
    name: 'PrivateSubnet'
    subnetPrefix: '10.0.3.0/24'
  }
  {
    // ProtectedSubnetの名前とアドレス範囲
    name: 'ProtectedSubnet'
    subnetPrefix: '10.0.4.0/24'
  }
]

@description('FWSubnetのAzure FirewallのIPアドレス')
param SpokeFWIP string = '10.0.2.4'
```
<br>

次に、その下に下記のコードを記述します。
デプロイするリージョンを変数に代入します。
```
// リージョンをJapanEastに指定
var location = 'JapanEast'
```
<br>

次に、その下に下記のコードを記述します。
仮想ネットワークを作成し、さきほど定義したサブネットの配列分のサブネットを繰り返し(for文)で作成しています。
```
// 仮想ネットワークとサブネットの作成
resource virtualNetwork 'Microsoft.Network/virtualNetworks@2023-11-01' = {
  name: vnetName
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: [ vnetAddressPrefix ]
    }
    // 繰り返しでサブネットを作成
    subnets: [for subnet in Subnets: {
      name: subnet.name
      properties: {
        addressPrefix: subnet.subnetPrefix
      }
    }]
  }
}
```
<br>

次に、ルートテーブルを定義します。
下記のコードを下に記述してください。
今回はGWサブネット以外のルートテーブルは省略します。
```
// GWサブネットのルートテーブルの作成
resource routeTable1 'Microsoft.Network/routeTables@2023-11-01' = {
  name: 'GW-rt'
  location: location
  properties: {
    routes: [
      {
        name: 'ToFW'
        properties: {
          addressPrefix: Subnets[2].subnetPrefix // プライベートサブネット
          nextHopType: 'VirtualAppliance'
          nextHopIpAddress: SpokeFWIP
        }
      }
    ]
  }
  dependsOn: [
    virtualNetwork
  ]
}
```
<br>

次に、NSGを定義します。
下記のコードを下に記述してください。
今回はプロテクテッドサブネット以外のNSGは省略します。
```
// ProtectedサブネットのNSGの作成とセキュリティ規則の設定
resource protectedNsg 'Microsoft.Network/networkSecurityGroups@2023-11-01' = {
  name: 'protected-nsg'
  location: location
  properties: {
    securityRules: [
      {
        name: 'DenyAllInbound'
        properties: {
          priority: 1000
          direction: 'Inbound'
          access: 'Deny'
          protocol: '*'
          sourcePortRange: '*'
          destinationPortRange: '*'
          sourceAddressPrefix: '0.0.0.0/0'
          destinationAddressPrefix: '*'
        }
      }
      {
        name: 'DenyAlloutbound'
        properties: {
          priority: 1010
          direction: 'Outbound'
          access: 'Deny'
          protocol: '*'
          sourcePortRange: '*'
          destinationPortRange: '*'
          sourceAddressPrefix: '*'
          destinationAddressPrefix: '0.0.0.0/0'
        }
      }
      {
        name: 'AllowFromPrivateSubnet'
        properties: {
          priority: 900
          direction: 'Inbound'
          access: 'Allow'
          protocol: '*'
          sourcePortRange: '*'
          destinationPortRange: '*'
          sourceAddressPrefix: Subnets[2].subnetPrefix
          destinationAddressPrefix: '*'
        }
      }
      {
        name: 'AllowToPrivateSubnet'
        properties: {
          priority: 990
          direction: 'Outbound'
          access: 'Allow'
          protocol: '*'
          sourcePortRange: '*'
          destinationPortRange: '*'
          sourceAddressPrefix: '*'
          destinationAddressPrefix: Subnets[2].subnetPrefix
        }
      }
    ]
  }
}
```
<br>

最後に、ルートテーブルとNSGをそれぞれサブネットに紐づけます。
下記のコードを追記してください。
```
// GWサブネットへのルートテーブル紐づけ
resource GatewaySubnet 'Microsoft.Network/virtualNetworks/subnets@2023-11-01' = {
  parent: virtualNetwork
  name: 'GatewaySubnet'
  properties: {
    addressPrefix: Subnets[0].subnetPrefix
    routeTable: {
      id: routeTable1.id
    }
  }
  dependsOn: [
    virtualNetwork
    routeTable1
  ]
}

// ProtectedサブネットへのNSG紐づけ
resource protectedSubnet 'Microsoft.Network/virtualNetworks/subnets@2023-11-01' = {
  parent: virtualNetwork
  name: 'ProtectedSubnet'
  properties: {
    addressPrefix: Subnets[3].subnetPrefix
    networkSecurityGroup: {
      id: protectedNsg.id
    }
  }
  dependsOn: [
    virtualNetwork
    protectedNsg
  ]
}
```

以上で、Bicepテンプレートの作成は完了です！
以下が完成したコード全体になります。

### main.bicep
```
// 変数の定義
@description('仮想ネットワークの名前')
param vnetName string = 'testVnet'

@description('仮想ネットワークのアドレスプレフィックス')
param vnetAddressPrefix string = '10.0.0.0/16'

@description('仮想ネットワークのサブネット')
param Subnets array = [
  {
    // GatewaySubnetの名前とアドレス範囲
    name: 'GatewaySubnet'
    subnetPrefix: '10.0.1.0/24'
  }
  {
    // FWSubnetの名前とアドレス範囲
    name: 'AzureFirewallSubnet'
    subnetPrefix: '10.0.2.0/24'
  }
  {
    // PrivateSubnetの名前とアドレス範囲
    name: 'PrivateSubnet'
    subnetPrefix: '10.0.3.0/24'
  }
  {
    // ProtectedSubnetの名前とアドレス範囲
    name: 'ProtectedSubnet'
    subnetPrefix: '10.0.4.0/24'
  }
]

@description('FWSubnetのAzure FirewallのIPアドレス')
param SpokeFWIP string = '10.0.2.4'

// リージョンをJapanEastに指定
var location = 'JapanEast'

// 仮想ネットワークとサブネットの作成
resource virtualNetwork 'Microsoft.Network/virtualNetworks@2023-11-01' = {
  name: vnetName
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: [ vnetAddressPrefix ]
    }
    // 繰り返しでサブネットを作成
    subnets: [for subnet in Subnets: {
      name: subnet.name
      properties: {
        addressPrefix: subnet.subnetPrefix
      }
    }]
  }
}

// GWサブネットのルートテーブルの作成
resource routeTable1 'Microsoft.Network/routeTables@2023-11-01' = {
  name: 'GW-rt'
  location: location
  properties: {
    routes: [
      {
        name: 'ToFW'
        properties: {
          addressPrefix: Subnets[2].subnetPrefix // プライベートサブネット
          nextHopType: 'VirtualAppliance'
          nextHopIpAddress: SpokeFWIP
        }
      }
    ]
  }
  dependsOn: [
    virtualNetwork
  ]
}

// ProtectedサブネットのNSGの作成とセキュリティ規則の設定
resource protectedNsg 'Microsoft.Network/networkSecurityGroups@2023-11-01' = {
  name: 'protected-nsg'
  location: location
  properties: {
    securityRules: [
      {
        name: 'DenyAllInbound'
        properties: {
          priority: 1000
          direction: 'Inbound'
          access: 'Deny'
          protocol: '*'
          sourcePortRange: '*'
          destinationPortRange: '*'
          sourceAddressPrefix: '0.0.0.0/0'
          destinationAddressPrefix: '*'
        }
      }
      {
        name: 'DenyAlloutbound'
        properties: {
          priority: 1010
          direction: 'Outbound'
          access: 'Deny'
          protocol: '*'
          sourcePortRange: '*'
          destinationPortRange: '*'
          sourceAddressPrefix: '*'
          destinationAddressPrefix: '0.0.0.0/0'
        }
      }
      {
        name: 'AllowFromPrivateSubnet'
        properties: {
          priority: 900
          direction: 'Inbound'
          access: 'Allow'
          protocol: '*'
          sourcePortRange: '*'
          destinationPortRange: '*'
          sourceAddressPrefix: Subnets[2].subnetPrefix
          destinationAddressPrefix: '*'
        }
      }
      {
        name: 'AllowToPrivateSubnet'
        properties: {
          priority: 990
          direction: 'Outbound'
          access: 'Allow'
          protocol: '*'
          sourcePortRange: '*'
          destinationPortRange: '*'
          sourceAddressPrefix: '*'
          destinationAddressPrefix: Subnets[2].subnetPrefix
        }
      }
    ]
  }
}

// GWサブネットへのルートテーブル紐づけ
resource GatewaySubnet 'Microsoft.Network/virtualNetworks/subnets@2023-11-01' = {
  parent: virtualNetwork
  name: 'GatewaySubnet'
  properties: {
    addressPrefix: Subnets[0].subnetPrefix
    routeTable: {
      id: routeTable1.id
    }
  }
  dependsOn: [
    virtualNetwork
    routeTable1
  ]
}

// ProtectedサブネットへのNSG紐づけ
resource protectedSubnet 'Microsoft.Network/virtualNetworks/subnets@2023-11-01' = {
  parent: virtualNetwork
  name: 'ProtectedSubnet'
  properties: {
    addressPrefix: Subnets[3].subnetPrefix
    networkSecurityGroup: {
      id: protectedNsg.id
    }
  }
  dependsOn: [
    virtualNetwork
    protectedNsg
  ]
}
```

### 3-2.AzureポータルからCloud Shellを使ってデプロイ

Azureポータル右上の[CloudShell]のボタンを押下し、開きます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/c95696d8-2945-1e81-5bcc-39030667240a.png)
<br>

[ファイルの管理]→[アップロード]から先ほど作成したmain.bicepをアップロードします。
アップロードが終了すると、左下に「ファイルが正常にアップロードされました」と表示されます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/4c3dd017-d87e-bbf8-d1a9-471329184314.png)
<br>

CloudShellで下記のコマンドを実行し、"testRG"というリソースグループを作成します。
```
az group create --name testRG --location JapanEast
```
<br>

CloudShellで下記のコマンドを実行し、リソースグループ名とテンプレートファイル名をshellの変数に格納します。
```
RESOURCE_GROUP='testRG'
VN_TEMPLATE_FILE='main.bicep'
```
<br>

以下のコマンドを実行し、デプロイを開始します。
完了までおよそ1分かかります。
```
az deployment group create \
  --resource-group $RESOURCE_GROUP \
  --template-file $VN_TEMPLATE_FILE
```
<br>

下記のようになれば、デプロイ成功です。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f0f4ecfb-a387-5961-ec86-044e039c9110.png)

<br>

Azureポータル上でもちゃんとサブネットが4つできているか確認してみましょう。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/3a7754e6-9bc0-89e6-ed65-40ef35ac3086.png)
<br>

以上で、デプロイは完了です！

### 3-3.リソース削除

CloudShellを開き、以下のコマンドを実行します。
こちらのコマンドで今回作成したすべてのリソースが削除されます。
```
az group delete --name testRG --yes
```

## おわりに

今回は、Bicepテンプレートを使用して複数サブネットのデプロイを自動化する方法を紹介しました。
Bicepテンプレートの記述方法は色々なリソースの作成に応用できるので、皆さんもどんどん自動化にチャレンジしてみてください！
