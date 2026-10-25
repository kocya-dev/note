# 運用上の Tips と設計判断

## 認証とデータアクセス

- ブラウザーに Cosmos DB や Storage の秘密鍵を埋め込まない。サーバー側 API が認可し、データサービスへアクセスします。
- Azure リソース間の認証にはマネージド ID と最小権限の RBAC を優先します。
- SWA の組み込み認証・ロール制御で足りるか、Entra ID 等の要件と照合します。バックエンド API 側でも認可を検証し、フロントの表示制御だけに依存しません。
- Blob の大容量アップロードでは、API が短時間・限定権限の SAS を発行し、ブラウザーから Blob に直接アップロードする方法を検討します。許可する操作・Blob・有効期限を絞ります。

## Event Hubs と Service Bus

| 要件 | 主な候補 |
|---|---|
| 高スループットのイベントストリーム、複数コンシューマー、順序付きパーティション処理 | Event Hubs |
| 業務コマンドやジョブのキュー、配信制御、DLQ、予約実行 | Service Bus |
| 時刻ベースの定期実行 | Azure Functions Timer Trigger |
| Cosmos DB の変更検知 | Cosmos DB Trigger / Change Feed |

Event Hubs はイベントを時間指定で配信するジョブスケジューラーではありません。再試行・チェックポイント・重複イベントへの対処を設計し、単一イベントとバッチ受信のどちらが適するかを選びます。

## API とアップロード

- SWA を通る API は 45 秒制限を前提にし、長時間処理を HTTP 呼び出し内で待たせません。
- `staticwebapp.config.json` で SPA のナビゲーション、ルート認可、ヘッダーなどを設定します。構成 JSON の仕様にない設定を独自に仮定しないよう注意します。
- CORS は SWA と同一オリジンの `/api` を使う通常の呼び出しでは不要になりやすい一方、Blob へのブラウザー直送信や別ドメイン API には個別設定が必要です。
- WebSocket やネットワーク分離バックエンドなど SWA API の制約に合わない要件は、Front Door / API Management 等を含む別構成と比較します。

## 環境・リリース管理

- 開発・検証・本番でデータ、ID、シークレット、Functions のトリガー先を分離します。
- PR プレビューで外部 API を使う場合、テスト用環境・データを使います。SWA の BYOF は PR プレビューにリンクされません。
- SWA のブランチ環境は安定したプレビュー URLを作れますが、カスタムドメインは使えず、地理分散もされません。
- SWA デプロイトークン、Functions の OIDC、Azure RBAC を別々の認証経路として管理します。
- リリースとインフラ変更を追跡できるよう、Git のコミット、環境、実行結果を紐付けます。

## 監視・障害対応

- Functions と API のログ・例外、処理時間、失敗数を Application Insights / Azure Monitor で観測します。
- ジョブ ID、相関 ID、イベント ID をログに残し、画面受付から非同期処理まで追跡可能にします。
- 失敗キュー / DLQ、再処理条件、データ重複時の挙動、アラート先を事前に決めます。
- 秘密情報、個人情報、トークン、SAS URL をログに出さないようにします。

## 公式資料

- [SWA の構成ファイル](https://learn.microsoft.com/azure/static-web-apps/configuration)
- [SWA のクォータと制限](https://learn.microsoft.com/azure/static-web-apps/quotas)
- [Blob Storage のユーザー委任 SAS](https://learn.microsoft.com/azure/storage/blobs/storage-blob-user-delegation-sas-create-cli)
- [Service Bus のスケジュールメッセージ](https://learn.microsoft.com/azure/service-bus-messaging/message-sequencing#scheduled-messages)
- [Cosmos DB Trigger for Functions](https://learn.microsoft.com/azure/azure-functions/functions-bindings-cosmosdb-v2-trigger)
- [Azure Functions の監視](https://learn.microsoft.com/azure/azure-functions/functions-monitoring)
