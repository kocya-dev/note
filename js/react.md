# React
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



