# Azure Startup Guide

Azure Entra ID にアプリを登録し、GitHub Actions の OIDC を利用して安全に Azure にデプロイするための最小構成手順をまとめる。

前提として、GitHub Actions から Azure へアクセスするには以下の情報が必要になる。

```
- Entra ID > アプリの登録
  - アプリID（clientId）
  - テナントID（tenantId）
  - オブジェクトID（objectId）
- Azure サブスクリプション
  - サブスクリプションID
- GitHub リポジトリ
  - owner / repo 名
  - ブランチ名（例: main, master）
```

## 手順

1. Azure CLI をインストールする

   ```bash
   az version
   ```

2. Azure にログインする

   ```bash
   az login
   ```

3. Entra ID でアプリを登録する

   ```bash
   az ad app create --display-name "<アプリ名>"
   ```

4. アプリの識別子を確認する

   ```bash
   az ad app list --display-name "<アプリ名>" --query "[].{appId:appId, objectId:id, displayName:displayName}" -o table
   ```

   - `appId` はアプリのクライアントID
   - `id` はアプリのオブジェクトID
   - `tenantId` は Azure テナントID

5. サブスクリプションID を確認する

   ```bash
   az account show --query "{subscriptionId:id, tenantId:tenantId, name:name}" -o table
   ```

   または:

   ```bash
   az account show --query id -o tsv
   ```

6. リソースグループを作成する

   ```bash
   az group create --name "my-rg-tokyo" --location japaneast
   ```

   作成済みのグループ確認:

   ```bash
   az group list --output table
   ```

7. 対象リソースに対するロールを割り当てる

   ```bash
   az role assignment create \
     --role Contributor \
     --scope "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName" \
     --assignee-object-id $assigneeObjectId \
     --assignee-principal-type ServicePrincipal
   ```

   - Web 画面からアプリを登録した場合は、既にロールが割り当てられていることがある
   - 実際のデプロイ先に応じて `Contributor` 以外の権限が必要になる場合がある

8. フェデレーション用の対象情報を整理する
   GitHub Actions の OIDC では、`subject` は GitHub リポジトリとブランチを一致させる必要がある。代表的な形式は次のとおり。

   ```bash
   gh api repos/{owner}/{repo} --jq '{owner_id: .owner.id, repository_id: .id}'
   ```

   - `subject` に含める値は、実際に GitHub Actions が実行されるブランチに合わせて指定する。例: `main` を使用するなら `refs/heads/main` を指定する。
   - `owner_id` と`repository_id` もそれぞれ`subject` に含める必要がある。

9. フェデレーション用 JSON を作成する

```json
{
  "name": "<CREDENTIAL-NAME>",
  "issuer": "https://token.actions.githubusercontent.com",
  "subject": "repo:<owner>@<owner ID>/<repo>@<repo ID>:ref:refs/heads/main",
  "description": "GitHub Actions OIDC federation",
  "audiences": ["api://AzureADTokenExchange"]
}
```

- `issuer` は GitHub Actions の OIDC 発行元
- `audiences` は Azure で定められた既定値
- `subject` は GitHub の `owner/repo:ref:refs/heads/<branch>` を厳密に一致させる

10. フェデレーション用 JSON を Azure に登録する

    ```bash
    az ad app federated-credential create \
      --id <APPLICATION-OBJECT-ID> \
      --parameters <jsonファイルパス>
    ```

    - `<APPLICATION-OBJECT-ID>` は `az ad app list` の `id`
    - JSON には `name`, `issuer`, `subject`, `description`, `audiences` を含める

11. GitHub リポジトリの Secrets を登録する
    GitHub の Settings > Secrets and variables > Actions で以下を登録する。

    ```text
    AZURE_CLIENT_ID=<アプリの appId>
    AZURE_TENANT_ID=<az account show --query tenantId --output tsv>
    AZURE_SUBSCRIPTION_ID=<az account show --query id -o tsv>
    ```

    GitHub Actions では OIDC を使うため、通常は `azure/login` アクションにこれらの値を渡す。

12. デプロイの確認
    GitHub Actions が実行されると、`azure/login` と `az account show` などで Azure に接続できることを確認する。
    ここで `subject` とリポジトリ名、ブランチ名の一致がずれていると認証に失敗するため注意する。
