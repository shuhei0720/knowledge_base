## カスタムポリシーで実現できる要件
Azureポリシーは、Azure内のリソースのプロパティの状態を監査し、定義した状態と異なる場合はARMテンプレートを使って自動でリソースを修正することができます。

ですので、例えば以下のような事例で活用できます。
- デプロイされたリソースに指定したタグと値が無かったら、タグを追加する
- VMがデプロイされた時、指定の拡張機能が無かったら、拡張機能を追加する

例えば今回は以下のような要件を定義します。
```
Azure環境にプライベートDNSゾーンが作成されたら全て、HubVnetに仮想ネットワークリンクを作成したい。
```

## カスタムポリシーを開発する前にやるべきこと
Azureポリシーは**組み込みのポリシーで実現できるならば、MSにポリシーの運用を任せられるので基本的に組み込みポリシーを使った方が良い**です。
なので、まずは要件を実現できる組み込みポリシーが無いか探します。
以下のように探します。
- Azureポータルの「ポリシー」→「定義」→キーワードを入力して検索
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/c49664ae-3fd7-4c03-b44d-5cd1556ef232.png)
- [AzAdvertizer](https://www.azadvertizer.net/azpolicyadvertizer_all.html)でキーワードを入力して探す
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e8f6c1a0-ce4c-41f2-b2dc-004e9f32549f.png)

AzAdvertizerはコミュニティが開発しているカスタムポリシーが多数あり、カスタムポリシーの作成時も参考にできます。

## カスタムポリシーのテンプレート
まずは、以下のテンプレートを使います。ここから作成していきます。
```json
{
  "mode": "<mode>",
  "parameters": {
    <parameters>
  },
  "policyRule": {
  "if": {
    <rule>
  },
    "then": {
      "effect": "<effect>"
    }
  }
}
```

## カスタムポリシー作成
空のテンプレートで作成したプロパティを説明しながらカスタムポリシーを作っていきます。

### ①メタデータ
ポリシーに関するメタデータを記載します。以下のようなものがあります
- mode
評価されるリソースの種類を定義します。
`"all"` : リソース グループ、サブスクリプション、およびすべてのリソースの種類を評価します
`"indexed"` : タグと場所をサポートするリソースの種類のみを評価します
例えば、indexedモードでは、タグがつけられない「Microsoft.Network/routeTables/routes」などは評価されません。要件が無ければ、「all」を使用することが推奨です。
メタデータにはほかにも、表示名を指定する「displayname」,説明を記載する「description」,任意のメタデータを記載する「metadata」などがあります。

例えば、以下のように記載します。
```json
{
  "mode": "all",
```

### ②パラメータ
ポリシー内で使用するパラメータを記載します。
`"name"` : パラメータの名称
`"type"` : パラメータの型(string,array,object,integer等)
`"metadata"` : ユーザーに表示されるパラメータの説明(description,displayname等)
`"defaultValue"` : 値が指定されなかった場合のデフォルト値
`"allowedValues"` : パラメータに指定できる値のリスト

例えば、以下のように記載します。
```json
{
  "mode": "all",
  "parameters": {
    "virtualNetworkResourceId": {
      "type": "Array",
      "metadata": {
        "displayName": "Vnet リソースID",
        "description": "VnetのリソースIDを次のように指定してください: ['/subscriptions/{subscription id}/resourceGroups/{resourceGroup name}/providers/Microsoft.Network/virtualNetworks/{virtual network name}]'"
      },
      "defaultValue": []
    },
    "registrationEnabled": {
      "type": "Boolean",
      "metadata": {
        "displayName": "自動登録を有効化するか",
        "description": "仮想ネットワークリンクの自動登録オプションを有効化するかどうか"
      },
      "defaultValue": false
    },
    "effect": {
      "type": "String",
      "metadata": {
        "displayName": "Effect",
        "description": "ポリシーの効果"
      },
      "allowedValues": [
        "DeployIfNotExists",
        "Disabled"
      ],
      "defaultValue": "DeployIfNotExists"
    }
  },
```
### ③ポリシールール
このプロパティは「if」と「then」で構成されます。
「then」には「if」が満たされた場合の効果を定義します。

