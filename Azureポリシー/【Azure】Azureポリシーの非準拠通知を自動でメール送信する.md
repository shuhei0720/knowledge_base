今回は、AzureでAzureポリシーの非準拠通知をLogic appsを使って対象サブスクリプションの管理者に通知する仕組みを作成します。

## Azure Logic Appsとは
コードを記述せずに、自動化されたワークフローを作成できるサービスです。
事前構築済みのトリガーやコネクタを組み合わせることで、**タスクの自動化やスケジュール設定、メールの自動送信などを簡単に実現**できます。

## 他サービスとの比較
Microsoftは、Azure Logic Appsのほかにも多くのサービスを提供しており、混同しやすいサービスもあります。その違いについてみていきましょう。
### 1. Azure Functions/Azure Automationとの違い
Azure Logic AppsとAzure Functions/Azure Automationは、どれもサーバーレスでオーケストレーションができるサービスなので、混同しやすいですが、**コードを使用するかしないかが大きな違いです。**

・Azure Functions/Azure Automation：プログラミング言語を使ってアプリを作成するためのAzureサービス
・Azure Logic Apps：ノーコード・ローコードでアプリ開発を行うサービス

### 2. Microsoft Power Automateとの違い
Azure Logic AppsとMicrosoft Power Automateは、どちらもワークフローができるサービスです。
その違いは使用する用途の違いです。

Azure Logic Appsはシステム開発などに使用されるのに対し、Microsoft Power Automateは企業の業務プロセスの自動化などに使用されます。

また、Azure Logic Appsを元にMicrosoft Power Automateは作られているため、操作性など似ている点が多いと言われています。

## Logic appsの料金について
Logic appsのホスティングオプションは「従量課金」と「Standard」の二つの選択肢があります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/31eba5f4-22b3-44f6-a439-98a850b37a5e.png)
この違いについて、説明します。

### 従量課金
従量課金 タイプでは、基盤の管理をMicrosoftに完全に委任できます。
メリットとしては、基盤を管理することなく、よりSaaSに近い使い方ができるのがこのプランになります。
デメリットは、他テナントの利用者と共用でリソースを利用しているため自分でリソースのサイジングが出来ないことや、VNet 統合といった Azure リソースを利用してアクションを実行することが出来ないことが挙げられます。
今回のように、自動化タスクを作成する場合はこちらのプランが推奨されます。

### Standard
Standard タイプでは基盤部分に App Service および Functions を利用しているため、それらの機能を利用したワークフローを構築することが可能となります。
つまり、VNet 統合で VNet 上のリソースに対してアクションを実行したり、プラットフォーム部分について自分でリソースのサイジングが出来るので、パフォーマンスチューニングをしやすいなどのメリットがあります。
デメリットとしては、基盤としてAzure FunctionsやApp Service、Azure Kubernetesを利用するので、リソースに対して、それらを使用しているかどうかに関係なく課金されるので、高額になりやすいことが挙げられます。
将来的にAzure FunctionやApp Serviceへの移行が考えられるアプリを作成する場合などに使用します。

プランの詳細については、以下をご覧ください。
[Azure Logic apps 課金と価格のモデル](https://learn.microsoft.com/ja-jp/azure/logic-apps/logic-apps-pricing)


## ハンズオン構成イメージ
今回構成するLogic appsは以下のようなイメージです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/7a825789-aa84-46e0-b8bf-d849658ca9bb.png)

①システム割り当てマネージドIDで認証し、Azure Monitorにクエリを実行
　※KQLでポリシー非準拠リソースとその情報を取得
②クエリ結果をJSONで回答
③システム割り当てマネージドIDで認証し、Azure Resource Managerコネクタ経由を使用してREST API(読み取り)を実行
　※②の取得結果からサブスクリプションIDを取得し、サブスクリプションのロールの割り当て一覧を取得
④リソース読み取り結果をJSONで回答
⑤システム割り当てマネージドIDで認証し、HTTPコネクタを使用してGraph API(読み取り)を実行
　※④の取得結果から、ロールが割り当てられているユーザーのメールアドレスを取得
⑥Graph API読み取り結果をJSONで回答
⑦Outlook.comコネクタを使用してこれまで取得したJSONからメールを組み立て、送信

