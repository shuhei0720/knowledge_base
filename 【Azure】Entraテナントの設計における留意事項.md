今回はAzureのEntraテナントの設計における留意事項を一つご紹介したいと思います。

## Entraテナントとは
テナントとは、Azureでは組織の管理単位を示します。(○○.comなどのドメイン)
例えるのであれば、全てを纏めて入れる箱のようなものです。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/b766f7dd-c3af-47bc-a23b-8bf0ff2dd0ff.png)

テナントと組織は、常に1対1で紐づきます。
テナントの概念はMicrosoft製品全体に影響します。
Azureだけではなく、Microsoft365なども一緒に管理します。

## よくあるパターン
Entraの実装としてよくあるパターンはオンプレですでに持っているADのユーザーをEntraに同期して、ADユーザーでEntraにもログインできるようにするパターンです。
よくあるのはTeamsなどのM365を利用したいから同期用のEntraを作る企業が多いです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/8698c2f0-9bbf-4ea9-826c-abcdbd698d27.png)

## アンチパターン
上記よくあるパターンでAD同期用のEntraを使っている企業が、Azureを試験的に使いたいニーズが生まれAzure用にEntraテナントを新たにつくるパターンがよくありますが、これは**アンチパターン**です。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/a4928787-cdf4-4099-acdb-5df55a00e637.png)

Azureを試験的に使いたい場合も今後Azure基盤が大きくなる可能性を考えて、必ず既存で使っているAD同期用EntraでAzureを使うことをお勧めします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/48b55a1b-90ce-47ac-a8f6-e408bdb744cd.png)

これは[CAF(Azureを使用する場合のベストプラクティス集)](https://learn.microsoft.com/ja-jp/azure/cloud-adoption-framework/)にも記載されていますが、CAFは膨大な資料なので試験的な利用でここまで目を通す人は少ないでしょう。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/89114a33-3209-4471-a84c-825649552a03.png)
[Microsoft Entra テナントを定義する](https://learn.microsoft.com/ja-jp/azure/cloud-adoption-framework/ready/landing-zone/design-area/azure-ad-define)

## なぜアンチパターンはだめなのか
Azureを深く利用していると、ADと連携したユーザーをAzureで利用したいというニーズが必ず出てきます。(ファイルサーバーをAzureに移行したい等)。そうなった場合にテナントがわかれていると**テナント移行**していく必要があります。
このテナント移行という作業が面倒な作業になるのですが、その面倒さはのちほど解説します。
ここでは、以下にようにAzure環境を複数作成するのがなぜだめか解説します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/d75e2165-25f1-4c34-a7b7-b14a06094e49.png)

Azureを大規模に利用していくとAzure環境全体でセキュリティやガバナンスを利かせたくなります。
この際使用するサービスが、Defender for CloudとAzureポリシーというサービスなのですが、これらのサービスは基本的に**テナント内での利用**を想定しています。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/c4716e24-b805-44be-8545-9b46d34a5786.png)
ポリシーは「ストレージをパブリックにすることを禁止する」などのガバナンスや、「DNSの設定を自動で行う」など様々な仕組みを実装していくもののでテナントごとに、二重管理するのは無理があります。
Defenderに関しても、SOC運用を行うにあたって二つのDefenderポータルをまたいで調査するのが面倒になってしまします。

## テナント移行作業の困難さ
アンチパターンでテナントを実装し、Azure基盤が大きくなってきた場合、AzureをAD連携用テナントに移行し直したいというニーズが出てくることが往々にしてあります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3585159/e1812ae9-e3c2-4ed5-a5f3-cd2bb3047d42.png)

私はこのテナント移行のプロジェクトを経験したのですが、この作業は以下の理由で非常に困難です。
- ユーザーはEntraで管理されているので、ユーザーに割り当てた権限など全て作り直し
- 一部のAzureリソースはEntraと密接に連携して動作するものもあるため、移行が破壊的な変更になるリソースがある(リソース作り直し)
- ポリシー、Defender設定等も作り直し
- Azure上に存在する全リソースに対して影響調査し、移行作業計画を作らないといけない。(大きいところだと100を超えるサブスクリプションもあるでしょう。その中の全てのリソースの影響調査となると気が遠くなります。)

## 伝えたいこと
すでにM365を利用していてEntraテナントを持っている場合、Azureもそのテナントで利用開始しましょう、、、!

