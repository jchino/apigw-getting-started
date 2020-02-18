# API ゲートウェイ

チュートリアルで作成するアプリケーションの構成

## API ゲートウェイを使用するために必要な設定

### 仮想クラウド・ネットワークの設定

### API ゲートウェイ・インスタンスから Oracle Functions のリソースへのアクセス

```text
Allow any-user to use functions-family in compartment ${Functions compartment name}
where
all {
  request.principal.type= 'ApiGateway',
  request.resource.compartment.id = ${API gateway's compartment id}
}
```

## ゲートウェイの作成

1.  OCI コンソールにログインします。
    OCI コンソールの画面の左上にあるハンバーガー・メニューをクリックし、 **「開発者メニュー」** → **「API ゲートウェイ」** を選択します。

1.  API ゲートウェイのページが表示されます。
    **「ゲートウェイの作成」** ボタンをクリックします。

1.  **「ゲートウェイの作成」** が表示されます。
    次の項目を指定します。

    - **「名前」** :
    - **「タイプ」** :
      作成するゲートウェイのタイプを選択します。
      このチュートリアルでは、**「パブリック」** を選択します。
    - **「コンパートメント」** :
      ゲートウェイを配置するコンパートメントを選択します。
    - **「仮想クラウド・ネットワーク」** :
    - **「サブネット」** :

    すべての項目を指定したら、 **「作成」** ボタンをクリックします。

1.  **「ゲートウェイの詳細」** ページが表示されます。

    ゲートウェイの作成が完了すると、ステータスがアクティブになります。

## デプロイメントの作成

1.  画面の左にある **「リソース」** メニューで **「デプロイメント」** を選択します。

    **「デプロイメントの作成」** ボタンをクリックします。

1.  **「基本情報」** を指定します。
    **「最初から」** が選択されていることを確認して、次の項目を入力します。

    - **「名前」** :
    - **「パス接頭辞」** :
    - **「コンパートメント」** :

    画面の左下にある **「次」** ボタンをクリックします。

1.  **「ルート」** の情報を入力します。

    - **「パス」** :
    - **「メソッド」** :
    - **「タイプ」** :
    - **「アプリケーション」** :
    - **「機能名」** :

    画面の下にある **「次」** ボタンをクリックします。

1.  **「確認」** 画面が表示され、ここまでに入力した内容が表示されます。
    **「保存」** ボタンをクリックします。

## アプリケーションのテスト

デプロイメントが作成されると、 **「デプロイメントの詳細」** ページに、デプロイメントのエンドポイントが表示されます。

<!-- Document Reference -->
[paasdocs_oraclefunctions]: https://oracle-japan.github.io/paasdocs/documents/faas/oraclefunctions/handson/getting-started/ "Oracle Functions ことはじめ"
