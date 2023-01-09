# Webpack
## 環境構築

install時にglobalを付与するとprojectではなくPC全体にインストール可
※globalはpathが通る。localの場合は./node_modules/bin以下。

```
npm install 
    webpack
    webpack-cli
    sass
    sass-loader
    css-loader
    style-loader
```

あれば便利なもの
```
VS-Code
    Live Server
```

* 注意点  
  * Webpackはバージョン更新による記載ルールの変化が大きい。

---
## 参考
https://zenn.dev/antez/articles/58307946cf4f3e  
https://zenn.dev/antez/articles/638382faa06bd7  
https://zenn.dev/hrkmtsmt/articles/93653309e2a13d  

---
## 基本のコマンドと定義方法

### バンドルのコマンド
```
npx webpack --mode development
npx webpack --mode production
```

dstフォルダが生成される
development は圧縮されない

```
npx webpack --mode development --devtool none
```

など、オプション指定でバンドル方法を変更する

---
### 定義ファイル

オプション定義をwebpack.config.jsで定義する。  
デフォルトファイル名(webpack.config.js)の場合はオプション指定不要。

```
npx webpack --mode development --devtool none --config ./<configname>.js
```
↓
webpack.config.js
```js
module.exports = {
    mode: 'development',
    devtool: 'none'
}
```
`devtool: 'none'`はwebpack5以降は`devtool`を指定しないことで実現可能。

---
#### entry point 
デフォルトはsrc/index.js
```js
module.exports = {
    ～略～
    entry: './src/index.js'
}
```
複数entry pointを纏める場合
```js
module.exports = {
    ～略～
    entry: ['./src/index.js', './src/index2.js']
}
```
複数entry pointで個別にdstに出力する場合
(SPAではない場合などで複数entry pointがある場合を想定)
```js
module.exports = {
    ～略～
    entry: {
        app: './src/index.js', 
        sub: './src/index2.js'
    }
}
```

---
#### output
バンドルの出力先、ファイル名を変更する
```js
const path = require('path')
module.exports = {
    mode: 'development',
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'public'),
        filename: 'bundle.js'
    }
}

```
```
./src/index.js  // entry
./src/index.html
./webpack.config.js
```
↓
```
./public/bundle.js
```

---
#### loader
loader = js以外(sass / css など)をバンドルするためのモジュールを指す。   
各種ファイルに対して変換用のloaderを通してどのようにjsに統合するかのルールを定める。  
```js
module.exports = {
    module:{
        rules:[
            {
                test: /\.scss$/, // load対象を指定: 終端(=$)が.scssとなるファイル
                use: [
                    // 下からloaderは実行される
                    'style-loader', // 3. cssをhtmlにstyle情報を注入
                    'css-loader',   // 2. cssをjsにバンドルする
                    'sass-loader'   // 1. sass情報をcssにコンパイルする
                ]
            }
        ]
    }
}
```




