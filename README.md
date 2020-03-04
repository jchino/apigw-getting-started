# Oracle Cloud Infrastructure API ゲートウェイ - チュートリアル

Oracle Cloud Infrastructure (OCI) では、API ゲートウェイ・サービスを提供しています。
OCI API ゲートウェイ・サービスを使用することで、OCI 上のリソースをセキュアに REST API として公開することができます。

このチュートリアルでは OCI API ゲートウェイ・サービスを使用して、OCI が提供している Function as a Service (FaaS) である Oracle Functions のファンクションを REST API として公開する手順を説明します。

## このチュートリアルで作成するアプリケーション

![このチュートリアルで作成するアプリケーション][overview]

[overview]: images/overview.png "このチュートリアルで作成するアプリケーション"

OCI 上にリソースを作成する際は必ず、どの ***コンパートメント*** に所属するかを指定する必要があります。

このチュートリアルでは、1つのコンパートメントに API ゲートウェイや関連するネットワークのコンポーネント（仮想クラウド・ネットワーク、サブネット、インターネット・ゲートウェイ）、Oracle Functions のファンクションを作成する前提で説明を進めます。

### IAM ポリシーの設定

OCI では、 Identity and Access Management （IAM） と呼ばれる仕組みによって各種リソースへのアクセスを制御します。
このチュートリアルの手順にしたがって API をデプロイするためには、関連するリソースへの操作を許可するポリシーを定義する必要があります。

`Administrators` グループには、テナント内のすべてのリソースを管理することができるポリシー `Tenant Admin Policy` が適用されています。
そのため `Administrators` グループに所属しているユーザーがこのチュートリアルを実施する場合は、ユーザーのためのポリシーを設定する必要はありません。
トライアル環境の場合、トライアルに申し込んだユーザーは自動的に `Administrators` グループに所属しています。

個別のリソースに対して許可する操作を設定する場合は、次のようなポリシーを設定する必要があります。

- API ゲートウェイ関連リソースの管理操作を許可
- ネットワーク関連リソースの管理操作を許可
- Oracle Functions 関連リソースの管理操作を許可

OCI API ゲートウェイ・サービスを使用するために必要な IAM ポリシーの設定は、API ゲートウェイのドキュメントのポリシーの作成に関するページ （[英語版][creatingpoliciess_en]/[日本語機械翻訳版][creatingpoliciess_ja]） を参照してください。

[creatingpoliciess_en]: https://docs.cloud.oracle.com/en-us/iaas/Content/APIGateway/Tasks/apigatewaycreatingpolicies.htm "API Gateway - Create Policies to Control Access to Network and API Gateway-Related Resources"
[creatingpoliciess_ja]: https://docs.cloud.oracle.com/ja-jp/iaas/Content/APIGateway/Tasks/apigatewaycreatingpolicies.htm "API ゲートウェイ - ネットワークおよびAPIゲートウェイ関連リソースへのアクセスを制御するポリシーの作成"

### このチュートリアルの流れ

1. [仮想クラウド・ネットワーク (VCN) の作成][create_vcn]
1. [Oracle Functions アプリケーションとファンクションのデプロイ][deploy_functions]
1. [API ゲートウェイのインスタンスの作成][create_instance]
1. [API のデプロイ][deploy_api]
1. [API のテスト][test_api]
1. [ログ・ファイルのダウンロード][download_logfiles]

[create_vcn]:        #section1 "仮想クラウド・ネットワーク (VCN) の作成"
[deploy_functions]:  #section2 "Oracle Functions アプリケーションとファンクションのデプロイ"
[gw_policies]:       #section3 "API ゲートウェイから Oracle Functions のリソースへのアクセス"
[create_instance]:   #section4 "API ゲートウェイのインスタンスの作成"
[deploy_api]:        #section5 "API のデプロイ"
[test_api]:          #section6 "API のテスト"
[download_logfiles]: #section7 "ログ・ファイルのダウンロード"

## 仮想クラウド・ネットワーク (VCN) の作成

仮想クラウド・ネットワーク (Virtual Cloud Network; VCN) は、OCI に作成するプライベート・ネットワークです。
API ゲートウェイのインスタンスを作成し、API をデプロイするためには、事前に VCN を作成する必要があります。

API ゲートウェイのインスタンスは、リージョナル・サブネット（特定の可用性ドメインに関連付けられていないサブネット）を使用する必要があります。
また、インターネットからのリクエストを受け付けるためにはインターネット・ゲートウェイが必要です。

