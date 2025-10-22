## Migration

- AWS Migration Hub

  - 移行プロジェクトの進捗を追跡するためのサービス
  - 移行に関するメトリクスやレポートを提供
  - AWS の他の移行サービスとの統合が容易
  - AWS Athena と統合し、移行データの分析をサポート
  - インスタンスタイプの推奨機能を提供
    - 移行対象のオンプレミスサーバーの CPU、メモリ、ストレージ使用率に基づいて、最適な EC2 インスタンスタイプを推奨

- AWS Application Discovery Service

  - オンプレミス環境のインベントリを自動的に収集
  - アプリケーションの依存関係をマッピング
  - 移行計画の策定を支援
  - AWS Migration Hub と統合
  - Application Discovery Agent
    - オンプレミス環境の詳細な分析を実行
  - Agentless Discovery Connector
    - エージェントをインストールせずにオンプレミス環境の情報を収集
      Discovery Agent と比較して、収集できる情報は限定的

- Migration Evaluator

  - 移行のコストとリスクを評価するためのツール
  - 現在のオンプレミス環境を分析し、AWS への移行に関する推奨事項を提供
  - VMware 環境の移行をサポート
  - Migration Evaluation エージェントを使用して、オンプレミス環境の詳細な分析を実行、分析結果は AWS Migration Hub に送信される

- AWS Application Migration Service

  - オンプレミス環境から AWS へのアプリケーション移行を簡素化
    - VM を AWS に移行するためのツールを提供
  - アプリケーションの移行、変換、検証をサポート

- AWS Database Migration Service

  - オンプレミス環境から AWS へのデータベース移行を簡素化
  - データベースの移行、変換、検証をサポート
  - AWS の他の移行サービスとの統合が容易
    - ソースとして Cassandra および、データターゲットとして Amazon Keyspaces はサポートしていない。
