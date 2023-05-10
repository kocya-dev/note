* Cognito
  * [Cognito + API Gateway + Lambda で実行権限を動的に制御したい](https://blog.serverworks.co.jp/2021/12/22/125040)
  * [Cognitoで認証されたユーザーだけがAPI Gatewayを呼び出せるオーソライザーを使ってみた](https://dev.classmethod.jp/articles/api-gateway-cognito-authorizer/)
  * [Amazon CognitoとCloudFrontで特定のユーザのみが閲覧できる仕組みを作る
  ](https://tech.macloud.jp/entry/2022/05/13/144430)
  * [Lambda@Edgeを使用したCloudFrontのCognito認証を試してみた](https://dev.classmethod.jp/articles/cloud-front-cognito-auth/)
  
* CDK
  * [AWS CDKとGitHub ActionsでLambdaで動くAPIをTypeScriptで作る](https://tech.smartcamp.co.jp/entry/gh-action-and-cdk)
  * [AWS CDK V2でCloudFront+S3の静的サイトホスティングを構築してみた](https://dev.classmethod.jp/articles/i-tried-building-cloudfronts3-static-site-hosting-with-aws-cdk-v2/)
  * [AWS CDKで複数の環境を設定する際の設定値(config)の渡し方](https://dev.classmethod.jp/articles/aws-cdk-multi-environment-config/)
  * [AWS Dev Day Online Japan C-2: AWS CDKはどう使いこなすのか、初期開発から運用までのノウハウ](https://www.youtube.com/watch?v=xLrCEHEYcCM)
  * [AWS CDK Workshop](https://cdkworkshop.com/ja/)
  * [CDKインポートの実力はどうなのか？　〜我々調査隊はアマゾンの奥地へと向かった〜](https://tech.nri-net.com/entry/how_about_cdk_import)
  * [CDKを使って既存S3バケットのPUTイベントをトリガーにLambda関数を実行しようとしたらハマった話](https://dev.classmethod.jp/articles/cdk-s3notification-kick-lambda/)
  * CloudFront
    * Lambda@Edge (この方法はアクセス毎にlambdaが実行され、認可処理が行われるのでイマイチ)
      * [[AWS CDK] Lambda@Edge関数とCloudFront DistributionのConstructは別スタックに分ければ、関数削除時のデプロイが1度で済むのか試してみた](https://dev.classmethod.jp/articles/it-seems-better-to-divide-lambda-edgecloudfront-distribution-resources-into-separate-stacks/)
      * [[AWS CDK] Lambda@Edge関数の作成または更新時に自動的に最新バージョンを発行し、CloudFront Distributionに紐付ける](https://dev.classmethod.jp/articles/aws-cdk-i-tried-linking-lambda-edge-to-cloudfront-distribution/)
      * [[AWS CDK] 同じApp Construct内で異なるリージョンのStackをデプロイできるのか試してみた](https://dev.classmethod.jp/articles/aws-cdk-to-see-if-stacks-in-different-regions-can-be-deployed-in-the-same-app-construct/)
      * [AWS CDKで別リージョンに基本認証用Lambda@Edgeを作成するスタックをデプロイしてAmazon CloudFrontに設定する](https://tech.nri-net.com/entry/aws_cdk_cross_region_stack_deployment_lambda_edge)
      * [AWS CDKで別リージョンにスタックをデプロイしてパラメータをリージョン間で受け渡す方法 －AWS CDKカスタムリソースの実装例](https://tech.nri-net.com/entry/aws_cdk_cross_region_stack_deployment_method)
    * 署名付きCookie関連 
      * [CloudFront+S3に対して自作システムにログインしないと配信・アクセスできないようにする。](https://www.pnkts.net/2018/06/11/s3-signed-cookies/)
        * Lambda@Edgeを使わずcookieで制御する
      * [CloudFrontの署名付きCookieでプライベートコンテンツの配信](https://dev.classmethod.jp/articles/cloudfront-signed-cookie/)
      * [署名付き Cookie を使用して HLS コンテンツを取得してみた](https://dev.classmethod.jp/articles/get-hls-using-signedcookie/)
      * [ふらっとAWS CloudFrontの署名付きCookieを使って認証画面を作った話](https://qiita.com/Kodak_tmo/items/ff656c4b18b59849c011)
      * [AWS CDKでリソース作成 第9回: CloudFront編](https://tech.excite.co.jp/entry/2022/12/06/125603)
    * キャッシュ対策
      * [S3にCloudFrontを通すことで月20万ぐらい節約した話](https://katsusand.dev/posts/aws-s3-to-cloudfront/)
  * CI/CD高速化
    * [AWS CDKによるデプロイ作業を高速にするには？](https://kakehashi-dev.hatenablog.com/entry/2021/10/05/080018)
    * [AWS CDKにLambda関数を数秒でデプロイするhotswap deployments機能が追加されました](https://zenn.dev/intercept6/articles/eed5b5cef89eb2)  
      開発時のみ。本番環境では非推奨。
    * [CDKで複数スタックの並行デプロイを行ってみた](https://dev.classmethod.jp/articles/cdk-concurrency-deployment/)
      * `--concurrency 3` のように`cdk deploy`コマンドのオプション指定を行なう
    * [JAWS-UG CDK支部#4で「concurrencyで爆速並列デプロイ」というタイトルでLTしました](https://qiita.com/hedgehog051/items/5caa438a4c1802442632)
* S3
  * [LambdaがS3からGetObjectするのにかかる時間を計測してみた [追記, 修正あり]](https://dev.classmethod.jp/articles/mesure-download-time-of-get-object-from-s3-on-lambda/)

* 他
  * [Cognito+API Gateway+S3ホスティングでサーバレスアプリを構築する | AWSチュートリアル](https://www.bioerrorlog.work/entry/aws-serverless-tutorial)
  * [マルチアカウント環境のリソースを可視化。「Workload Discovery on AWS」を試してみる](https://aws.amazon.com/jp/builders-flash/202209/workload-discovery-on-aws/?awsf.filter-name=*all)
