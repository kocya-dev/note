## セキュリティ

- AWS Global Accelerator
  - エッジロケーションから最適経路でのトラフィック転送を行う。
  - any cast IP による IP アドレス固定化
  - ヘルスチェック機能あり
    - フェイルオーバー数秒以内に完結
      (Route53 では数十秒～数分かかる)
- AWS Shield
  - DDoS 攻撃からの保護
  - AWS WAF と連携可能
  - AWS Shield Advanced
    - DDoS 攻撃の検知と自動対処
    - 24 時間体制の DDoS 専門家によるサポート
- AWS Firewall Manager
  - セキュリティポリシーの一元管理
  - 複数アカウントやリソースにまたがるポリシーの適用
  - 対象サービス
    - AWS WAF
    - AWS Shield Advanced
    - VPC セキュリティグループ
    - AWS Network Firewall
    - など
- AWS Network Firewall
  - VPC 内のトラフィックを監視・制御
  - セキュリティグループやネットワーク ACL と連携
  - カスタムルールの作成が可能
- AWS Security Token Service (STS)
  - 一時的なセキュリティ認証情報の発行
  - クロスアカウントアクセスやフェデレーションをサポート
  - IAM ロールと連携して、最小権限の原則を実現

複数 vpn をインターネットアクセス時に Network Firewall を経由させる構成例:

```

vpc[] - Transit Gateway - vpc[Network Firewall - NAT Gateway] - internet

```

- AWS WAF
  - ウェブアプリケーションファイアウォール
  - HTTP/HTTPS トラフィックのフィルタリング
  - OWASP Top 10 やカスタムルールに基づく保護
    - レート制限や IP アドレス制限の設定
    - リクエスト地域やヘッダー情報に基づくフィルタリング
    - サイズやパターンマッチングによるフィルタリング
    - SQL インジェクションやクロスサイトスクリプティング (XSS) の防止

client - WAF - CloudFront/API Gateway/AppSync/ALB