このチュートリアルでは、Quickstart ワークフローを使用して VCN を作成する手順について説明します。
Quickstart ワークフローを使用することで、API ゲートウェイ・サービスを使用するために必要なリージョナル・サブネットやインターネット・ゲートウェイを作成し、構成することができます。

1.  OCI コンソールにログインします。
    画面左上のハンバーガー・アイコンをクリックし、 **「コア・インフラストラクチャ」** にある **「ネットワーキング」** → **「仮想クラウド・ネットワーク」** を選択します。

1.  **「仮想クラウド・ネットワーク」** のページが表示されます。
    画面左側の **「リスト範囲」** の下の **「コンパートメント」** で、VCN を作成するコンパートメントを選択します。

1.  **「ネットワーキング Quickstart」** ボタンをクリックします。
    **「ネットワーキング Quickstart」** ボックスが表示されたら **「インターネット接続性を持つ VCN」** を選択し、 **「ワークフローの開始」** ボタンをクリックします。

1.  **「インターネット接続性を持つ VCN の作成」** - **「構成」** 画面が表示されます。
    **「基本情報」** には、VCN の名前と作成するコンパートメントを指定します。

1.  **「VCN とサブネットの構成」** には、次のようにVCNとサブネットの CIDR ブロックを指定します。

    + **「VCN CIDR ブロック」** : `10.0.0.0/16`
    + **「パブリック・サブネット CIDR ブロック」** : `10.0.0.0/24`
    + **「プライベート・サブネット CIDR ブロック」** : `10.0.1.0/24`
    + **「このVNCでDNSホスト名を使用」** : チェックする

    入力し終わったら **「次へ」** ボタンをクリックします。

1.  **「インターネット接続性を持つ VCN の作成」** - **「確認および作成」** 画面が表示されます。
    ここまでに入力した内容に応じて作成されるコンポーネントが確認できます。

### パブリック・サブネットのセキュリティ・リストの更新

このチュートリアルで作成する API ゲートウェイは、インターネット経由のリクエストを受け付けるようにします。
HTTPS リクエストを受け付けることができるようにパブリック・サブネットのセキュリティ・リストを更新する必要があります。

1.  作成した VCN の詳細ページを開き、画面左側の **「リソース」** メニューで **「サブネット」** を選択します。

1.  サブネットのリストから **「パブリック・サブネット-${VCN名}」** をクリックします。

1.  パブリック・サブネットに適用されているセキュリティ・リスト **「Default Security List for ${VCN名}」** をクリックします。

1.  **「イングレス・ルールの追加」** ボタンをクリックします。
    次のルールを追加します。

    + **「ソース・タイプ」** : **「CIDR」** を選択 （デフォルト値）
    + **「ソース CIDR」** : `0.0.0.0/0` と入力
    + **「IP プロトコル」** : **「TCP」** を選択 （デフォルト値）
    + **「宛先ポート範囲」** : `443` と入力

    入力し終わったら、 **「イングレス・ルールの追加」** ボタンをクリックします。

## Oracle Functions アプリケーションとファンクションのデプロイ

このチュートリアルでは、『[Oracle Functions ことはじめ][paasdocs]』の手順にしたがって Oracle Functions のアプリケーションとファンクションをデプロイしてください。
Oracle Functions のアプリケーションは、[仮想クラウド・ネットワーク (VCN) の作成][create_vcn] で作成した VCN のプライベート・サブネットを使用します。

[paasdocs]: https://oracle-japan.github.io/paasdocs/documents/faas/oraclefunctions/handson/getting-started/ "PaaS Docs - Oracle Functions ことはじめ"

## API ゲートウェイから Oracle Functions のリソースへのアクセス

今回のチュートリアルのように API のバックエンド・サービスとして Oracle Functions のアプリケーションを使用する場合は、API ゲートウェイが Oracle Functions 関連のリソースを使用することを許可する IAM ポリシーを定義する必要があります。

```text
Allow any-user to use functions-family in compartment ${使用する Oracle Functions のコンパートメント名}
where
all {
  request.principal.type= 'ApiGateway',
  request.resource.compartment.id = ${API ゲートウェイを配置するコンパートメントの OCID}
}
```

> ***注意:***
> 上のサンプルは、ポリシーの定義内容を理解しやすくするためにステートメントに改行を追加していますが、 OCI コンソールでポリシーを定義する際にはステートメントに改行を含めることはできません。
> ポリシーを定義する際には、適宜読み替えてください。

## API ゲートウェイのインスタンスの作成