#### (1) if
「if」にはポリシーをいつ適用するかの条件を定義します。

##### 1) 論理演算子
`"not"` : 条件を反転させる(true→false)
`"allOf"` : すべての条件がtrueならばtrue(and条件)
`"anyOf"` : 1つの条件がtrueならばtrue(or条件) 

##### 2) 条件
`"equals"` : 指定した値が等しい場合true
`"like"` : "Microsoft.*"のようにワイルドカードで値を指定して比較する
`"match"` : 正規表現で指定して比較する
`"contains"` : 指定した値が含まれる場合はtrue
`"in"` : 指定した配列の中に含まれる場合はtrue
`"exists"` 指定したプロパティが存在する場合はtrue

##### 3) フィールド
条件で比較する対象のリソースのプロパティを指定する
`name` : リソースの名前
`kind` : リソースの種類
`type` : リソースタイプ
`id` : リソースID
`tags` タグの配列
`tags['<tagName>']` : 指定したタグの値


```json
{
  "mode": "All",
  "parameters": {
    <parameters>
  },
  "policyRule": {
    "if": {
      "allOf": [
        {
          "equals": "Microsoft.Network/privateDnsZones",
          "field": "type"
        }
      ]
    },
  }
}
```

#### (2) then
「if」が満たされた時に、行う動作や作成するリソースを定義します。

##### 1) effect
ポリシーが行う動作の種類を指定します。
`audit`：準拠状況を表示する。是正は行わない。(デプロイ前に評価)
`auditIfNotExists`：準拠状況を表示する。是正は行わない。(デプロイ後に評価)
`deny`：条件を満たさない場合、デプロイを拒否する。
`append`：条件を満たさない場合、デプロイ時にプロパティを追加する。
`modify`：条件を満たさない場合、デプロイ時にプロパティを修正する。
`★deployIfNotExists`：条件を満たさない場合、指定したARMテンプレートをデプロイする(デプロイ後に評価)

以下では、deployIfNotExistsのプロパティを説明します。

##### 2) existenceCondition
デプロイ対象のリソースのあるべき状態を指定します。
「if」条件と同じ構文を使用します。
この状態を満たしていない場合、**Azureポータルで非準拠と表示**されます。

##### 3) type
デプロイ対象のリソースの種類を定義します。

##### 4) roleDefinitionIds
ポリシーがリソースをデプロイする時に必要なRBAC権限を指定します。

##### 5) deployment
条件を満たさなかったときのデプロイの内容を記載します。
配下のpropertiesでデプロイのプロパティを記載します。

###### 1. mode
ARMテンプレートのデプロイモードを指定します。
ARMテンプレートはリソースグループに対してデプロイを行いますが、そのリソースグループに存在する既存のリソースへの影響が変わります。
デプロイモードは以下の2種類があります。

`complete`：このモードでは、ARMテンプレートで指定したリソースがリソースグループ内の完全な状態と認識され、テンプレートに記載されていない既存のリソースがリソースグループに存在した場合は**削除されます。**
`increment`：このモードでは、ARMテンプレートで記載されていない、既存のリソースがリソースグループに存在した場合、**削除**はされません。
※ARMテンプレートで既存のリソースが記載されている場合の**変更**は行われます。
※変更の場合も、既存リソースの**一部のプロパティだけ更新する**ことは**できません。** 既存のリソースのプロパティも含めて全てのプロパティを最終的な状態として指定する必要があります。

###### 2. parameters
デプロイで指定するパラメータを記載します。
ここにパラメータを記載することで、ポリシーが取得した値をARMテンプレートの中に渡すことができます。

###### 3. template
ここにデプロイするARMテンプレートを記載します。
以下では、ARMテンプレートのプロパティについて解説します。