## ハンズオン

### Logic appsの作成
1. Azureポータルで「logic apps」と検索し、[logic apps]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/462d3075-17c5-43ff-9253-b45355f00fdd.png)

2. 「Add」を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/bb3936f0-dcf0-4538-b359-cccac15bc6ac.png)

3. 従量課金の「マルチテナント」を選択し、[選択]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/0947dd75-09f4-4451-99eb-186c62b40b5a.png)

4. 必要事項を入力し、「確認および作成」を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/5ebfebb8-e534-45ec-a3e5-b1cd3f2a033d.png)

5. [作成]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/02159386-7d9b-4285-a099-9b7ecd1eac71.png)

### Logic apps用のマネージドID作成
1. 先ほど作成したデプロイが完了したら[リソースに移動]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/2a3b82e0-51ef-403f-ae36-dc6452b93526.png)

2. 左ペインの「設定」配下の[ID]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/96484ec5-0b5f-48f0-be28-d047bad80857.png)

3. システム割り当てマネージドIDを「オン」にし、[保存]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/0107e1d9-8457-4f1c-b645-e8673b2841f0.png)

### マネージドIDへのRBAC権限割り当て
1. システム割り当てマネージドIDの「オブジェクトID」をコピーしておく
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/a75f0eff-f3cd-4a72-ae05-791cb6b0a585.png)

2. 上部検索バーで「管理グループ」と検索し、[管理グループ]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/6519339d-d299-43ad-8d10-dcbb8e970095.png)

3. 通知を行いたいサブスクリプションを全て含む管理グループを選択
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/bd1a2b6f-b0c8-4cc8-a472-106b3038c42e.png)

4. [アクセス制御(IAM)]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/5ffa676d-d01c-4f99-a31a-a195192b493f.png)

5. 「ロールの割り当て」タブを開き、[追加]→[ロールの割り当ての追加]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/fc629b39-0dc8-46da-b190-135a1ebfbb49.png)

6. 「ロール」タブで[閲覧者]を選択し、[次へ]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/95c13264-eea6-4425-aad8-eb9b0072c20e.png)

7. 「メンバー」タブの「アクセスの割当先」で[マネージドID]を選択し、「メンバー」の[メンバーを選択する]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/34e5d15e-7989-42d0-bee1-40b287ca0ba3.png)

8. 先ほど作成したLogic appsのシステム割り当てマネージドIDを選択し、[選択]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/333d3a37-c0d1-4725-bef1-eb48ff350060.png)

9. [レビューと割り当て]を押下し、そのまま割り当てを作成する
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/7b14fe63-9b16-45c2-ade5-98eb42bd98c3.png)

### マネージドIDへのEntra権限の割り当て
1. Azureポータル上部の検索バーにて「entra」と検索し、[Microsoft Entra ID]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/962eee99-967b-4572-882f-61309437ea53.png)

2. 左ペインの「管理」配下の[ロールと管理者]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/2f88d470-8b35-4910-a4b0-1e31490ffc8d.png)

3.  検索ボックスで「ディレクトリ閲覧者」と検索し、[ディレクトリ閲覧者]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/acf88e48-2d9b-4882-b22d-8840d3c0e547.png)

4. [割り当ての追加]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/76bf6d45-a1f1-4668-8bb4-bd923e67b4fb.png)

5. 検索ボックスで、先ほどコピーしたLogic apps用のシステムマネージドIDのオブジェクトIDを検索する。エンタープライズアプリケーションがヒットするので、選択し、[追加]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/67668813-3dec-451d-95b2-df770c4c1dd7.png)

### Logic appsの編集
1.最初に作成したLogic appsの画面に移動し、左ペインの「開発ツール」配下の[ロジックアプリデザイナー]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/bbfff5de-d57d-4a5e-93e0-4d4dfb4299a6.png)

2.[トリガーの追加]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/83bbd49c-fedc-4cc2-9682-d347231fe438.png)

3.「Built-in tools」の[Schedule]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/32439b6a-ed89-4d63-9546-aefe51877351.png)

