# Vite

https://ja.vitejs.dev/guide/#%E6%A6%82%E8%A6%81

## 環境構築

npm create vite@latest

1. プロジェクト名を決定
2. フレームワークの選択
3. 使用言語の選択 (JS/TS/JS+SWC/TS+SWC)

## 実行

1. 作成したプロジェクトディレクトリに移動
2. npm install (依存モジュールをインストール)
3. npm run dev
4. ブラウザで表示されるアドレスにアクセス

その他
* プロジェクトのビルド
  * npm run build
* ビルドした状態のプレビュー
  * npm run preview

https://chaika.hatenablog.com/entry/2022/05/13/083000
https://ja.vitejs.dev/guide/build.html

npm i 後のnode_moduldesの状態
```
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        2023/03/21     16:02                .bin
d-----        2023/03/21     16:02                @esbuild
d-----        2023/03/21     16:02                @swc
d-----        2023/03/21     16:02                @types
d-----        2023/03/21     16:02                @vitejs
d-----        2023/03/21     16:02                csstype
d-----        2023/03/21     16:02                esbuild
d-----        2023/03/21     16:02                function-bind
d-----        2023/03/21     16:02                has
d-----        2023/03/21     16:02                is-core-module
d-----        2023/03/21     16:02                js-tokens
d-----        2023/03/21     16:02                loose-envify
d-----        2023/03/21     16:02                nanoid
d-----        2023/03/21     16:02                path-parse
d-----        2023/03/21     16:02                picocolors
d-----        2023/03/21     16:02                postcss
d-----        2023/03/21     16:02                react
d-----        2023/03/21     16:02                react-dom
d-----        2023/03/21     16:02                resolve
d-----        2023/03/21     16:02                rollup
d-----        2023/03/21     16:02                scheduler
d-----        2023/03/21     16:02                source-map-js
d-----        2023/03/21     16:02                supports-preserve-symlinks-flag
d-----        2023/03/21     16:02                typescript
d-----        2023/03/21     16:02                vite
-a----        2023/03/21     16:02          13617 .package-lock.json
```

