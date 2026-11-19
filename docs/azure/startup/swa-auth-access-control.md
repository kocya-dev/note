# SWA 組み込み認証によるアクセス制限の調査

- 日付: 2026-11-19
- 状態: 調査メモ（未決定。採用する場合は ADR を追加する）

ADR 0007 では、アクセス制御を `authenticated` ロールのままにしています。そのため、ログインできる人なら誰でも全データを扱えます。このメモでは、利用者を「招待したユーザー」または「指定したテナントのユーザー」に限定する方法を、SWA の組み込み認証を使う前提で整理します。

## 対象範囲

この調査では、SWA が提供する `/.auth/*` の仕組み（組み込み認証）を使う方法だけを扱います。組み込み認証には次の 2 つの形態があります。

| 形態                     | 内容                                                                                                                            | 使えるプラン   |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------- | -------------- |
| 事前構成済みプロバイダー | SWA が管理するアプリ登録を使う。設定なしで `/.auth/login/aad`（Entra ID）と `/.auth/login/github` が使える                      | Free・Standard |
| カスタムプロバイダー     | 自分で作った Entra ID のアプリ登録を、`staticwebapp.config.json` の `auth` で SWA に登録する。ログイン処理は引き続き SWA が行う | Standard のみ  |

アプリのコードで MSAL などを使ってログインを実装する方法や、Front Door・APIM などを前段に置く方法は、対象外とします。

## 現状の構成

- `src/frontend/public/staticwebapp.config.json` で全ルート（`/*`）を `authenticated` ロールに限定しています。未認証の 401 は `/.auth/login/aad` へ 302 リダイレクトします。
- 事前構成済みのプロバイダーを使っています。Bicep（`infra/main.bicep`）には認証の設定がなく、SKU は `Free` です。
- API は `x-ms-client-principal` ヘッダーから `userId` を取り出します（`src/api/src/lib/http.ts` の `getPrincipal`）。

> **現状の注意点**: 事前構成済みのプロバイダーでは、GitHub でのログイン（`/.auth/login/github`）も有効です。Entra ID のアカウントを持たない人でも、GitHub のアカウントでログインすれば `authenticated` になります。どの方式を採るかにかかわらず、塞いでおくことをおすすめします（「方式 A」の手順 1）。

## プランごとにできること

| やりたいこと                                                       | Free             | Standard                                               |
| ------------------------------------------------------------------ | ---------------- | ------------------------------------------------------ |
| ログインした人だけに限定する（現状）                               | ✅               | ✅                                                     |
| 招待したユーザーだけに限定する                                     | ✅（最大 25 人） | ✅（招待では最大 25 人）                               |
| 指定したテナントだけに限定する                                     | ❌               | ✅（カスタムプロバイダー）                             |
| テナント内の特定のユーザー・グループだけに限定する（人数制限なし） | ❌               | ✅（Entra の「割り当てが必要」、または `rolesSource`） |
| ロールを関数で動的に割り当てる（`rolesSource`）                    | ❌               | ✅（カスタムプロバイダーが前提）                       |
| 料金                                                               | 無料             | 有料（アプリごとに月額。目安は約 9 USD/月）            |

Free でテナントを制限できない理由:

- 事前構成済みの `aad` プロバイダーは、SWA が管理するマルチテナントのアプリ登録を使います。発行元（テナント）を指定する設定がありません。
- 組み込み認証で API に渡る `x-ms-client-principal` には、テナント ID（`tid`）などのクレームが含まれません。そのため、API 側でテナントを判定することもできません。
- `userDetails`（メールアドレス）のドメインで判定する方法も考えられますが、テナントを保証するものではないため採用しません。

---

## 方式 A: 招待制（Free・Standard）

SWA のロール管理にある招待機能で、独自ロール（例: `member`）を特定のユーザーに割り当てます。そのうえで、全ルートをそのロールに限定します。

### 仕組み

1. 招待したいユーザーのメールアドレスとロールを指定して、招待リンクを発行します。
2. ユーザーがリンクを開いて Entra ID でログインすると、そのユーザーに `member` ロールが付きます。
3. ロールのない人もログインはできますが、どのルートでも 403 になります。

### 設定手順

