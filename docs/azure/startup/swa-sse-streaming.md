# SWA マネージド Functions で SSE を逐次届けるための技術メモ

- 日付: 2026-11-20
- 状態: 技術メモ（決定は [ADR 0005](../decisions/0005-ai-review-streaming.md)）
- 関連: PR #16（判定コメントの SSE 化）

Azure Static Web Apps（SWA）のマネージド Functions（Node.js v4）から、Server-Sent Events（SSE）をブラウザーに逐次届けるための条件と制約をまとめます。PR #16 のプレビュー環境での測定結果をもとにしています。

## 要点

- 応答の `Content-Type` は、`text/event-stream` と**完全に一致**させる。`; charset=utf-8` を付けるだけで、SWA の経路は応答全体をためてから返す。
- Functions 側では、`app.setup({ enableHttpStream: true })` で HTTP ストリーミングを有効にする。そのうえで、本文を `ReadableStream`（または Node.js の `Readable`）で返す。
- SWA の `/api` には約 45 秒の要求時間の上限がある。長時間つなぎっぱなしにする用途（通知の購読など）には向かない。生成 AI のコメントのように、短時間で終わる応答を逐次表示する用途に限って使う。
- この挙動（完全一致のときだけ逐次転送）は公式資料に書かれていない。実測で確かめたもので、将来変わる可能性がある。

## 必要な設定

### API（Functions）

| 項目                        | 内容                                                                                                                                                                             | このリポジトリでの場所                               |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| HTTP ストリーミングの有効化 | ハンドラーを登録する前に `app.setup({ enableHttpStream: true })` を呼ぶ                                                                                                          | `src/api/src/index.ts`                               |
| ライブラリのバージョン      | `@azure/functions` 4.3.0 以降（このリポジトリは 4.17.1）                                                                                                                         | `src/api/package.json`                               |
| 応答ヘッダー                | 次の 3 つだけにする（charset、`no-transform`、`x-content-type-options` は付けない）<br>`content-type: text/event-stream`<br>`cache-control: no-cache`<br>`x-accel-buffering: no` | `src/api/src/lib/sse.ts` の `SSE_HEADERS`            |
| 本文                        | `ReadableStream<Uint8Array>` を返し、イベントごとに `enqueue` する                                                                                                               | `src/api/src/functions/review.ts` の `toEventStream` |
| 文字コード                  | SSE は仕様上 UTF-8 固定なので、charset がなくても文字化けしない                                                                                                                  | —                                                    |

### クライアント（ブラウザー）

- `EventSource` は GET しか送れない。判定 API は POST で本文を送るので、`fetch` の `response.body.getReader()` で読み、自前でイベントに分ける（`src/frontend/src/api/sse.ts`）。
- チャンクの境目は、イベントの途中、UTF-8 の文字の途中、`\r\n` の途中のどこにでも来る。`TextDecoder` の `stream: true` でデコードし、バッファーにためて空行で区切る。
- コメント行（`:` で始まる行）は、SSE の仕様どおり読み飛ばす。

## 制約と注意点

| 制約                            | 内容                                                                                                                | 対応                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| Content-Type の完全一致         | `text/event-stream` 以外（charset 付きや `text/plain`）は、応答全体がためられて Brotli（`br`）で圧縮される          | ヘッダーを `SSE_HEADERS` の 3 つに固定し、テストで値を確認している                                              |
| 圧縮                            | ためられた応答は、`cache-control: no-transform` を付けても Brotli で圧縮された。完全一致の SSE では圧縮されなかった | 圧縮の有無で逐次転送が決まるわけではない（`identity` でも、charset 付きはためられた）                           |
| 要求時間の上限                  | SWA の `/api` は約 45 秒                                                                                            | Foundry 呼び出しのタイムアウトを 40 秒にし、出力を短くしている（ADR 0005 の決定 4）                             |
| 応答後の状態コード              | 本文を送り始めた後は、状態コードを変えられない                                                                      | 最初の差分の前の失敗は JSON のエラー（400・404・502）で返す。途中の失敗は `error` イベントで知らせる            |
| 切れたことの検出                | `done` も `error` も来ないまま本文が終わることがある                                                                | クライアントは「途中で切れた」としてエラーにする                                                                |
| クライアントの切断              | Node.js v4 には、C# の `CancellationToken` に相当するものがない（shibayan 氏の記事）                                | `ReadableStream` の `cancel` で Foundry の反復を止めているが、ホストが切断を伝えるかは未確認                    |
| マネージド Functions の制約     | HTTP トリガーのみ。プランやランタイムの機能も選べない                                                               | 長時間の接続や別のトリガーが必要なら、独立した Functions アプリを使い、SWA の `/api` を経由しない構成を検討する |
| 独立した Functions アプリの上限 | HTTP 応答までの上限は 230 秒（公式資料）                                                                            | 今回は使っていない                                                                                              |
| 文書化されていない挙動          | 完全一致の条件は SWA の公式資料にない                                                                               | 挙動が変わった場合に備えて、下の再確認の方法を残す                                                              |

## 測定結果（PR #16 のプレビュー環境、2026-11-19）

測定には検証用エンドポイント `GET /api/diag/stream` を使った。1 秒ごとに 1 件、計 5 件を返す。ブラウザーのコンソールから `fetch` で読み、チャンクを受け取るたびに経過時間とバイト数を記録した。

### 1 回目: 以前のヘッダー（`text/event-stream; charset=utf-8` など）