1.  OCI コンソールにログインします。
    OCI コンソールの画面の左上にあるハンバーガー・アイコンをクリックし、 **「ソリューションおよびプラットフォーム」** にある **「開発者メニュー」** → **「API ゲートウェイ」** を選択します。

1.  API ゲートウェイのページが表示されます。
    画面左側の **「リスト範囲」** メニューの **「コンパートメント」** リストからゲートウェイを作成するコンパートメントを選択します。

    **「ゲートウェイの作成」** ボタンをクリックします。

1.  **「ゲートウェイの作成」** が表示されます。
    次の項目を指定します。

    + **「名前」** :
      ゲートウェイに付ける名前を入力します。
      このチュートリアルでは、 `tutorial-gw` と入力することにします。
    + **「タイプ」** :
      作成するゲートウェイのタイプを選択します。
      このチュートリアルではインターネット経由でのリクエストを受け付けるために、 **「パブリック」** を選択します。
    + **「コンパートメント」** :
      ゲートウェイを配置するコンパートメントを選択します。
    + **「仮想クラウド・ネットワーク」** :
      ゲートウェイが使用する仮想クラウド・ネットワーク (VCN) を選択します。
      このチュートリアルの『[仮想クラウド・ネットワーク (VCN) の作成][create_vcn]』で作成した VCN を選択してください。

      > ***備考***:
      > ゲートウェイと VCN は異なるコンパートメントに配置することも可能です。
      > ゲートウェイと VCN のコンパートメントが異なる場合は、 **「コンパートメントの変更」** をクリックして VCN が配置されているコンパートメントを選択する必要があります。

    + **「サブネット」** :
      ゲートウェイが使用するリージョナル・サブネットを選択します。
      このチュートリアルではゲートウェイが使用するパブリック・リージョナル・サブネットを選択します。

    すべての項目を指定したら、 **「作成」** ボタンをクリックします。

1.  **「ゲートウェイの詳細」** ページが表示されます。

    ゲートウェイの作成が完了すると、ステータスがアクティブになります。

> ***Note:***
> テナント内に作成できるゲートウェイの数には上限が設定されています。
> 次のようなメッセージが表示された場合は、ゲートウェイの数がすでに上限に達していることを表しています。
>
>> The following service limits were exceeded: gateway-count.
>> Request a service limit increase from the service limits page in the console.
>
> ゲートウェイの数が上限に達している場合は、上限の引き上げをリクエストできます。
> 詳細は、 API ゲートウェイのドキュメントのサービス制限に関するページ （[英語版][servicelimits_en]/[日本語機械翻訳版][servicelimits_ja]） を参照してください。

[servicelimits_en]: https://docs.cloud.oracle.com/en-us/iaas/Content/General/Concepts/servicelimits.htm "API Gateway - Service Limit"
[servicelimits_ja]: https://docs.cloud.oracle.com/ja-jp/iaas/Content/General/Concepts/servicelimits.htm "API ゲートウェイ - サービス制限"

## API のデプロイ

1.  作成したゲートウェイの詳細ページを開きます。
    画面の左側にある **「リソース」** メニューから **「デプロイメント」** を選択します。

    **「デプロイメントの作成」** ボタンをクリックします。

1.  **「基本情報」** を指定します。
    **「最初から」** が選択されていることを確認して、次の項目を入力します。

    + **「名前」** :
      デプロイする API に付ける名前を入力します。
      このチュートリアルでは、 `helloworld-api` と入力します。
    + **「パス接頭辞」** :
      デプロイする API の URL のパスを指定します。
      このチュートリアルでは、 `helloworld` と入力します。
    + **「コンパートメント」** :
      デプロイする API が属するコンパートメントを選択します。
      このチュートリアルでは、初期状態で選択されているゲートウェイが属しているコンパートメントを使用します。

1.  **「API リクエスト・ポリシー」** では、APIを実行するための認証や [CORS (Cross-Origin Resource Sharing)][CORS_MSDN]、リクエスト数の制限などの設定が可能です。
    このチュートリアルでは、何も設定しません。

1.  **「API ロギング・ポリシー」** では、アクセス・ログと実行ログを設定できます。
    このチュートリアルでは、 **「アクセス・ログ」** と **「実行ログ」** を有効にします。
    また、実行ログの **「ログ・レベル」** として **「情報」** を選択します。

    画面の左下にある **「次」** ボタンをクリックします。