1. **`staticwebapp.config.json` を変更する**

   ```json
   {
     "routes": [
       { "route": "/.auth/login/github", "statusCode": 404 },
       { "route": "/forbidden.html", "allowedRoles": ["anonymous", "authenticated"] },
       { "route": "/*", "allowedRoles": ["member"] }
     ],
     "navigationFallback": {
       "rewrite": "/index.html",
       "exclude": ["/api/*", "/assets/*", "/*.{css,js,svg,png,ico,json,html}"]
     },
     "responseOverrides": {
       "401": { "statusCode": 302, "redirect": "/.auth/login/aad" },
       "403": { "rewrite": "/forbidden.html" }
     },
     "platform": { "apiRuntime": "node:22" }
   }
   ```

   - ルールは上から順に評価されます。先に GitHub ログインを塞ぎ、403 の画面のパスを除外してから、`/*` を `member` に限定します。
   - `/api/*` も `/*` に含まれるため、API も `member` だけが呼べます。
   - 403 の画面（`src/frontend/public/forbidden.html`）を新しく作ります。権限がないことと、管理者へ招待を依頼する方法を載せます。
   - `navigationFallback.exclude` に `html` を追加し、`forbidden.html` が SPA の `index.html` に書き換えられないようにします。

2. **ユーザーを招待する**（ポータル、または Azure CLI）

   - ポータルの場合: Static Web App →「ロール管理」→「招待」で、プロバイダーに「Azure Active Directory」を選び、メールアドレス・ロール（`member`）・有効期限を入力します。発行されたリンクを本人に送ります。
   - Azure CLI の場合:

     ```bash
     az staticwebapp users invite \
       --name <SWA名> --resource-group <リソースグループ> \
       --authentication-provider aad \
       --user-details <招待するユーザーのメールアドレス> \
       --roles member \
       --domain <SWAの既定ホスト名> \
       --invitation-expiration-in-hours 168
     ```

3. **ロールを確認・変更・削除する**

   ```bash
   az staticwebapp users list   --name <SWA名> --resource-group <リソースグループ>
   az staticwebapp users update --name <SWA名> --resource-group <リソースグループ> \
     --authentication-provider aad --user-details <メールアドレス> --roles member
   ```

   ロールを外すと、そのユーザーはアクセスできなくなります。ポータルの「ロール管理」でも、ユーザーごとにロールを編集・削除できます。

### 注意点

- 招待で割り当てられるのは最大 25 人です（Free・Standard 共通）。
- 招待リンクの有効期限は最長 168 時間（7 日）です。期限が切れたら発行し直します。
- テナントは制限されません。招待された人なら、どのテナントのアカウントでもアクセスできます。
- 招待とロールの割り当ては手作業です。Bicep やワークフローでは管理しません。
- PR のプレビュー環境でロールの割り当てが共有されるかどうかは、導入時に確認します。
- ローカル開発では、SWA CLI（`swa start`）の認証エミュレーターでロールに `member` を入力すると再現できます。

### このリポジトリで変更するもの

- `src/frontend/public/staticwebapp.config.json`
- `src/frontend/public/forbidden.html`（新規）
- `docs/architecture.md`（アクセス制御の説明）、`docs/development.md`（ローカルでのロールの指定）
- ADR（ADR 0007 のアクセス制御の判断を置き換える）
- 招待の運用手順（チェックリストの「手作業」）

---

## 方式 B: テナント指定（Standard のみ）

自分で作った Entra ID のアプリ登録を、カスタムプロバイダーとして SWA に登録します。発行元を指定したテナントに固定すると、そのテナントのユーザーしかログインできなくなります。

### 仕組み

- `openIdIssuer` にテナント ID を含む URL を指定し、アプリ登録を「単一テナント」にします。ほかのテナントのユーザーは、Entra ID のサインインで拒否されます。
- カスタムプロバイダーを設定すると、事前構成済みのプロバイダー（GitHub を含む）はすべて無効になります。
- ログインのパスは `/.auth/login/aad` のまま変わりません。今の 401 リダイレクトはそのまま使えます。

### 設定手順

1. **SWA を Standard にする**（`infra/main.bicep`）

   ```bicep
   sku: {
     name: 'Standard'
     tier: 'Standard'
   }
   ```

2. **Entra ID でアプリを登録する**（手作業。対象テナントで行う）

   1. 「アプリの登録」→「新規登録」で、サポートされているアカウントの種類に「この組織ディレクトリのみ（シングルテナント）」を選びます。
   2. リダイレクト URI に、種類「Web」で `https://<SWAの既定ホスト名>/.auth/login/aad/callback` を追加します。カスタムドメインを使う場合は、そのホスト名の URI も追加します。
   3. 「認証」で「ID トークン」の発行を有効にします。
   4. 「証明書とシークレット」でクライアントシークレットを作り、値を控えます。
   5. 「概要」から、アプリケーション（クライアント）ID とディレクトリ（テナント）ID を控えます。

3. **SWA のアプリケーション設定にクライアント ID とシークレットを入れる**

   ```bash
   az staticwebapp appsettings set --name <SWA名> --resource-group <リソースグループ> \
     --setting-names AAD_CLIENT_ID=<クライアントID> AAD_CLIENT_SECRET=<シークレット>
   ```

   このリポジトリでは、ほかのアプリケーション設定と同じく、ワークフロー（`.github/workflows/deploy.yml`）からリポジトリの secret を使って設定します。値はソースファイルに書きません。

