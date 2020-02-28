# OCI API ゲートウェイを使用したサーバーレス・アプリケーション

## このチュートリアルで作成するアプリケーション

![このチュートリアルで作成するアプリケーション][overview]

[overview]: images/overview.png "このチュートリアルで作成するアプリケーション"

### コンパートメント

コンパートメントについて。
論理的なグループがなんたらかんたら

### 仮想クラウド・ネットワーク (VCN)

仮想クラウド・ネットワークについて。API ゲートウェイはリージョナル・サブネットに配置しなければならない。QuickStart を使うと簡単
インターネットからのアクセスできるようにするためにパブリック・サブネットを作成し、HTTPS (TCP/443) へのリクエストをとおす

### Oracle Functions アプリケーションとファンクション

『[Oracle Functions ことはじめ][paasdocs]』参照のこと

[paasdocs]: https://oracle-japan.github.io/paasdocs/documents/faas/oraclefunctions/handson/getting-started/ "PaaS Docs - Oracle Functions ことはじめ"

## IAM ポリシーの設定

OCI では、Identity and Access Management （IAM） と呼ばれる仕組みによって各種リソースへのアクセスを制御します。
トライアル環境の場合は、トライアルに申し込んだユーザーは自動的に `Administrators` グループに所属しており、テナント内のすべてのリソースにアクセスできるポリシーが定義済みです。

- API ゲートウェイ関連リソースの管理操作を許可するポリシー
- ネットワーク関連リソースの使用を許可するポリシー
- Oracle Functions 関連リソースの使用を許可するポリシー

### API ゲートウェイ関連リソースの管理操作を許可するポリシーの作成

API ゲートウェイのインスタンスを作成したり、作成したインスタンスを管理したりするためには、管理操作 (manage) を許可するポリシーを定義します。

```sh
Allow group ${API ゲートウェイの管理者グループの名前} to manage api-gateway-family in compartment ${APIゲートウェイのコンパートメントの名前}
```

例えば API ゲートウェイの管理者グループの名前が `apigw-managers`、API ゲートウェイのコンパートメントの名前が `apigw-compartment` の場合、ポリシーのステートメントは次のとおりです。

```sh
Allow group apigw-managers to manage api-gateway-family in compartment apigw-compartment
```

### ネットワーク関連リソースの使用を許可するポリシーの作成

```sh
Allow group ${API ゲートウェイの管理者グループの名前} to use network-family in compartment ${VNC のコンパートメントの名前}
```

例えば、API ゲートウェイの管理者グループの名前が `apigw-managers`、API ゲートウェイのリソースがアクセスする VNC のコンパートメントの名前が `apigw-compartment` の場合、IAM ポリシーのステートメントは次のとおりです。

```sh
Allow group apigw-managers to use network-family in compartment apigw-compartment
```

### Oracle Functions 関連リソースの使用を許可するポリシーの作成

```sh
Allow group ${APIゲートウェイの管理者グループの名前} to use functions-family in compartment ${Oracle Functions コンパートメントの名前}
```

例えば、API ゲートウェイの管理者グループの名前が `apigw-managers`、API ゲートウェイのリソースがアクセスする Oracle Functions のコンパートメントの名前が `apigw-compartment` の場合、IAM ポリシーのステートメントは次のとおりです。

```sh
Allow group apigw-managers to use functions-family in compartment apigw-compartment
```

## API ゲートウェイ・インスタンスから Oracle Functions のリソースへのアクセス

今回のチュートリアルのように API のバックエンド・サービスとして Oracle Functions を使用する場合は、API ゲートウェイが Oracle Functions 関連のリソースを使用することを許可する IAM ポリシーを定義する必要があります。

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

## ゲートウェイの作成

1.  OCI コンソールにログインします。
    OCI コンソールの画面の左上にあるハンバーガー・アイコンをクリックし、 **「ソリューションおよびプラットフォーム」** にある **「開発者メニュー」** → **「API ゲートウェイ」** を選択します。

1.  API ゲートウェイのページが表示されます。
    画面左側の **「リスト範囲」** メニューの **「コンパートメント」** リストからゲートウェイを作成するコンパートメントを選択します。

    **「ゲートウェイの作成」** ボタンをクリックします。

