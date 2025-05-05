kocya
  土曜日 16:24
@Devin

https://github.com/kocya-dev-org/zenn-ranking/blob/main/cdk/lib/cdk-stack.ts
を見て意見が欲しいです。
Cloudfront経由でAPI Gatewayに対してAPIを呼び出した時に同じパラメータであればキャッシュが有効になるようにしたいです。
現状はキャッシュが効いていますか？
また、有効にするためにどんな編集が必要か提案をください。

リアクションする

16 件の返信

Devin
アプリ  土曜日 16:24
Devin is running, you can send messages in this thread! Messages starting with 'aside' will be ignored. (open webapp)
16:25
CloudFrontを通じたAPIキャッシュの状況と改善提案
現状では、CloudFront経由でAPI Gatewayに対してAPIを呼び出した場合、キャッシュは*無効*になっています。
CDKスタックのコードを確認すると、CloudFrontディストリビューションの設定で、API Gateway向けの追加ビヘイビアに対して明示的にキャッシュが無効化されています：
typescript
additionalBehaviors: {
  [`/${API_PATH}/*`]: {
    origin: new origins.RestApiOrigin(api),
    viewerProtocolPolicy: cloudfront.ViewerProtocolPolicy.REDIRECT_TO_HTTPS,
    allowedMethods: cloudfront.AllowedMethods.ALLOW_ALL,
    cachePolicy: cloudfront.CachePolicy.CACHING_DISABLED,
  },
},
kocya-dev-org/zenn-ranking/cdk/lib/cdk-stack.ts:146-153
アーキテクチャドキュメントでは、APIのキャッシュについて以下のように記述されています：
#### API Gateway

キャッシュ有効、かつ期間が1日。
そのため日/週/月単位指定それぞれの呼び出しがあるとして、1日に3回リクエストが発行される見込み。
kocya-dev-org/zenn-ranking/doc/architecture.md:80-83
しかし、実際のコード実装ではキャッシュが無効化されているため、アーキテクチャドキュメントと実装に不一致があります。
キャッシュを有効にするための編集提案
CloudFrontでAPIリクエストのキャッシュを有効にするには、以下の変更が必要です：
1. カスタムキャッシュポリシーを作成する
2. API Pathの追加ビヘイビアにそのキャッシュポリシーを適用する
具体的なコード変更案：
typescript
// カスタムキャッシュポリシーを作成
const apiCachePolicy = new cloudfront.CachePolicy(this, `${PREFIX}-api-cache-policy`, {
  cachePolicyName: `${PREFIX}-api-cache-policy`,
  comment: 'Cache policy for API requests',
  defaultTtl: cdk.Duration.days(1),
  minTtl: cdk.Duration.minutes(5),
  maxTtl: cdk.Duration.days(1),
  enableAcceptEncodingGzip: true,
  enableAcceptEncodingBrotli: true,
  queryStringBehavior: cloudfront.CacheQueryStringBehavior.allowList(
    // キャッシュキーに含めるクエリパラメータを指定
    'date' // 例: dateパラメータでキャッシュを分ける
  ),
});

// CloudFrontディストリビューションの設定でキャッシュポリシーを適用
additionalBehaviors: {
  [`/${API_PATH}/*`]: {
    origin: new origins.RestApiOrigin(api),
    viewerProtocolPolicy: cloudfront.ViewerProtocolPolicy.REDIRECT_TO_HTTPS,
    allowedMethods: cloudfront.AllowedMethods.ALLOW_ALL,
    cachePolicy: apiCachePolicy, // カスタムキャッシュポリシーを適用
  },
},
この変更により、同じパラメータ（例：同じdateクエリパラメータ）でAPIを呼び出した場合、CloudFrontがレスポンスをキャッシュし、指定した期間（デフォルトで1日）はAPI Gatewayやバックエンドのリソースを使用せずにレスポンスを返すことができます。
注意点
1. キャッシュキーに含めるクエリパラメータは、実際のAPIで使用しているパラメータに合わせて調整してください。
2. TTL（Time To Live）の値は、データの更新頻度に合わせて適切に設定してください。
3. キャッシュを有効にすると、データの更新がすぐに反映されなくなるため、データの鮮度とパフォーマンスのバランスを考慮する必要があります。
Notes
アーキテクチャドキュメントでは「キャッシュ有効、かつ期間が1日」と記述されていますが、実際のコードでは`CACHING_DISABLED`が設定されています。この不一致を解消するためにも、上記の変更が推奨されます。
また、バッチ処理は日本時間の午後4時（UTC 16:00）に実行されるように設定されているため、キャッシュのTTLをこの時間に合わせて調整することも検討できます。
さらに詳しく調査しましょうか？例えば、特定のAPIエンドポイントのキャッシュ戦略や、キャッシュ無効化の方法などについて詳しく知りたいですか？