###### 4. schema
テンプレート言語のバージョンを指定します。決まり文句です。

###### 5. contentVersion
テンプレートのバージョンを指定します。テンプレートをバージョン管理している場合はこの値を更新していきます。

###### 6. parameters
テンプレート内で使用するパラメータを記載します。templateセクションで指定したパラメータ名と同じパラメータ名を記載します。(外側で指定したパラメータが渡す側だとしたら、これはパラメータの受け取り側だと思ってください。)

###### 7. resources
リソースの各プロパティを指定します。
リソースごとのプロパティは以下のように調べます。

- [Azureリソースリファレンス](https://learn.microsoft.com/ja-jp/azure/templates/)ページへ移動する
- 検索バーで作成したいリソース名・リソースの種類等で検索する ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/6ba0b5b1-b96c-4ba3-8f33-2c65562ab30b.png)
- 「この記事では、xxx」のリンクを押下する ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/b848ff53-6a85-4589-99ab-c07bffa6d89b.png)
- 右側のメニューから辿って目的のリソースを探していく ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/bfd3264a-2688-44da-9c10-8316c7e10b01.png)
- 「リソースの形式」欄にテンプレートの書き方が書いてある ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/bfe13eb2-98a6-4201-a7f8-6f6280e08419.png)
- 「プロパティ値」欄に各プロパティの書き方が書いてある 
※表記の日本語がおかしい場合は英語で表示するのが無難です ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e0faff58-ab84-4a52-9a48-b776270af770.png)

以上で「then」セクションの記載は完了です。
以下、「then」セクションを記載したテンプレートの例を示します。

```json
{
  "mode": "All",
  "parameters": {
    <parameters>
  },
  "policyRule": {
    "if": {
      <rule>
    },
    "then": {
      "effect": "[parameters('effect')]",
      "details": {
        "existenceCondition": {
          "allOf": [
            {
              "equals": "Microsoft.Network/privateDnsZones/virtualNetworkLinks",
              "field": "type"
            },
            {
              "in": "[parameters('virtualNetworkResourceId')]",
              "field": "Microsoft.Network/privateDnsZones/virtualNetworkLinks/virtualNetwork.id"
            }
          ]
        },
        "type": "Microsoft.Network/privateDnsZones/virtualNetworkLinks",
        "roleDefinitionIds": [
          "/providers/microsoft.authorization/roleDefinitions/b12aa53e-6015-4669-85d0-8515ebb3ae7f"
        ],
        "deployment": {
          "properties": {
            "mode": "incremental",
            "parameters": {
              "privateDnsZoneName": {
                "value": "[field('name')]"
              },
              "virtualNetworkResourceId": {
                "value": "[parameters('virtualNetworkResourceId')]"
              },
              "registrationEnabled": {
                "value": "[parameters('registrationEnabled')]"
              }
            },
            "template": {
              "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
              "contentVersion": "1.0.0.0",
              "parameters": {
                "privateDnsZoneName": {
                  "type": "String"
                },
                "virtualNetworkResourceId": {
                  "type": "array"
                },
                "registrationEnabled": {
                  "type": "bool"
                }
              },
              "resources": [
                {
                  "type": "Microsoft.Network/privateDnsZones/virtualNetworkLinks",
                  "apiVersion": "2018-09-01",
                  "name": "[concat(parameters('privateDnsZoneName'),'/',concat(parameters('privateDnsZoneName'),'-', last(split(parameters('virtualNetworkResourceId')[copyIndex()],'/'))))]",
                  "location": "global",
                  "copy": {
                    "name": "vnetlink-counter",
                    "count": "[length(parameters('virtualNetworkResourceId'))]"
                  },
                  "properties": {
                    "registrationEnabled": "[parameters('registrationEnabled')]",
                    "virtualNetwork": {
                      "id": "[parameters('virtualNetworkResourceId')[copyIndex()]]"
                    }
                  }
                }
              ]
            }
          }
        }
      }
    }
  }
}
```