1.  **「ゲートウェイの作成」** が表示されます。
    次の項目を指定します。

    + **「名前」** :
      ゲートウェイに付ける名前を入力します。
      このチュートリアルでは、 `tutorial-apigateway` と入力することにします。
    + **「タイプ」** :
      作成するゲートウェイのタイプを選択します。
      このチュートリアルではインターネット経由でのリクエストを受け付けるために、 **「パブリック」** を選択します。
    + **「コンパートメント」** :
      ゲートウェイを配置するコンパートメントを選択します。
    + **「仮想クラウド・ネットワーク」** :
      ゲートウェイが使用する仮想クラウド・ネットワーク (VCN) を選択します。
      ゲートウェイを配置するコンパートメントと VCN が配置されているコンパートメントが異なる場合は、 **「コンパートメントの変更」** をクリックして VCN が配置されているコンパートメントを選択する必要があります。
    + **「サブネット」** :
      ゲートウェイが使用するリージョナル・サブネットを選択します。
      このチュートリアルではゲートウェイが使用するパブリック・リージョナル・サブネットを選択します。

    すべての項目を指定したら、 **「作成」** ボタンをクリックします。

1.  **「ゲートウェイの詳細」** ページが表示されます。

    ゲートウェイの作成が完了すると、ステータスがアクティブになります。

> ***注意:***
> テナント内に作成できるゲートウェイの数には上限が設定されており、初期状態では10に設定されています。
> 次のようなメッセージが表示された場合は、ゲートウェイの数がすでに上限に達していることを表しています。
>
>> The following service limits were exceeded: gateway-count.
>> Request a service limit increase from the service limits page in the console.
>
> ゲートウェイの数が上限に達している場合は、上限の引き上げをリクエストできます。
> 詳細は、API ゲートウェイのドキュメントのサービス制限に関するページ（[英語版][servicelimits_en]/[日本語機械翻訳版][servicelimits_ja]）を参照してください。

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
    + **「パス接頭辞」** :
      デプロイする API の URL のパスを指定します。
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
    ルートとは、ゲートウェイが受信したリクエストをバックエンド・サービスに転送するための設定です。
    1つの API に複数のルートを定義することができます。
    このチュートリアルでは、1つのルートを定義します。
    **「1のルーティング」** に次の項目を入力します。

    + **「パス」** :
    + **「メソッド」** :
      対応する HTTP メソッドを選択します。
      このチュートリアルでは、**「GET」** と **「POST」** の２つを選択します。
    + **「タイプ」** :
      バックエンド・サービスの種類を選択します。
      このチュートリアルでは、 **「Oracle Functions」** を選択します。
    + **「アプリケーション」** :
      バックエンド・サービスとなる Oracle Functions アプリケーションを選択します。
      もし、デプロイする API を Oracle Functions アプリケーションとは異なるコンパートメントに配置する場合は、 **「コンパートメントの変更」** をクリックして適切なコンパートメントを選択する必要があります。
    + **「機能名」** :
      起動されるファンクションを選択します。

    画面の下にある **「次」** ボタンをクリックします。

1.  **「確認」** 画面が表示され、ここまでに入力した内容が表示されます。
    **「保存」** ボタンをクリックします。

[CORS_MSDN]: https://developer.mozilla.org/ja/docs/Web/HTTP/CORS "オリジン間リソース共有 (CORS) - HTTP | MDN"

## API のテスト

```sh
curl -is -X GET https://<ゲートウェイのホスト名>/<デプロイメントのパス接頭辞>/<ルートのパス>
```

```sh
curl -is -X POST \
  -H "Content-Type: text/plain" \
  -d "Candy" \
  https://<ゲートウェイのホスト名>/<デプロイメントのパス接頭辞>/<ルートのパス>
```

## ログの確認

デプロイメントの基本情報でロギングを有効にすると、ログは OCI オブジェクト・ストレージのストレージ・バケットに格納されます。

1.  OCI コンソールにログインしていない場合は、ログインします。
    画面左上のハンバーガー・アイコンをクリックし、 **「コア・インフラストラクチャ」** にある **「オブジェクト・ストレージ」** → **「オブジェクト・ストレージ」** を選択します。

    **「オブジェクト・ストレージ」** ページが表示されます。

1.  画面左側の **コンパートメント** で、ゲートウェイを作成したコンパートメントを選択します。
    選択したコンパートメント内のバケットのリストが表示されます。

1.  名前が `oci-logs._apigateway...` で始まるバケットをクリックします。
    格納されているログ・ファイルのリストが表示されます。
    + `api_gateway_deployment_accesslog/${デプロイメントの OCID}/${タイムスタンプ}.log.gz` : アクセス・ログ
    + `api_gateway_deployment_executionlog/${デプロイメントの OCID}/${タイムスタンプ}.log.gz` : 実行ログ
