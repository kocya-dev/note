# Bicep による環境構築

## IaC の役割

Bicep は Azure Resource Manager の宣言型 IaC 言語です。リソースの作成・更新、設定、ロール割り当てをコードで再現し、開発・検証・本番の差分をパラメーターで管理します。Azure 中心の構成では Bicep が有力な選択肢です。複数クラウドや Azure 外 SaaS まで同じ IaC で管理する要件があれば Terraform 等も比較します。

## ファイル構成例

```text
infra/
├─ main.bicep
├─ modules/
│  ├─ swa.bicep
│  ├─ functions.bicep
│  ├─ data.bicep
│  └─ monitoring.bicep
└─ environments/
   ├─ dev.bicepparam
   ├─ test.bicepparam
   └─ prod.bicepparam
```

秘密値はパラメーターファイルや Git に平文で置かず、マネージド ID、Key Vault、GitHub OIDC などを使います。環境差分は名前、SKU、冗長性、診断設定などに限定し、同じテンプレートを使うのが基本です。

## 段階的な導入

1. Bicep と Azure CLI を準備し、テンプレートをコンパイル・lint します。
2. 開発用リソースグループを用意し、必要なリソースと依存関係を定義します。
3. `what-if` で変更予定を確認してからデプロイします。
4. Functions のシステム割り当てマネージド ID を有効化し、Cosmos DB / Storage / Event Hubs に必要なデータプレーンロールを付与します。
5. アプリ設定、監視、診断、ネットワーク制御をコードで管理します。
6. CI/CD で検証とデプロイを自動化し、本番は保護された Environment の承認後に実行します。

```sh
az bicep build --file infra/main.bicep
az deployment group what-if \
  --resource-group <resource-group> \
  --template-file infra/main.bicep \
  --parameters infra/environments/dev.bicepparam
az deployment group create \
  --resource-group <resource-group> \
  --template-file infra/main.bicep \
  --parameters infra/environments/dev.bicepparam
```

Bicep デプロイの実行主体には、リソース作成だけでなくロール割り当てを行う権限が必要です。データアクセス用 RBAC をデプロイする場合、一般的な Contributor だけでは不足することがあります。権限は最小化し、昇格権限を恒常付与しない運用を検討します。

## Deployment Stacks を使う場合

Deployment Stacks はデプロイしたリソースの管理と、テンプレートから外れたリソースの扱いを制御する機能です。利用する場合は `action-on-unmanage` の選択が重要です。`deleteResources` では管理対象から外れたリソースを削除し得ます。誤削除を避けるため、初回は通常の ARM デプロイと what-if から始め、Stack の削除・更新挙動を検証してから導入します。deny 設定も通常運用や障害対応を妨げないよう設計します。

```sh
az stack group create \
  --name <stack-name> \
  --resource-group <resource-group> \
  --template-file infra/main.bicep \
  --parameters infra/environments/dev.bicepparam \
  --action-on-unmanage detachAll \
  --deny-settings-mode none
```

`detachAll` は管理対象から外れたリソースを残してスタック管理から切り離します。環境の廃棄も含めて削除動作を明示的に検討した後に設定してください。複数スタックに分割する場合、共有リソースの所有者を一つに定め、重複管理を避けます。

## アプリ設定の扱い

Bicep で Function App のアプリ設定を宣言する場合、その設定値を IaC の唯一の管理元とします。ポータルでの手動変更は次の IaC デプロイで戻る・同期される可能性があります。アプリ設定を配列/辞書で管理する方法や resource API の挙動により更新の詳細は異なるため、対象リソースのテンプレート仕様と what-if で確認します。

## 公式資料

- [Bicep の概要](https://learn.microsoft.com/azure/azure-resource-manager/bicep/overview)
- [Bicep のデプロイ](https://learn.microsoft.com/azure/azure-resource-manager/bicep/deploy-cli)
- [Bicep デプロイ スタック](https://learn.microsoft.com/azure/azure-resource-manager/bicep/deployment-stacks)
- [What-if デプロイ](https://learn.microsoft.com/azure/azure-resource-manager/bicep/deploy-what-if)
- [Azure リソースへのマネージド ID アクセス](https://learn.microsoft.com/entra/identity/managed-identities-azure-resources/overview)