4.[Recurrence]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/572a4507-50d1-40a1-925a-6da0aa8b62b3.png)

5.以下のように通知頻度を入力する(サンプルでは、毎週月曜日0:00に実行するようにしている)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/376b5384-c94e-434d-8e99-edf8cfc99b49.png)

6.[Recurrence]の下の[+]→[アクションの追加]を押下する
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/86f7ffef-afff-4978-9c95-966236b1d3fc.png)

7.「アクションの追加」画面の検索ボックスで「クエリ」と検索し、[クエリを実行して結果を一覧表示する V2]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/dbc6ea20-2c52-4431-b73d-3b28fc3a9690.png)

8.「Create connection」画面で「認証の種類」を[Logic AppsのマネージドID]に設定し、[Create new]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/73644a35-54af-4844-87a7-7be1bca80119.png)

9.表示された「パラメータ」タブで以下の情報を入力
※<ポリシー割り当てID>はイニシアティブの割り当てIDを想定しています

| パラメータ名 | 設定値 |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| クエリ | arg("").PolicyResources <br>\| where type == 'microsoft.policyinsights/policystates' <br>\| where properties.policyAssignmentId == "&lt;ポリシーまたはイニシアティブ割り当てID&gt;" <br>\| where properties.complianceState == 'NonCompliant' <br>\| extend NonCompliantResourceId = properties.resourceId, PolicyAssignmentName = properties.policyAssignmentName <br>\| project subscriptionId, resourceGroup, properties.policyDefinitionReferenceId, properties.complianceState, properties.resourceType, properties.resourceId |
| サブスクリプション | 検索対象のLog Analyticsワークスペースが存在するサブスクリプション |
| リソースグループ | 検索対象のLog Analyticsワークスペースが存在するリソースグループ |
| リソースの種類 | Log Analytics Workspace |
| リソース名 | 検索対象のLog Analyticsワークスペース |
| 時間範囲の種類 | Relative |
| TimeRange | Last 24 hours |

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f86c8432-84ee-4ba8-8619-7376160e9347.png)

10.「クエリを実行して結果を一覧表示する V2」の下の[+]→[アクションの追加]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/6f09dbdd-2829-490d-af2d-389b4df40d88.png)

11.「アクションの追加」画面で[Control]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e34a6737-9a14-4d85-81ae-ac0c8a78b32d.png)

12.[For each]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/502efcd0-c2a8-4ecf-b24c-d6f6532e4ef1.png)

13.「For each」画面の「パラメータ」タブで「Select An Output From Previous Steps」のテキストボックスを押下し、右側に表示される[fx]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/8149a794-de9c-4c69-8c62-37660753e058.png)

14.[動的なコンテンツタブ]を選択し、上部のテキストボックスに以下のように入力する
※「クエリを実行して結果を一覧表示する_V2」アクションの出力結果(各リソースのポリシーの非準拠の配列に対してループを実行する
```
body('クエリを実行して結果を一覧表示する_V2')?['value']
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/23ff6cf0-c080-407d-bd69-ed577778516b.png)

15.デザイナー画面の「For each」配下の[+]→[アクションの追加]を押下する
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/d243e6eb-3523-452a-9cae-d76effdc4a63.png)

16.「アクションの追加」画面の検索ボックスで「プロバイダー内のリソースを読み取る」と検索し、[プロバイダー内のリソースを読み取る]を押下する
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e025a0f3-7582-41ad-9fc0-86d11cf19058.png)

17.「Create connction」画面の「認証」で[Managed identity]を選択する
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/30e656e5-30e9-4984-803d-3316a6a0dade.png)

18.そのまま「Create new]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/1f56ee00-880e-48e3-a409-2720842544e6.png)

19.「プロバイダー内のリソースを読み取る」画面で以下の情報を入力
※サブスクリプションはFor eachで現在回している配列内のサブスクリプションIDを指定しています
| パラメータ名 | 設定値 |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| サブスクリプション<br>※fxで動的コンテンツを入力 | ```item()?['subscriptionId']``` |
| リソースプロバイダー | Microsoft.Authorization |
| 短いリソースID | roleAssignments |
| クライアントAPIバージョン | 2022-04-01 |

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/99a6ba2a-05d7-43b3-ade8-83390fd7c6fb.png)

20.デザイナー画面の「プロバイダー内のリソースを読み取る」の下の[+]→[アクションの追加]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/81d0c74d-989b-4387-87c4-36655aa080ae.png)

21.「アクションの追加」画面で[Control]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/791a6cd5-9bd8-479e-874a-69c4b49b6e1f.png)

22.[For each]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/06519f32-a380-40bb-8a9b-1b81c4a8144f.png)

23.「For each 1」画面の「パラメータ」タブで「Select An Output From Previous Steps」のテキストボックスを押下し、表示された[fx]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/9c698345-df4c-43fa-9ac1-0c88c921aaab.png)

24.「動的なコンテンツ」を押下し、上部のテキストボックスに以下のように入力し、[追加]を押下する
※「プロバイダー内のリソースを読み取る」で取得した各RBAC割り当てに対してループを回します
```
body('プロバイダー内のリソースを読み取る')?['value']
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ad3c075d-5b7f-496f-b87a-7856b2bce1fe.png)

