# Azure Static Web Apps を使った開発標準プロセス

この資料は、Azure に不慣れな開発者が、Static Web Apps (SWA) を中心に Web アプリを設計・開発・デプロイするための標準的な進め方を説明します。フロントエンドと軽量 API は SWA にまとめ、時間のかかるバッチやイベント処理は独立した Azure Functions に任せ、Azure リソースは Bicep で管理する構成を例にします。

## この資料の読み方

1. [アーキテクチャとサービスの役割](./architecture.md)
2. [開発プロセスとローカル環境](./development.md)
3. [CI/CD とコードデプロイ](./cicd-deploy.md)
4. [Bicep による環境構築](./infrastructure.md)
5. [運用上の Tips と設計判断](./tips.md)

## 想定する標準構成

- **Azure Static Web Apps**: SPA などの静的ファイル配信、認証・ルーティング、軽量な HTTP API。
- **独立した Azure Functions**: Timer / Event Hubs / Service Bus / Cosmos DB などのトリガーを使うバッチ・非同期処理。
- **Cosmos DB / Blob Storage / Event Hubs 等**: アプリケーションデータ、ファイル、イベントの保存・連携。
- **Bicep**: Azure リソース、設定、RBAC をコードで定義。
- **GitHub Actions**: PR 検証、SWA のプレビュー、本番コードデプロイ、必要時のインフラデプロイ。

この設計では、日々のアプリ変更はコードだけをデプロイし、リソース構成を変えるときに Bicep をデプロイします。要件に応じて Managed Functions / BYOF、Event Hubs / Service Bus などを選びます。

> Azure の料金、対応ランタイム、リージョン、各サービスの上限は変わることがあります。実環境の設計時には本文中の Microsoft Learn と料金ページで最新情報を確認してください。
