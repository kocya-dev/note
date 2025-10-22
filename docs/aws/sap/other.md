## Application Integration

- AWS Elastic Beanstalk

  - アプリケーションのデプロイと管理を簡素化
  - Java、.NET、PHP、Node.js、Python などのプラットフォームをサポート
  - オートスケーリングやロードバランシングを自動的に設定

- AWS IoT Core

  - フルマネージドな IoT デバイス接続サービス
  - デバイスの接続、管理、セキュリティを簡素化
  - デバイスからのデータ収集、分析、アクションをサポート
  - AWS Lambda、Amazon S3、Amazon DynamoDB などとの統合が可能

## Other

- Amazon Textract

  - フルマネージドなテキスト抽出サービス
  - 画像や PDF ドキュメントからテキストを抽出
  - 構造化データとしての出力をサポート
  - 機械学習を活用した高精度なテキスト認識

- Amazon Comprehend

  - フルマネージドな自然言語処理サービス
  - テキストの感情分析、トピックモデル、キーフレーズ抽出をサポート
  - 機械学習を活用した高精度なテキスト分析

- AWS Secrets Manager

  - フルマネージドなシークレット管理サービス
  - アプリケーションの認証情報や API キーを安全に保存、管理
  - 自動的なシークレットのローテーションをサポート
  - シークレットのアクセス制御や監査ログを提供
  - マルチリージョンでのシークレットの複製をサポート

- AWS Directory Service for Microsoft Active Directory

  - Microsoft Active Directory を AWS 上でフルマネージドで提供
  - ドメインコントローラーのセットアップや運用、パッチ適用、バックアップなどを AWS が自動で管理
  - オンプレミスの Active Directory と信頼関係を構築可能（ハイブリッド構成が可能）
  - AWS の各種サービス（Amazon RDS、WorkSpaces、QuickSight など）と連携し、AD 認証やグループ管理が可能
  - 標準の Active Directory 管理ツール（AD 管理センター、グループポリシーなど）が利用可能

- AWS Directory Service Simple AD

  - フルマネージドな簡易的な Active Directory 互換ディレクトリサービス
    - Microsoft Active Directory の基本機能を Samba ベースで提供
  - 小規模な環境向けに設計されており、コスト効率が高い (max 5000 user, 20000 obj まで)
  - 基本的な AD 機能（ユーザー、グループ、ポリシー管理など）を提供
    - 一部の AD 機能 (信頼関係、AD FS、AD LDS) はサポートしていない
  - AWS の各種サービス（Amazon RDS、WorkSpaces、QuickSight など）と連携可能

- AWS CloudFormation

  - インフラストラクチャをコードとして管理
  - テンプレートを使用してリソースのプロビジョニングを自動化
  - バージョン管理や再利用が容易
  - カスタムリソースに lambda 関数を使用して、独自のリソースタイプを定義可能
  - リソースに DependsOn 属性を使用して、リソースの作成順序を制御可能
  - StackSets を使用して、複数のアカウントやリージョンにわたるスタックの一括管理が可能

- AWS Certificate Manager
  - SSL/TLS 証明書の管理を簡素化
  - 証明書の取得、更新、削除を自動化
  - AWS の各種サービス（Elastic Load Balancing、Amazon CloudFront など）と統合可能
    - CloudFront
      - SSL/TLS 証明書を使用して、コンテンツ配信を保護
    - Application Load Balancer
      - SSL/TLS 証明書を使用して、トラフィックを保護
    - Route 53
      - SSL/TLS 証明書を使用して、ドメイン名の検証を保護
- AWS Compute Optimizer

  - EC2 インスタンスや EBS ボリューム、 Lambda の最適化を支援
  - リソースの使用状況を分析し、最適なインスタンスタイプを推奨
  - コスト削減やパフォーマンス向上に寄与
  - Lambda などから SDK 経由で ExportLambdaFunctionRecommendations コマンドを送信することでレポートを CSV 形式でエクスポート可能

