# 開発プロセスとローカル環境

## リポジトリの例

```text
repo/
├─ src/frontend/          # SPA
├─ src/api/               # Managed Functions を採用する場合
├─ src/batch-functions/   # 独立 Functions
├─ infra/                 # Bicep と環境別パラメーター
└─ .github/workflows/     # CI/CD
```

実際のフレームワークに合わせて構成は調整します。機密値、接続文字列、ローカル設定ファイルは Git にコミットしません。

## 開発の標準フロー

1. **要件を分ける**: 画面要求、短時間の HTTP API、非同期処理、定期処理、データ保存、認証・ネットワーク要件に分解します。
2. **サービスを選ぶ**: 45 秒以内に応答する API は SWA 経由、長時間・イベント処理は独立 Functions とします。イベントの特性に応じて Event Hubs / Service Bus 等を選定します。
3. **Bicep で開発環境を用意**: リソースグループ、SWA、Functions、データサービス、ID と権限をデプロイします。
4. **ローカルで開発・確認**: フロントはフレームワーク標準の開発サーバー、API は SWA CLI と Azure Functions Core Tools を使って統合確認します。
5. **ブランチと PR**: 小さな変更単位でブランチを作成し、静的解析・ビルド・必要な自動確認を通してレビューします。
6. **プレビューで画面確認**: PR ごとに作られる SWA 環境をレビューに使います。BYOF の API は PR 環境にリンクされないため、API 結合確認方法を別に決めます。
7. **main へマージ**: 本番環境へのコードデプロイが実行されます。インフラ変更は別ジョブまたは別ワークフローで扱います。
8. **運用確認**: Application Insights / Azure Monitor のログとメトリックを確認し、障害対応と再処理手順を整えます。

## ローカル開発の準備

- Node.js など、採用したフロントエンド・Functions ランタイム
- Azure CLI (`az`)、Static Web Apps CLI (`swa`)、Azure Functions Core Tools (`func`)
- Bicep CLI（Azure CLI から利用するか、個別に導入）
- Azure サブスクリプションと、開発リソースを作成する権限

```sh
# Functions Core Tools で API を別途 localhost:7071 に起動している場合
swa start http://localhost:5173 --api-location http://localhost:7071
```

API フォルダーを SWA CLI に直接渡す構成では、`--api-location ./src/api` のように配置を指定します。

フロントエンドと API をローカルで統合する場合、SWA CLI の `swa start` がローカルのフロントエンド開発サーバーと API をつなぎます。Functions Core Tools が API を実行します。プロジェクトのランタイム・CLI バージョンに合わせて公式手順を参照してください。

```sh
# Functions Core Tools で API を別途 localhost:7071 に起動している場合
swa start http://localhost:5173 --api-location http://localhost:7071
```

API フォルダーを SWA CLI に直接渡す構成では、`--api-location ./src/api` のように配置を指定します。

Functions の `local.settings.json` はローカル専用です。秘密値を含め得るため `.gitignore` 対象とし、Azure 側の設定と混同しないようにします。開発データは本番データと分離してください。

## PR で確認すること

- フロントエンドのビルドが成功する。
- API の入力検証、認証・認可、異常応答を確認する。
- バッチの重複実行、再試行、失敗記録を確認する。
- Bicep の lint / build / what-if を実施し、想定外の変更を検出する。
- PR プレビューで画面を確認し、BYOF の場合は接続先や代替テスト方法を明示する。

## 主な公式資料

- [SWA CLI を使ったローカル開発](https://learn.microsoft.com/azure/static-web-apps/local-development)
- [SWA プレビュー環境](https://learn.microsoft.com/azure/static-web-apps/preview-environments)
- [Azure Functions Core Tools](https://learn.microsoft.com/azure/azure-functions/functions-run-local)
