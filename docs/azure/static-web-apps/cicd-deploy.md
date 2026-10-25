# CI/CD とコードデプロイ

## まず区別するもの

- **アプリコードのデプロイ**: フロントエンド / API / Functions の成果物を公開します。
- **インフラのデプロイ**: Bicep でリソースやアプリ設定、権限を変更します。

変更頻度と影響範囲が異なるため、通常はワークフローを分けます。コードだけの変更で Bicep を実行する必要はありません。

## 推奨する GitHub Actions の分割

| ワークフロー | 起動条件 | 処理 |
|---|---|---|
| SWA | PR と main への変更 | ビルド、SWA へ配置、PR プレビューの作成・クローズ |
| 独立 Functions | Functions の変更、手動実行 | ビルド、成果物パッケージ、Functions へデプロイ |
| インフラ | `infra/**` の変更、手動実行 | Azure 認証、Bicep 検証、what-if、承認・デプロイ |

パスフィルターを使う場合は、共有ライブラリや lockfile、ワークフロー自体の変更も対象に含めます。PR 時の Functions デプロイは、通常の開発者向けプレビュー環境への自動デプロイと同じものではありません。共有開発環境や本番へ PR からデプロイしないよう保護します。

## SWA の例

```yaml
name: Static Web Apps
on:
  push:
    branches: [main]
    paths: ['src/frontend/**', 'src/api/**']
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches: [main]
    paths: ['src/frontend/**', 'src/api/**']

jobs:
  build_deploy:
    if: github.event_name == 'push' || github.event.action != 'closed'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          action: upload
          app_location: /src/frontend
          api_location: /src/api # Managed Functions を使わない場合は空欄
          output_location: dist
  close_preview:
    if: github.event_name == 'pull_request' && github.event.action == 'closed'
    runs-on: ubuntu-latest
    steps:
      - uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
          action: close
```

これはパスや出力先を示す雛形です。採用フレームワーク、SWA CLI / GitHub Action の現在の仕様、リポジトリのフォルダー構成に合わせて修正します。`output_location` は通常 `app_location` を基準としたビルド出力フォルダーです。BYOF だけを使う場合は `api_location` を指定しません。

## 独立 Functions の例

Functions のランタイム、OS、プラン、ビルド方式により必要なパッケージ内容が異なります。Azure Functions Action の公式手順に沿って、依存関係を含む配置パッケージを作成します。Node.js の例:

```yaml
permissions:
  id-token: write
  contents: read

steps:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
    with:
      node-version: '20'
      cache: npm
      cache-dependency-path: src/batch-functions/package-lock.json
  - run: npm ci
    working-directory: src/batch-functions
  - run: npm run build --if-present
    working-directory: src/batch-functions
  - uses: azure/login@v3
    with:
      client-id: ${{ secrets.AZURE_CLIENT_ID }}
      tenant-id: ${{ secrets.AZURE_TENANT_ID }}
      subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
  - uses: Azure/functions-action@v1
    with:
      app-name: ${{ vars.AZURE_FUNCTIONAPP_NAME }}
      package: src/batch-functions
```

この Functions の YAML は骨格です。ビルド出力と本番依存関係をパッケージに含める方法はプロジェクト構成ごとに確認します。Flex Consumption でリモートビルドする場合は `remote-build: true` を使い、他のリモートビルド設定と併用しません。

GitHub Actions から Azure へは可能なら OpenID Connect (OIDC) と `azure/login` を使います。Entra ID 側に federated credential を設定し、GitHub の Environment / repository / branch 条件を絞り、必要最小限の Azure RBAC を付与します。SWA のデプロイトークンは別の資格情報です。GitHub Secrets に保管し、漏えい時は再発行します。

## 手動コードデプロイ

緊急対応などでローカルからデプロイする場合も、環境・対象・承認を記録し、通常運用の CI/CD と同等の成果物を使うのが望ましいです。

```sh
# SWA CLI のログインとデプロイ
swa login --app-name <swa-name> --resource-group <resource-group>
swa deploy ./dist --env production

# Azure Functions Core Tools を使った例
func azure functionapp publish <function-app-name>
```

オプションや認証挙動は CLI バージョンで異なるため、実行前に `swa deploy --help` / `func azure functionapp publish --help` と公式手順を確認します。CI ではローカルの `az login` に依存しない認証方式を使います。

## 本番リリースの流れ

1. PR でビルドとレビューを通す。
2. PR プレビューで画面を確認する。
3. main にマージして成果物をデプロイする。
4. デプロイ後にヘルスチェック、ログ、メトリックを確認する。
5. 失敗時は直前のリリース成果物へ戻すか、修正を再デプロイする。

SWA の GitHub Actions は通常 PR プレビューも管理します。標準ブランチ保護、GitHub Environments の承認、環境別 Secrets を利用し、開発・検証・本番の接続先を混ぜないようにします。

## 公式資料

- [SWA の GitHub Actions ビルド構成](https://learn.microsoft.com/azure/static-web-apps/build-configuration)
- [SWA のデプロイ トークン](https://learn.microsoft.com/azure/static-web-apps/deployment-token-management)
- [Azure Functions 用 GitHub Actions](https://learn.microsoft.com/azure/azure-functions/functions-how-to-github-actions)
- [Azure Login Action / OIDC](https://github.com/Azure/login)