- Amazon WorkSpaces

  - フルマネージドな仮想デスクトップサービス（VDI: Virtual Desktop Infrastructure）
  - Windows、Amazon Linux 2、Ubuntu、Rocky Linux、Red Hat Enterprise Linux をサポート
  - 物理ハードウェアの調達や複雑なソフトウェアインストールが不要
  - 複数デバイスや Web ブラウザからアクセス可能

  ## WorkSpaces の種類

  ### WorkSpaces Personal

  - **永続的な仮想デスクトップ**
  - 個人専用のカスタマイズされたデスクトップ環境
  - 物理デスクトップと同様の使用感
  - 高度にパーソナライズされたデスクトップが必要なユーザー向け

  ### WorkSpaces Pools

  - **非永続的な仮想デスクトップ**
  - 厳密に管理されたデスクトップ環境
  - エフェメラルなインフラストラクチャ上でホスト
  - 標準化された環境へのアクセスが必要なユーザー向け

  ## 主要機能

  **認証・ディレクトリ統合**

  - Simple AD、AWS Managed Microsoft AD、AD Connector をサポート
  - オンプレミス Active Directory との統合
  - Microsoft Entra ID（旧 Azure AD）統合（Windows 10/11 のみ）
  - Microsoft Intune での管理対応

  **プロトコル**

  - PCoIP（PC-over-IP）
  - DCV（NICE Desktop Cloud Visualization）

  **セキュリティ**

  - 多要素認証（MFA）対応
  - AWS KMS による暗号化（データ、ディスク I/O、スナップショット）
  - IP アドレス制限機能

  **課金モデル**

  - **AlwaysOn**: 月額固定料金（無制限利用）
  - **AutoStop**: 時間課金（非接続時自動停止）

  ## バンドル（構成パッケージ）

  ### 基本バンドル

  - **Value**: 基本的なテキスト編集、軽い Web ブラウジング
  - **Standard**: Web ブラウジング、メール、インスタントメッセージング
  - **Performance**: ワード処理、スプレッドシート、音声処理
  - **Power**: ソフトウェア開発、データ処理、ビデオ会議
  - **PowerPro**: データウェアハウス、BI アプリケーション

  ### 高性能バンドル

  - **GeneralPurpose.4xlarge/8xlarge**: バッチ処理、CPU ベース ML トレーニング
  - **Compute**: 高いグラフィックス性能と CPU 性能
  - **Graphics.g4dn**: CAD/CAM、グラフィックデザイン
  - **GraphicsPro**: 3D レンダリング、ML トレーニング、ゲームストリーミング

  ## 技術要件・制限

  **ネットワーク要件**

  - PCoIP: 最大 250ms（推奨 100ms 以下）のネットワーク遅延
  - DCV: RTT 250ms 以下推奨（400ms 超でパフォーマンス大幅低下）
  - RTT 375ms 超で PCoIP 接続シャットダウン

  **カスタマイズ**

  - カスタムイメージとバンドルの作成可能
  - BYOL（Bring Your Own License）対応
  - AWS Marketplace for Desktop Apps からアプリケーション購入可能

  ## クライアントアプリケーション

  - Windows、macOS、iPad クライアント
  - Web Access（ブラウザベース）
  - Android クライアント（Chromebook 対応）
  - Linux クライアント
    ※ Amazon Linux WorkSpaces は Web Access 非対応

  ## 管理機能

  - 既存のデスクトップ管理ツールを利用可能
  - 自動スケーリング（WorkSpaces Pools）
  - カスタムイメージとバンドル管理
  - ユーザー招待とアクセス管理

  ## 参考 URL

  - サービス概要: https://docs.aws.amazon.com/workspaces/latest/adminguide/amazon-workspaces.html
  - バンドル詳細: https://aws.amazon.com/workspaces/details/#Amazon_WorkSpaces_Bundles
  - 料金情報: https://aws.amazon.com/workspaces/pricing/
