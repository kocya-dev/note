# Cognito

## 概念

### ユーザープール

ユーザーが誰か？を認証するためのプール。

### IDプール

認証されたユーザーに何を許可するか、を認可するためのプール。  
未認可/認可済みのロールが作成されるので、IAMで追加ロールにポリシーをアタッチする。

## 設定

* コールバックURL
  * サインイン後にリダイレクトされるURL
* サインアウトURL
  * サインアウト後にリダイレクトされるURL

## ホステッドUI

[AmplifyでCognitoのHosted UIを利用した認証を最低限の実装で動かしてみて動作を理解する](https://dev.classmethod.jp/articles/learn-authentication-using-cognitos-hosted-ui-with-amplify/)

ホステッドUIのURLは下記ルールに従う  
ログイン画面
https://<your_domain>/login?client_id=<your_app_client_id>&response_type=code&scope=<scope>&redirect_uri=<your_callback_url>
サインアップ  
/signup  

MFA 毎回コード入力要求される

## 任意のドメインで認証する場合の手順

### 準備(ドメイン名取得(有料))

1. `Route53`でホストゾーンを選択
2. `レコード`タブで`レコード作成`
3. `レコード名`にauthを設定 (cognito推奨)
4. `レコードタイプ`は`A`  
    [Aレコードに関する参考情報](https://www.value-domain.com/media/a-record/)

auth.hogehoge.com のようなドメインになる。

### 設定

1. ユーザープールの`カスタムドメインを作成`から`アプリケーションの統合`
2. Cognitoドメインを削除
3. `カスタムドメインを作成`
4. ドメイン名を入力、ACM証明書を作成する