| ケース                             | content-encoding | 受信                  |
| ---------------------------------- | ---------------- | --------------------- |
| `text/plain`                       | br               | 4114 ms に 1 チャンク |
| SSE（パディング 2 KB）             | br               | 5116 ms に 1 チャンク |
| SSE + パディング 16 KB             | br               | 5087 ms に 1 チャンク |
| SSE + パディング 64 KB             | br               | 5085 ms に 1 チャンク |
| SSE + `content-encoding: identity` | identity         | 5088 ms に 1 チャンク |
| SSE + Node.js の `Readable.from`   | br               | 5088 ms に 1 チャンク |

パディングの量、圧縮、ストリームの実装は、どれも原因ではなかった。

### 2 回目: Content-Type を `text/event-stream` ちょうどにした

| ケース                                 | 受け取った content-type            | content-encoding | 受信                                                    |
| -------------------------------------- | ---------------------------------- | ---------------- | ------------------------------------------------------- |
| SSE（新しいヘッダー、パディング 2 KB） | `text/event-stream`                | なし             | 101 / 1100 / 2103 / 3103 / 4117 / 5103 ms の 6 チャンク |
| SSE（以前のヘッダー）                  | `text/event-stream; charset=utf-8` | br               | 5082 ms に 1 チャンク                                   |
| SSE（新しいヘッダー、パディングなし）  | `text/event-stream`                | なし             | 79 / 1086 / 2085 / 3085 / 4085 / 5086 ms の 6 チャンク  |
| SSE（新しいヘッダー）+ `identity`      | `text/event-stream`                | identity         | 96 / 1231 / 2091 / 3090 / 4090 / 5085 ms の 6 チャンク  |

Content-Type が完全に一致したときだけ逐次届いた。パディングは不要だった。

## 未確認の事項

- 提案フォームの判定コメントが、プレビュー環境で少しずつ表示されること（PR #16 の確認項目）。
- 45 秒を超えたときの切れ方（切断されるのか、まとめて返るのか）。
- ローカル（`func start` と Vite の dev server のプロキシ）で逐次届くこと。開発環境に Functions Core Tools がなかったため、試していない。
- クライアントが途中で切断したときに、ホストがストリームを止めるか。

## 挙動が変わったときの再確認

検証用エンドポイントは PR #16 で削除した。再確認するときは、PR #16 のブランチの履歴から次の 2 つのコミットの `src/api/src/functions/diag-stream.ts` を戻し、プレビュー環境で測る。

- `fefb132`: クエリで条件を切り替える版
- `3b92831`: `legacy=1` を追加した版

コミットの ID は rebase 後のもの。

```js
const cases = ['?format=sse', '?format=sse&legacy=1'];
for (const q of cases) {
  const t0 = performance.now();
  const res = await fetch('/api/diag/stream' + q, { cache: 'no-store' });
  const times = [];
  const reader = res.body.getReader();
  for (;;) {
    const { done, value } = await reader.read();
    if (done) break;
    times.push(`${Math.round(performance.now() - t0)}ms/${value.length}B`);
  }
  console.log(
    q,
    res.headers.get('content-type'),
    res.headers.get('content-encoding') ?? '-',
    times.join(' '),
  );
}
```

チャンクが約 1000 ms 間隔で並べば、逐次届いている。1 チャンクにまとまれば、ためられている。

## 参考資料

### 公式資料

- Azure Functions（Node.js）の HTTP ストリーミング: https://learn.microsoft.com/en-us/azure/azure-functions/node-http-stream
- SWA の API の概要（`/api` の制約、45 秒の上限など）: https://learn.microsoft.com/en-us/azure/static-web-apps/apis-overview/
- SWA のマネージド Functions と独立した Functions アプリの比較: https://learn.microsoft.com/en-us/azure/static-web-apps/apis-functions
- Azure Functions のホスティングと上限（HTTP 応答の 230 秒など）: https://learn.microsoft.com/en-us/azure/azure-functions/functions-scale

### 事例・記事

- Content-Type の完全一致で解決した事例（FastAPI、Azure Web App）: https://stackoverflow.com/questions/78584265/fastapi-sse-working-locally-but-not-in-azure-web-app
  - `Content-Type: text/event-stream`、`Cache-Control: no-cache`、`X-Accel-Buffering: no` で逐次届いたとの報告。今回の対策の根拠。
- Azure OpenAI のストリームを Azure Functions（C#）で中継する例: https://zenn.dev/microsoft/articles/azure-openai-stream-azure-function
  - `text/event-stream` で書き込み、チャンクごとに `FlushAsync` する。確認はローカルのみ。isolated モードでは順番が前後する現象があり、遅延を入れて回避している。
- Azure Functions Node.js v4 の HTTP ストリーミングと SSE（shibayan 氏）: https://blog.shibayan.jp/entry/2024/03/08/162404
  - `enableHttpStream` と `text/event-stream` の例。確認はローカルのみ。SWA のマネージド Functions でも使える可能性に触れているが、試してはいない。切断を検出する手段がないこと、長時間の接続はロードバランサーのタイムアウトに注意が必要なことにも触れている。

公式資料の URL のうち、Functions の HTTP ストリーミング、SWA の API の概要、マネージド Functions との比較、ホスティングと上限の 4 件は、調査の過程でユーザーから共有されたもの。記載内容は共有時の要約に基づいており、このメモの作成時に本文を読み直してはいない。