kocya
  土曜日 16:29
ありがとうございます。
キャッシュキーにはcount, order, unit, rangeの4つのクエリパラメータを追加してください。
その上で、対応内容をissueに登録してください。

Devin
アプリ  土曜日 16:29
了解しました。count, order, unit, rangeの4つのクエリパラメータをキャッシュキーに追加し、対応内容をissueに登録します。
16:29
対応完了のお知らせ
CloudFrontでAPIキャッシュを有効化する内容をGitHubのissueとして登録しました：
https://github.com/kocya-dev-org/zenn-ranking/issues/28
対応内容のまとめ
1. 現状の分析：
   * 現在はCloudFrontのキャッシュが`CACHING_DISABLED`で明示的に無効化されています
   * アーキテクチャドキュメントでは「キャッシュ有効、かつ期間が1日」と記述されており、実装との不一致があります
2. 提案した変更：
   * カスタムキャッシュポリシーを作成し、TTLを1日に設定
   * キャッシュキーに指定された4つのクエリパラメータ（count, order, unit, range）を含める
   * API Pathの追加ビヘイビアにそのキャッシュポリシーを適用
この変更により、同じパラメータでAPIを呼び出した場合にCloudFrontがレスポンスをキャッシュし、バックエンドリソースの使用を削減できます。
何か他にご質問やご要望がありましたら、お知らせください。


kocya
  土曜日 16:30
記載内容の実装をお願いします

Devin
アプリ  土曜日 16:30
承知しました。提案した内容を実装します。CloudFrontでAPIキャッシュを有効化するための変更を行います。
16:31
Created PR https://github.com/kocya-dev-org/zenn-ranking/pull/30. I will monitor the PR for results and let you know once it's ready for review.
16:33
PR状況の報告
CloudFrontでAPIキャッシュを有効化する変更のPRを作成しました：
https://github.com/kocya-dev-org/zenn-ranking/pull/30
CI結果
* :チェックマーク_緑: CDKの検証: 成功
* :チェックマーク_緑: フロントエンドの検証: 成功
* :x: バックエンドの検証: 失敗
バックエンド検証の失敗について
バックエンドのテストで失敗が発生していますが、これは私たちのCloudFrontキャッシュの変更とは無関係です。
失敗の詳細:
AssertionError: expected [ { id: 1, …(3) }, …(2) ] to have a length of 4 but got 3
batch/test/batchHandler.test.ts:134:22
この失敗は`batchHandler.test.ts`の134行目で、配列の長さが期待値の4ではなく3であることが原因です。私たちの変更はCDKスタックのCloudFront設定のみに関するもので、バックエンドのコードには一切触れていません。
対応方針
1. CDKの変更自体は正しく、CDKの検証も成功しています
2. バックエンドのテスト失敗は既存の問題であり、私たちの変更とは無関係です
このPRをこのまま進めるか、バックエンドのテスト問題も修正すべきかご指示ください。


kocya
  土曜日 16:36
対応方針「1」でお願いします。バックエンドの問題はそのままにしておいてください。

Devin
アプリ  土曜日 16:36
承知しました。対応方針「1」で進めます。CDKの変更のみを適用し、バックエンドの問題はそのままにしておきます。PRはこのまま進めていきます。
16:36
PR完了のお知らせ
CloudFrontでAPIキャッシュを有効化する変更のPRが完了しました：
https://github.com/kocya-dev-org/zenn-ranking/pull/30
実装内容
1. カスタムキャッシュポリシーを作成
   * TTLを1日に設定
   * 最小TTLを5分に設定
   * 最大TTLを1日に設定
   * Gzip/Brotli圧縮を有効化
2. キャッシュキーに以下の4つのクエリパラメータを追加
   * count
   * order
   * unit
   * range
3. API Pathの追加ビヘイビアにカスタムキャッシュポリシーを適用
CI結果
* :チェックマーク_緑: CDKの検証: 成功
* :チェックマーク_緑: フロントエンドの検証: 成功
* :x: バックエンドの検証: 失敗 (ご指示通り、バックエンドの問題はそのままにしています)
PRはレビュー可能な状態です。ご確認いただき、マージするかどうかご判断ください。
また、Devinのセッションでも作業内容を確認いただけます：
https://app.devin.ai/sessions/da3959a724274963a222823ba2d99eb3


kocya
  土曜日 16:39
対応内容をマージしました。ありがとうございます。

Devin
アプリ  土曜日 16:39
PRがマージされたとのこと、ありがとうございます。CloudFrontのAPIキャッシュが正しく設定され、同じパラメータでの呼び出し時にキャッシュが有効になるようになりました。


kocya
  土曜日 16:41
SLEEP

Devin
アプリ  土曜日 16:42
Devin went to sleep.