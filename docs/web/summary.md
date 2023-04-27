# Web開発

## モダンWeb開発環境

### ここ数年の開発のトレンドについて  
https://www.nri-digital.jp/tech/20220216-8262/  
https://wakka-inc.com/blog/1937/  
https://qiita.com/simezi9/items/d5ace310958be3b715aa  
https://ncdc.co.jp/columns/7187/  
https://engineering.linecorp.com/ja/blog/uit-baseline-for-front-end-development-in-2022/  

### JavaScriptの標準について
https://www.tohoho-web.com/js/what.htm

* ECMAScript5.1th Edition(ES5.1)
  * IE11でもほぼ動作する
* ECMAScript2015(ES6)
  * IE11はほぼ不可。Chrome/Edge/FireFox/Safariはほぼ100%。
* ECMAScript2016(ES7)
* ECMAScript2017(ES8)
  * FireFox15.2未満はOK
* ECMAScript2018(ES9)
  * FireFox15.2は一部未対応
* ECMAScript2019(ES10)
* ECMAScript2020(ES11)
* ECMAScript2021(ES12)
  * Chrome/Edgeはほぼ100%。
* ECMAScript2022(ES13)
  * Chrome/EdgeはRegExpのみ未対応部あり。
* ECMAScript2023(ES14)

各ブラウザの対応状況  
https://kangax.github.io/compat-table/es6/    

ES2015(ES6)に合わせる情報があるが、何がメリットか？慣例か？

### 環境概略

* js
  * typescript
    * --(eslint=静的解析/prettier=formatter)-->typescript
      * --(Babel等でtranspile)--> js(ES6)
        * --(webpackなどのbundler)--> 単一js
* css
  * sass 
    * --(webpackなどのbundler)--> css
      * --(webpackなどのbundler)--> js

---

## その他ツールチェーン

---
### Next.js
Reactを用いたSSR機能を提供するフレームワーク。  
描画は早いが、表示データが頻繁に更新されるアプリケーションには不向き。

---
### Redux
アプリケーションの状態管理を担うライブラリ。  
https://qiita.com/keitakn/items/7433c89ce52073e861a1  
https://qiita.com/jabba/items/5ebea8d19809a4e0f8c6  
https://qiita.com/kitagawamac/items/49a1f03445b19cf407b7  
https://qiita.com/Statham/items/9f830dac5fee00deb9df  

###  

---
## アーキテクチャ選定
---
### SPA vs MPA

https://deliv.tech/2020/11/10/mpa-vs-spa/

---
## 環境構築例

### その1
Typescript + React + Webpack + Express  
ESLint / Prettier / Jest / React Testing Library / Chakra UI  
https://none-if-none-else-none.hatenablog.com/entry/2021/02/27/120000
https://none-if-none-else-none.hatenablog.com/entry/2021/03/06/120000

* Express
  * サーバーサイドJavascript / Node.js のWebアプリケーションフレームワーク
* ESLint
  * Node.jsで動作するlinter(静的コード分析)
* Prettie
  * フォーマッタ
* Jest
  * 単体テストライブラリ。Typescript環境ではTS-Jestから利用する
* React Testing Library
  * ブラウザ環境を疑似的に再現するテストツール 
* Chakra UI
  * UIフレームワーク(https://chakra-ui.com/)

### その2

Vite + TypeScript + React
https://ics.media/entry/210708/#contents-anchor-react  
https://chaika.hatenablog.com/entry/2022/05/13/083000  

