# AGENTS.md

## このリポジトリについて

個人用の技術メモや調査資料を Markdown で管理し、Zensical で静的サイトに変換して GitHub Pages で公開するリポジトリです。通常の編集対象は `docs/` 以下の Markdown と関連画像です。

## ディレクトリと設定

- `docs/`: 公開する文書。分野ごとに `aws/`、`azure/`、`web/`、`tech/`、`tools/`、`process/`、`windows/` などに整理されています。
- `docs/index.md`: サイトのトップページ。
- `mkdocs.yml`: Zensical が読み込むサイト名、テーマ、Markdown 拡張などの設定。明示的な `nav` はなく、既定のファイル順でナビゲーションが生成されます。
- `requirements.txt`: Zensical と Markdown 拡張の依存関係。
- `.github/workflows/deploy_zensical.yml`: Zensical のビルドと Pages 公開を行う GitHub Actions。
- `site/`: Zensical の生成物。原則として直接編集せず、ビルドで再生成します。
- `sample/`: サンプル用の静的ファイル。現在の Pages ワークフローの公開対象は `site/` です。

## 文書を編集するとき

- 新しい公開文書は、内容に合う既存の `docs/` 配下の分野ディレクトリに Markdown (`.md`) で追加します。適切な場所がなければ、内容に沿ったディレクトリを作成します。
- 見出しは内容を説明する階層にし、ページ冒頭に主題が分かる見出しを置きます。既存資料の形式を尊重し、不要な一括整形や無関係な書き換えを避けます。
- 既存ページや画像へのリンクは、ファイル移動・改名後も解決できる相対パスを使います。画像などの添付ファイルは関連する文書の近くに置きます。
- GitHub Flavored Markdown 固有機能に依存する場合は、Zensical の Markdown 拡張で処理できるか確認します。利用可能な拡張には admonition、タブ、折りたたみ、脚注、定義リスト、Mermaid コードフェンスなどがあります。
- 図は Mermaid を利用できます。Mermaid 図は `mermaid` コードフェンスで記述します。
- `mkdocs.yml` の `site_url` は GitHub Pages の公開 URL です。公開先を変更するときは設定も更新してください。

## 開発・ビルド

Python 環境を用意して、リポジトリのルートで次を実行します。

```sh
python -m pip install -r requirements.txt
zensical serve
```

ローカルで CI と同じクリーンビルドを行う場合:

```sh
zensical build --clean
```

GitHub Actions は Ubuntu 上で Python 3.11 をセットアップし、`requirements.txt` をインストールして Zensical でビルドします。ローカルと CI の Python バージョンが異なる場合、差異がビルド結果に影響する可能性があります。

## 公開ワークフロー

- `.github/workflows/deploy_zensical.yml` は `main` ブランチへの push と `workflow_dispatch` で実行されます。
- ビルド成功後、`site/` を Pages artifact としてアップロードし、GitHub Pages にデプロイします。
- 公開手順や Actions の権限、Python バージョンを変更するときは、このワークフローと Zensical 設定の両方を確認します。npm / `gh-pages` の設定はリポジトリにありますが、現行の Pages ワークフローでは使用していません。