1.  **「ルート」** の情報を入力します。
    ルートとは、ゲートウェイが受信したリクエストをバックエンド・サービスにルーティングするための設定です。
    1つの API に複数のルートを定義することができます。
    このチュートリアルでは、1つのルートを定義します。
    **「1のルーティング」** に次の項目を入力します。

    + **「パス」** :
      このルートで使用される URL のパスを指定します。
      このチュートリアルでは `hello` と入力します。
    + **「メソッド」** :
      対応する HTTP メソッドを選択します。
      このチュートリアルでは、**「GET」** と **「POST」** の２つを選択します。
    + **「タイプ」** :
      バックエンド・サービスの種類を選択します。
      このチュートリアルでは、 **「Oracle Functions」** を選択します。
    + **「アプリケーション」** :
      バックエンド・サービスとなる Oracle Functions アプリケーションを選択します。
      『[Oracle Functions ことはじめ][paasdocs]』の手順にしたがってファンクションがデプロイされている場合は、 **「helloworld-app」** を選択してください。

      > ***Note***:
      > API とファンクションは異なるコンパートメントに配置することも可能です。
      > API とファンクションのコンパートメントが異なる場合は、 **「コンパートメントの変更」** をクリックしてファンクションが配置されているコンパートメントを選択する必要があります。

    + **「機能名」** :
      起動するファンクションを選択します。
      『[Oracle Functions ことはじめ][paasdocs]』の手順にしたがってファンクションがデプロイされている場合は、 **「helloworld-func」** を選択してください。

    画面の下にある **「次」** ボタンをクリックします。

1.  **「確認」** 画面が表示され、ここまでに入力した内容が表示されます。
    **「保存」** ボタンをクリックします。

[CORS_MSDN]: https://developer.mozilla.org/ja/docs/Web/HTTP/CORS "オリジン間リソース共有 (CORS) - HTTP | MDN"

## API のテスト

デプロイした API をテストしてみましょう。
[cURL][curl] や [Postman][postman] など REST API を実行するツールを使用してリクエストを送信できます。

[curl]:     https://curl.haxx.se/ ":// curl"
[postman]:  https://www.postman.com/ "Postman | The Collaboration Platform for API Development"

API ゲートウェイにデプロイした API のエンドポイントは、次のようになります。

```text
https://<ゲートウェイのホスト名>/<デプロイメントのパス接頭辞>/<ルートのパス>
```

このチュートリアルの手順のとおりにデプロイメントとルートを設定した場合、デプロイメントのパス接頭辞は `helloworld-api`、ルートのパスは `hello` に設定されています。

ゲートウェイのホスト名は、 OCI コンソールのゲートウェイの詳細ページで確認することができます。

例えば、cURL を使用して GET リクエストを送信するには、次のように実行します。

```sh
curl -is -X GET https://<ゲートウェイのホスト名>/helloworld-api/hello
```

バックエンド・サービスが『[Oracle Functions ことはじめ][paasdocs]』で作成したファンクションの場合は、POST リクエストでパラメータを送信できます。

```sh
curl -is -X POST \
  -H "Content-Type: text/plain" \
  -d "Candy" \
  https://<ゲートウェイのホスト名>/helloworld-api/hello
```

## ログ・ファイルのダウンロード

デプロイメントの基本情報でロギングを有効にすると、ログ・ファイルは OCI オブジェクト・ストレージのストレージ・バケットに格納されます。
ログ・ファイルは次の手順でダウンロードできます。

1.  OCI コンソールにログインします。
    画面左上のハンバーガー・アイコンをクリックし、 **「コア・インフラストラクチャ」** にある **「オブジェクト・ストレージ」** → **「オブジェクト・ストレージ」** を選択します。

    **「オブジェクト・ストレージ」** ページが表示されます。

1.  画面左側の **コンパートメント** で、ゲートウェイを作成したコンパートメントを選択します。
    選択したコンパートメント内のバケットのリストが表示されます。

1.  名前が `oci-logs._apigateway...` で始まるバケットをクリックします。
    バケットに保存されているオブジェクトのリストが表示されます。
    ログ・ファイルは次の名前で保存されます。

    + `api_gateway_deployment_accesslog/${デプロイメントの OCID}/${タイムスタンプ}.log.gz` : アクセス・ログ
    + `api_gateway_deployment_executionlog/${デプロイメントの OCID}/${タイムスタンプ}.log.gz` : 実行ログ

    メニュー・アイコンをクリックし、 **「ダウンロード」** をクリックすることでファイルをダウンロードできます。

    ログ・ファイルは gzip 形式で圧縮されています。
    Windows 環境の場合は、gzip 形式に対応した解凍ツール ([7-Zip][7-zip] など) をインストールする必要があります。

[7-zip]:  https://sevenzip.osdn.jp/ "圧縮・解凍ソフト 7-Zip"