4. **`staticwebapp.config.json` に `auth` を追加する**

   ```json
   {
     "auth": {
       "identityProviders": {
         "azureActiveDirectory": {
           "registration": {
             "openIdIssuer": "https://login.microsoftonline.com/<テナントID>/v2.0",
             "clientIdSettingName": "AAD_CLIENT_ID",
             "clientSecretSettingName": "AAD_CLIENT_SECRET"
           }
         }
       }
     },
     "routes": [{ "route": "/*", "allowedRoles": ["authenticated"] }],
     "responseOverrides": {
       "401": { "statusCode": 302, "redirect": "/.auth/login/aad" }
     }
   }
   ```

   テナント ID は秘密情報ではありませんが、環境ごとに変わる値です。複数の環境を持つ場合は、ビルド時に差し込む方法を検討します。

### テナント内のユーザーをさらに絞る場合（任意）

- **方法 1: Entra 側で絞る（簡単。おすすめ）**
  エンタープライズアプリケーション（同じ名前のアプリ）→「プロパティ」で「割り当てが必要ですか?」を「はい」にします。「ユーザーとグループ」で、許可するユーザーやグループを割り当てます。割り当てられていない人は、Entra ID のサインインで拒否されます。人数の上限はなく、SWA の設定は `authenticated` のままで済みます。
- **方法 2: SWA 側でロールを決める（`rolesSource`）**
  `auth.rolesSource` に API の関数（例: `/api/roles`）を指定します。ログインのたびに SWA がその関数を POST で呼び出します。関数は、本文の `claims`（グループ ID など）から判定したロールを `{ "roles": ["member"] }` の形で返します。ルートは `allowedRoles: ["member"]` に限定します。画面や API ごとに権限を分けたいときに向いています。グループのクレームを使う場合は、アプリ登録の「トークン構成」でグループ要求を追加します。

### 注意点

- **PR のプレビュー環境**: プレビュー環境はホスト名が PR ごとに変わります。一方、Entra のリダイレクト URI にはワイルドカードを使えません。今のワークフローは PR ごとにプレビュー環境を作るため、プレビュー用の URI を登録する手順を追加するか、プレビューでのログイン確認をあきらめるかを決める必要があります。
- **シークレットの更新**: クライアントシークレットには有効期限があります。期限が切れる前に作り直し、secret を更新する運用が必要です。
- **料金**: Standard の月額料金がかかります。
- **ローカル開発**: SWA CLI の認証エミュレーターでは、テナントの制限は再現されません。本物の Entra ID での確認は、デプロイした環境で行います。

### このリポジトリで変更するもの

- `infra/main.bicep`（SKU）
- `src/frontend/public/staticwebapp.config.json`（`auth`）
- `.github/workflows/deploy.yml`（`AAD_CLIENT_ID`・`AAD_CLIENT_SECRET` の設定と、リポジトリの secret）
- `rolesSource` を使う場合は、`src/api/src/functions/` の新しいハンドラーと `src/api/src/index.ts` への import
- `docs/architecture.md`、`docs/development.md`、`AGENTS.md`（secret 名とプラン）
- ADR（Standard への変更とアクセス制御の判断）
- アプリ登録とユーザー割り当ての手順（チェックリストの「手作業」）

---

## 方式の選び方

| 状況                                         | おすすめ                                          |
| -------------------------------------------- | ------------------------------------------------- |
| PoC のまま、知っている数人だけに絞りたい     | 方式 A（Free のまま招待制）                       |
| 組織のテナントのユーザーだけにしたい         | 方式 B                                            |
| 組織のテナントの、さらに一部の人だけにしたい | 方式 B ＋「割り当てが必要」                       |
| 25 人を超える人を個別に許可したい            | 方式 B ＋「割り当てが必要」、または `rolesSource` |

どちらの方式でも、API のコード（`getPrincipal`）は変える必要がありません。ロールが必要な場合は、`x-ms-client-principal` の `userRoles` で確認できます。

## 参考資料

- [Azure Static Web Apps の認証と承認](https://learn.microsoft.com/azure/static-web-apps/authentication-authorization)
- [Azure Static Web Apps のカスタム認証](https://learn.microsoft.com/azure/static-web-apps/authentication-custom)
- [Azure Static Web Apps でユーザー情報にアクセスする](https://learn.microsoft.com/azure/static-web-apps/user-information)
- [Microsoft Graph を使ってロールを割り当てる（`rolesSource`）](https://learn.microsoft.com/azure/static-web-apps/assign-roles-microsoft-graph)
- [Azure Static Web Apps のプラン](https://learn.microsoft.com/azure/static-web-apps/plans)
- [Azure Static Web Apps のクォータ](https://learn.microsoft.com/azure/static-web-apps/quotas)