25.デザイナー画面の「For each 1」配下の[+]→[アクションの追加]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/80c190eb-97d4-45de-97fc-3929a11692a7.png)

26.「アクションの追加」画面で[Control]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/3cb02971-7b31-400a-94b6-8f48f86de2ae.png)

27.[Condition]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f6b79cfb-ed22-47b6-b4a6-f87d68404768.png)

28.ここでは、以下のような条件を指定していく
```サブスクリプションのRBAC割り当てから読み取ったユーザーに対して、PrincipalTypeがUserかつ、割り当てられているロールがOwnerまたはUser AdministratorであればTrue```

一番上の「値を選択してください」テキストボックスを押下し、表示される[fx]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/c25fa6bc-8d6f-4a17-91bc-eb415b4253c7.png)

29.以下のように入力し[追加]を押下
```items('For_each_1')?['properties']['principalType']```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/2a83c78d-edb9-414f-819e-d64de5dc3420.png)

30.「is equal to」の右側には```User```と入力する
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/d3c5af7f-04b9-4c91-95a4-a892a68d72a3.png)

31.[New item]→[Add group]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/19c99c4f-4ba3-423c-999a-0c34f4f685b4.png)

32.表示された「値を選択してください」のテキストボックスを押下し、[fx]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f3a4bac7-5ecf-4be2-b0d5-3216b8522d72.png)

33.以下のように入力し、[追加]を押下
```items('For_each_1')?['properties']['roleDefinitionId']```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/324a89d7-ce50-4e2f-bfc3-ace202a87651.png)

34.「is equal to」の右側のテキストボックスには以下のように入力します
※OwnerロールのIDです
```/providers/Microsoft.Authorization/roleDefinitions/8e3af657-a8ff-443c-a75c-2fe8c4bcb635```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/f2ce7ee2-d108-46bf-857d-cb182ab4ae57.png)

35.以下の画像のように[New item]→[Add row]を入力する
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/47a39e8b-38d7-4e3a-857a-c939903cd56b.png)

36.表示された「値を選択してください」のテキストボックスを押下し、[fx]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/daf66803-0b9c-4dbd-90ba-a9f351f2a5db.png)

37.以下のように入力し、[追加]を押下する
```items('For_each_1')?['properties']['roleDefinitionId']```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/81dbd0c7-9a47-4f30-a34a-b00fb20be094.png)

38.「is equal to」の右側のテキストボックスには以下のように入力します
※User AdministratorロールのIDです
```/providers/Microsoft.Authorization/roleDefinitions/18d7d88d-d35e-4fb5-a5c3-7773c20a72d9```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/86de2d29-843b-445b-9b64-4e70a2a29ef7.png)

39.下図のように、黄線部の条件式を[OR]、黄線部の条件演算子を[contains]に変更する
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/12bc21f5-0a07-4ce7-86e5-29ead516fb43.png)

40.デザイナー画面の「True」配下の[+]→[アクションの追加]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/51bb9fc8-4926-4b39-9677-412a167c7660.png)

