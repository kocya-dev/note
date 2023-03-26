# React

# 環境構築
```
npx create-react-app <project name>
or
npm init react-app <project name>
```
```
npm i create-react-app
```
することで、npxを使用せずとも済む。
```
create-react-app <project name>
```
※create-react-appはwebpackの設定も自動で行なわれる。    


## VSCode拡張
ES7+ React/Redux/React-Native snippets
VSCode React Refactor


## npm / yarn について  
https://qiita.com/mzmz__02/items/4ba43b69c8878a9ca99e  
https://qiita.com/e99h2121/items/7e38e592dc45b7c0407d  



## 再レンダリング
コンポーネントのproperty、ステート変化時、親コンポーネントの再レンダリングによって対象要素の再レンダリングがおこなわれる。

## import / export

```js
// コンポーネント側
const hoge = () => { xxxx };
export default hoge;    // コンポーネントで1つしか宣言できない

// 使用する側
import hoge from "xxxx";
```
```js
// コンポーネント側
export const hoge = () => { xxxx };

// 使用する側
import { hoge } from "xxxx";
```



