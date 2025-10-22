## Message

- AWS Step Functions

  - サーバーレスアプリケーションのワークフローを簡素化
  - 複数の AWS サービスを統合し、状態管理を提供

- Amazon SNS

  - フルマネージドなメッセージ通知サービス
  - アプリケーション間のメッセージ配信を簡素化
  - SMS、E メール、モバイルプッシュ通知などの配信方法をサポート

- Amazon SES

  - フルマネージドなメール送信サービス
  - 大量のメールを迅速に送信可能
  - バウンスや苦情の管理をサポート

  ## SendTemplatedEmail API

  ### 概要

  - テンプレートを使用してメールを作成し、即座にキューに入れて送信
  - 既存のメールテンプレートを参照して使用（CreateTemplate で事前作成が必要）
  - パーソナライズされたメール送信が可能

  ### 使用要件

  - 既存のメールテンプレートが必要（CreateTemplate で作成）
  - 送信元は検証済みのメールアドレスまたはドメインである必要がある
  - サンドボックス環境では検証済みアドレスまたはテストアドレスのみに送信可能
  - 最大メッセージサイズは 10MB
  - 1 回の呼び出しで 1 つの Destination パラメータのみ指定可能
  - Destination には最大 50 件の受信者（To、CC、BCC 合計）を含めることができる

  ### 主要パラメータ

  **必須パラメータ**

  - **Source**: 送信者のメールアドレス（検証済みである必要がある）
  - **Destination**: 受信者情報（To、CC、BCC フィールドを含む）
  - **Template**: 使用するテンプレート名
  - **TemplateData**: テンプレート内の置換値（JSON 形式、最大 262,144 文字）

  **オプションパラメータ**

  - ConfigurationSetName: 設定セット名
  - ReplyToAddresses: 返信先アドレス
  - ReturnPath: バウンス・苦情の転送先アドレス
  - ReturnPathArn: 送信認可用の ReturnPath の ARN
  - SourceArn: 送信認可用の Source の ARN
  - Tags: メールに適用するタグ（名前/値のペア）
  - TemplateArn: テンプレートの ARN

  ### レスポンス

  - **MessageId**: 送信されたメールの一意識別子

  ### エラー処理

  - AccountSendingPaused: アカウントのメール送信が無効
  - ConfigurationSetDoesNotExist: 設定セットが存在しない
  - ConfigurationSetSendingPaused: 設定セットのメール送信が無効
  - MailFromDomainNotVerified: MAIL FROM ドメインが未検証
  - MessageRejected: メッセージが拒否された
  - TemplateDoesNotExist: テンプレートが存在しない

  ### テンプレートについて

  **テンプレートの構成要素**

  - Subject: 件名（置換タグ使用可能）
  - HtmlPart: HTML メール本文（置換タグ使用可能）
  - TextPart: テキストメール本文（置換タグ使用可能）
  - 置換タグの形式: `{{tagname}}`

  **テンプレートの例**

  ```json
  {
    "TemplateName": "MyTemplate",
    "TemplateContent": {
      "Subject": "Greetings, {{name}}!",
      "Text": "Dear {{name}},\r\nYour favorite animal is {{favoriteanimal}}.",
      "Html": "<h1>Hello {{name}},</h1><p>Your favorite animal is {{favoriteanimal}}.</p>"
    }
  }
  ```

  ### 制限事項

  - ストアドテンプレート: 各リージョンで最大 20,000 テンプレート
  - 各テンプレートのサイズ: 最大 500KB（HTML とテキスト部分含む）
  - 置換変数の数に制限なし
  - SendBulkEmail では最大 50 の Destination オブジェクト

  ### ベストプラクティス

  - **レンダリング失敗通知の設定を強く推奨**
    - テンプレートにエラーがある場合、メールは受け入れられるが配信されない
    - Amazon SNS を使用してレンダリング失敗イベントの通知を設定
  - テンプレートデータの JSON 形式が正しいことを確認
  - 置換タグとテンプレートデータのキーが一致することを確認

  ### 参考 URL

  - API Reference: https://docs.aws.amazon.com/ses/latest/APIReference/API_SendTemplatedEmail.html
  - テンプレート使用方法: https://docs.aws.amazon.com/ses/latest/dg/send-personalized-email-api.html
  - コード例: https://docs.aws.amazon.com/ses/latest/dg/ses_example_ses_SendTemplatedEmail_section.html

- Amazon EventBridge

  - フルマネージドなイベントバスサービス
  - アプリケーション間のイベント駆動型アーキテクチャを簡素化
  - AWS の他のサービスや SaaS アプリケーションとの統合が容易

- Amazon SQS
  - フルマネージドなメッセージキューサービス
  - アプリケーション間の非同期通信をサポート
  - メッセージの送受信、キューの管理を簡素化
  - FIFO キューと標準キューの選択肢
  - メッセージの順序保証や重複排除が可能
  - デッドレターキューを使用して、処理に失敗したメッセージを管理