41.「アクションの追加」画面で[HTTP]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/3ee2bc5a-f5ea-4478-be8f-bda8d7cb5eab.png)

42.[HTTP]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/41abd0e5-dd25-48b6-8d69-d1fee766f163.png)

43.「HTTP」画面で「パラメーター」タブの「URI」テキストボックスに以下のように入力する
```https://graph.microsoft.com/v1.0/users/```
テキストボックスの一番右側にカーソルがある状態で[fx]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/54fb071c-2dfe-49c1-82fa-ceff7108a55d.png)

44.以下のように入力し、[追加]を押下する
```items('For_each_1')?['properties']['principalId']```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/1a5e0fe6-b020-4b1a-b65e-dbef6fbc7d00.png)

45.「Method」は[GET]を選択する
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/04df0d06-b338-46e9-8d24-e12a9d2b2d10.png)

46.「詳細パラメーター」欄の[Authentication]を選択する
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ad8bd50f-d1c0-4527-bdcb-794aca71e443.png)

47.「Authentication Type」が表示されるので、押下する
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/67835502-e55b-4c4f-b5bc-a029377d8e8c.png)

48.[Managed identity]を選択
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/27dd37f4-0dbe-4556-ad1a-fc6e12c6dd86.png)

49.「Audience」欄に以下のように入力
```https://graph.microsoft.com```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/54ea3f92-68f1-44ee-b3a1-05a1657c246f.png)

50.デザイナー画面の「HTTP」の下の[+]→[アクションの追加]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/ebc17af7-9fed-4013-824e-366756d64fd1.png)

51.「アクションの追加」画面の検索ボックスで「Office」と検索し、[メールの送信(V2)]を押下
**以下の操作から、通知メールの送信元となるユーザーでAzureポータルにログインし、操作する必要がある**
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/cdc0a7d8-3d19-4200-9415-eacc2989a96b.png)

52.[サインイン]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/06cf7991-8afb-497d-9417-66a1ab92dc8d.png)

53.「アカウントにサインイン」画面に移動するので、メールの送信元としたいアカウントでサインインする
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/6a4e7e65-3673-4114-8535-a83eb4320f33.png)

54.サインインに成功すると「メールの送信(V2)」の画面が開くので、以下のように入力する

| パラメータ名 | 設定値 |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 宛先<br>※fxを選択して入力 | ```body('HTTP')?['mail']``` |
| 件名 | 【通知】ポリシー違反リソース確認のお願い |
| 本文<br>※背景が灰色の箇所はfxを選択して入力 | お世話になっております。Azure基盤チームです。<br>利用していただいているサブスクリプションで、以下のリソースに対してポリシー違反警告が出ております。<br>サブスクリプションID：```items('For_each')?['subscriptionId']```<br>リソースグループ名：```items('For_each')?['resourceGroup']```<br>ポリシー名：```items('For_each')?['properties_policyDefinitionReferenceId']```<br>準拠状況：```items('For_each')?['properties_complianceState']```<br>リソースの種類：```items('For_each')?['properties_resourceType']```<br>リソースID：```items('For_each')?['properties_resourceId']```<br>ご確認、ご対応よろしくお願いいたします。<br>以上 |

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/769a7164-6e4b-4c0c-86a0-b51f7225d4bd.png)

55.デザイナーの画面に戻り、[Save]を押下

### Logic appsのテスト
1.デザイナーの画面で[Run]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/63a4dec5-ee95-4fd3-b34e-fd143e74fa3e.png)

2.左ペインの[実行履歴]を押下すると、実行ログが表示されるので先ほど実行したログを押下する
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/a27cfc8f-8c89-42e3-ac0d-e81d2b4061c3.png)

3.各ステップの実行内容が詳細に確認できる
例えば、[クエリを実行して結果を一覧表示する V2]を押下
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/fd2ff9d1-ddb1-47ce-9c29-6e4298eb3446.png)

4.すると、入力された情報や出力されたJSONの情報が詳細に確認できる
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/c2eeb65b-e0ab-4b63-b8a5-f5f65f419891.png)

