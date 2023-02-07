# 基本
## 分割代入

```js
const profile = {
    name: '名前', 
    age: 20
};

const {name, age} = profile;
const mssage = 'name is ${name}, age = ${age}';
```

```js
const profile = ['名前', 20];
const [name, age] = profile;
const mssage = 'name is ${name}, age = ${age}';
```

## デフォルト引き数

```js
const sayHello(name = 'ゲスト') => console.log('hello, ${name}!');
sayHello('bob');    // hello, bob!
sayHello();         // hello, ゲスト!
```

## スプレッド構文(配列展開)

```js
const ar = [1, 2];
console.log(ar);    // [1,2]
console.log(...ar); // 1,2
```
```js
const ar = [1, 2, 3, 4, 5];
const [v1, v2, ...ar2] = ar;    // 1,2,[3,4,5]
```
```js
const ar = [1, 2];
const ar2 = [...ar];    // ar2はarのコピー [1,2]
const ar3 = ar;         // ar3はarと同じ配列を指す
```

## map/filter
```js
const ar = [1, 2, 3];
ar.map((val, index)=>console.log('${index}番目 = ${val}')); 
ar.map((val)=>console.log(val)); // indexは省略可能
const ar2 = ar.map((val)=>val*2); // mapの本来の機能は射影 ar2 = [2, 4, 6]
```
```js
const ar = [1, 2, 3,4];
ar.filter((val)=>(val%2)==0); // 偶数のみの配列を生成
```

