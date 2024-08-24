# TS、JS 初心者向けに TS 初心者がよく使う配列操作の高階関数をまとめてみた

# はじめに

この記事は最近業務で TS をいじりはじめた TS 初心者の私がよく忘れる配列操作のコールバック関数をまとめてみたものです
自分への備忘録の側面が大きいですが皆さんの役に立てればと思います

# コールバック関数と高階関数

各関数の説明に入る前にコールバック関数と高階関数について説明しておきます。まず高階関数とは`関数を引数に取る関数`です。そしてコールバック関数は`高階関数に引数として取られる関数`のことです。
例えば以下の例の場合は`doSomething` 関数を呼び出す際に`callbackFunction`が引数として与えられています。ですので
`doSomething` が高階関数、`callbackFunction`がコールバック関数になります。
実際はコールバック関数はアロー関数で直書きして渡してあげるケースが多いと思います。

```TS
   function doSomething(callback: (message: string) => void) {
     const message = "Hello, Callback!";
     callback(message);
   }

   callbackFunction((msg) => {
     console.log(msg);
   });

   doSomething(callbackFunction)
   // "Hello, Callback!"
```

# 各配列に対して操作する高階関数

## forEach

for of と似たように、配列の各要素に対して反復処理ができるコールバック関数です。
あくまで高階関数なので break や continue はできないので注意しましょう。
また、for of と比べると非同期処理もできません。

```typescript
const numbers = [1, 2, 3];
numbers.forEach((value, index) => {
  console.log(`Index: ${index}, Value: ${value}`);
});
```

一見すると for of の劣化版にしかみえませんが、実はこれが大きなメリットでもあります。
というのも forEach のメリットは**各要素に対して処理をすることを明示的にできる**ことにあるからです。
forEach を見た開発者はその中に break や continue などループの制御がないことが一眼でわかるので、
他の**各要素に対して必ず処理をする**ことが伝わりやすくなります。

```TS
//for ofの場合
//breakやcontinueを警戒しなくてはならない
const numbers = [1, 2, 3, 4, 5];

for (const num of numbers) {
  if (num === 3) console.log(num*2);
  console.log(num); // 1, 2, 4, 5
}

//forEachの場合
//breakやcontinueはないので必ず全ての配列に対して同じ処理がされる
numbers.forEach((num) => {
  if (num === 3) console.log(num*2);
  console.log(num); // 1, 2, 4, 5
});

```

## map

配列の各要素に対して処理を行って、新しい配列を返す関数です。
ORM で取得した結果をそのまま返すのではなく、加工して返したい場合には結構便利です。
似た挙動をする**forEach は返り値として新しい配列を返すかどうか**が大きな違いなので、その観点で使い分けましょう

```typescript
const numbers = [1, 2, 3];
const doubled = numbers.map((value) => value * 2);
console.log(doubled); // [2, 4, 6]
```

## filter

その名の通り、コールバック関数を適用してフィルタリングした関数を返します。
例えば ORM で取得した結果から何かを除外したい場合とかには便利だったりします(もちろんなるべくクエリで除外できるのが理想ですが)

```typescript
const numbers = [1, 2, 3, 4];
const evens = numbers.filter((value) => value % 2 === 0);
console.log(evens); // [2, 4]
```

## reduce

配列を単一の値にまとめたい場合に使えます。
以下の例だと配列をその合計値に変換しています。

```typescript
const numbers = [1, 2, 3, 4];
const sum = numbers.reduce((accumulator, value) => accumulator + value, 0);
console.log(sum); // 10
```

# 配列の特定要素を検知する高階関数

## find

配列内の条件に一致する要素を探す関数です。
返り値はコールバック関数が true を返す最初の要素になります。
以下の例の場合は` { id: 3, name: "Bob" }`ではなく` { id: 2, name: "Bob" }`が返却されます

```typescript
const users = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" },
  { id: 3, name: "Bob" },
];
const user = users.find((user) => user.name === "Bob");
console.log(user); // { id: 2, name: "Bob" }
```

## some

配列内の要素が少なくとも 1 つ条件を満たしているかをチェックします。
find との違いは**boolean を返すか条件を満たす最初の要素を返すか**です。
用途に応じて使い分けましょう。

```typescript
const numbers = [1, 2, 3, 4];
const hasEven = numbers.some((value) => value % 2 === 0);
console.log(hasEven); // true
```

## every

配列内のすべての要素が条件を満たしているかをチェックします。
得られた配列が要件を満たしているのか確認したいときに便利です。
返り値はこちらも boolean です

```typescript
const numbers = [2, 4, 6];
const allEven = numbers.every((value) => value % 2 === 0);
console.log(allEven); // true
```

# さいごに

これが足りない！とかこれが間違ってる！とか色々あると思います。そういったものは**優しく**ご指摘いただけると助かります。
あと自分はバックエンドエンジニアなのでフロントでよく使うものは欠けていると思います。それは申し訳ないです。
少しでもこの記事が誰かの助けになればと思います。
