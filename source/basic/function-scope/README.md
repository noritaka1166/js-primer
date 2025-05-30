---
author: azu
description: "スコープという変数などを参照できる範囲を決める概念を紹介します。ブロックスコープや関数スコープなどがどのような働きをしているのかや複数のスコープが重なったときにどのように変数の参照先が決まるのかを紹介します。また、スコープに関係する動作としてクロージャーという性質を紹介します。"
sponsors: []
---

# 関数とスコープ {#function-and-scope}

定義された関数はそれぞれのスコープを持っています。スコープとは変数や関数の引数などを参照できる範囲を決めるものです。
JavaScriptでは、新しい関数を定義するとその関数にひもづけられた新しいスコープが作成されます。関数を定義するということは処理をまとめるというだけではなく、変数が有効な範囲を決める新しいスコープを作っていると言えます。

スコープの仕組みを理解することは関数をより深く理解することにつながります。なぜなら関数とスコープは密接な関係を持っているからです。
この章では関数とスコープの関係を中心に、スコープとはどのような働きをしていて、スコープ内では変数の名前から取得する値がどのように決まるかを見ていきます。

JavaScriptのスコープは、ES2015において直感的に理解しやすい仕組みが整備されました。
基本的にはES2015以降の仕組みを理解していればコードを書く場合には問題ありません。

しかし、既存のコードを理解するためには、ES2015より前に決められた古い仕組みについても知る必要があります。
なぜなら、既存のコードは古い仕組みを使って書かれていることもあるためです。
また、JavaScriptでは古い仕組みと新しい仕組みを混在して書くことができます。
古い仕組みによるスコープは直感的でない挙動も多いため、古い仕組みについても補足していきます。

## スコープとは {#what-is-scope}

スコープとは変数の名前や関数などの参照できる範囲を決めるものです。
スコープの中で定義された変数はスコープの内側でのみ参照でき、スコープの外側からは参照できません。

身近なスコープの例として関数によるスコープを見ていきます。

次のコードでは、`fn`関数のブロック（`{`と`}`）内で変数`x`を定義しています。
この変数`x`は`fn`関数のスコープに定義されているため、`fn`関数の内側では参照できます。
一方、`fn`関数の外側から変数`x`は参照できないため`ReferenceError`が発生します。

{{book.console}}
```js
function fn() {
    const x = 1;
    // fn関数のスコープ内から`x`は参照できる
    console.log(x); // => 1
}
fn();
// fn関数のスコープ外から`x`は参照できないためエラー
console.log(x); // => ReferenceError: x is not defined
```

このコードを見てわかるように、変数`x`は`fn`関数のスコープにひもづけて定義されます。
そのため、変数`x`は`fn`関数のスコープ内でのみ参照できます。

関数は**仮引数**を持てますが、仮引数は関数のスコープにひもづけて定義されます。
そのため、仮引数はその関数の中でのみ参照が可能で、関数の外からは参照できません。

<!-- Note: 関数の仮引数はvarで宣言されたものと同じ扱い

関数の宣言
- Set F.[[FormalParameters]] to ParameterList.
- https://tc39.es/ecma262/#sec-functioninitialize
- function objectの[[Environment]]ではなく、[[FormalParameters]]に代入される
- 仮引数は関数のスコープではなくfunction objectに紐づく

関数の初期化処理
- https://tc39.es/ecma262/#sec-functiondeclarationinstantiation
- [[FormalParameters]]は初期化時に`varEnv`に対して代入される
- つまり、仮引数はvarで宣言しているのと同じになる
- スコープという意味では内側のみから参照でき、外からは参照できないという点で同じ

 -->

{{book.console}}
```js
function fn(arg) {
    // fn関数のスコープ内から仮引数`arg`は参照できる
    console.log(arg); // => 1
}
fn(1);
// fn関数のスコープ外から`arg`は参照できないためエラー
console.log(arg); // => ReferenceError: arg is not defined
```

このような、関数によるスコープのことを**関数スコープ**と呼びます。

「[変数と宣言][]」の章にて、`let`や`const`は同じスコープ内に同じ名前の変数を二重に定義できないという話をしました。
これは、各スコープには同じ名前の変数は1つしか宣言できないためです（`var`による変数宣言と`function`による関数宣言は例外的に可能です）。

[import, identifier-duplicated-invalid](./src/identifier-duplicated-invalid.js)

一方、スコープが異なれば同じ名前で変数を宣言できます。
次のコードでは、`fnA`関数と`fnB`関数という異なるスコープで、それぞれ変数`x`を定義できていることがわかります。

{{book.console}}
```js
// 異なる関数のスコープには同じ"x"を定義できる
function fnA() {
    let x;
}
function fnB() {
    let x;
}
```

このように、スコープが異なれば同じ名前の変数を定義できます。
スコープの仕組みがないと、グローバルな空間内で一意な変数名を考える必要があります。
スコープがあることで同じ名前の変数をスコープごとに定義できるため、スコープの役割は重要です。

## ブロックスコープ {#block-scope}

`{`と`}`で囲んだ範囲をブロックと呼びます（「[文と式][]」の章を参照）。
ブロックもスコープを作成します。
ブロック内で宣言された変数は、スコープ内でのみ参照でき、スコープの外側からは参照できません。

{{book.console}}
```js
// ブロック内で定義した変数はスコープ内でのみ参照できる
{
    const x = 1;
    console.log(x); // => 1
}
// スコープの外から`x`を参照できないためエラー
console.log(x); // => ReferenceError: x is not defined
```

ブロックによるスコープのことを**ブロックスコープ**と呼びます。

<!-- Notes: ブロックスコープと仕様

- ブロック`{}`は新しい[NewDeclarativeEnvironment](https://tc39.es/ecma262/#sec-newdeclarativeenvironment "NewDeclarativeEnvironment")を作成する
- つまり、新しいLexicalEnvironmentというスコープを作成している
- https://tc39.es/ecma262/#sec-block-runtime-semantics-evaluation
- そして評価する際に、そのブロック内に宣言されている変数をスコープに対してひもづけている
- https://tc39.es/ecma262/#sec-blockdeclarationinstantiation

 -->

if文やwhile文などもブロックスコープを作成します。
単独のブロックと同じく、ブロックの中で宣言した変数は外から参照できません。

{{book.console}}
```js
// if文のブロック内で定義した変数はブロックスコープの中でのみ参照できる
if (true) {
    const x = "inner";
    console.log(x); // => "inner"
}
console.log(x); // => ReferenceError: x is not defined
```

for文は、ループごとに新しいブロックスコープを作成します。
このことは「各スコープには同じ名前の変数は1つしか宣言できない」のルールを考えてみるとわかりやすいです。
次のコードでは、ループごとに`const`で`element`変数を定義していますが、エラーなく定義できています。
これは、ループごとに別々のブロックスコープが作成され、変数の宣言もそれぞれ別々のスコープで行われるためです。

{{book.console}}
```js
const array = [1, 2, 3, 4, 5];
// ループごとに新しいブロックスコープを作成する
for (const element of array) {
    // forのブロックスコープの中でのみ`element`を参照できる
    console.log(element);
}
// ループの外からはブロックスコープ内の変数は参照できない
console.log(element); // => ReferenceError: element is not defined
```

<!-- Note: forとブロックスコープの仕様

- 仕様ではforをIterateする際に`CreatePerIterationEnvironment`で新しい NewDeclarativeEnvironmentを作成している
- [9. Variables and scoping](http://exploringjs.com/es6/ch_variables.html#sec_let-const-loop-heads)
- [ECMAScript 2015 Language Specification – ECMA-262 6th Edition](http://www.ecma-international.org/ecma-262/6.0/#sec-createperiterationenvironment)

 -->

## スコープチェーン {#scope-chain}

関数やブロックはネスト（入れ子）して書けますが、同様にスコープもネストできます。
次のコードではブロックの中にブロックを書いています。
このとき外側のブロックスコープのことを`OUTER`、内側のブロックスコープのことを`INNER`と呼ぶことにします。

```js
{
    // OUTERブロックスコープ
    {
        // INNERブロックスコープ
    }
}
```

スコープがネストしている場合に、内側のスコープから外側のスコープにある変数を参照できます。
次のコードでは、内側のINNERブロックスコープから外側のOUTERブロックスコープに定義されている変数`x`を参照できます。
これは、ブロックスコープに限らず関数スコープでも同様です。

{{book.console}}
```js
{
    // OUTERブロックスコープ
    const x = "x";
    {
        // INNERブロックスコープからOUTERブロックスコープの変数を参照できる
        console.log(x); // => "x"
    }
}
```

変数を参照する際には、現在のスコープ（変数を参照する式が書かれているスコープ）から外側のスコープへと順番に変数が定義されているかを確認します。
上記のコードでは、内側のINNERブロックスコープには変数`x`はありませんが、外側のOUTERブロックスコープに変数`x`が定義されているため参照できます。
つまり、次のようなステップで参照したい変数を探索しています。

1. INNERブロックスコープに変数`x`があるかを確認 => ない
2. ひとつ外側のOUTERブロックスコープに変数`x`があるかを確認 => ある

一方、現在のスコープも含め、外側のどのスコープにも該当する変数が定義されていない場合は、`ReferenceError`の例外が発生します。
次の例では、どのスコープにも存在しない`xyz`を参照しているため、`ReferenceError`の例外が発生します。

{{book.console}}
```js
{
    // OUTERブロックスコープ
    {
        // INNERブロックスコープ
        console.log(xyz); // => ReferenceError: xyz is not defined
    }
}
```

このときも、現在のスコープ（変数を参照する式が書かれているスコープ）から外側のスコープへと順番に変数が定義されているかを確認しています。
しかし、どのスコープにも変数`xyz`は定義されていないため、`ReferenceError`の例外が発生していました。
つまり次のようなステップで参照したい変数を探索しています。

1. INNERブロックスコープに変数`xyz`があるかを確認 => ない
2. ひとつ外側のOUTERブロックスコープに変数`xyz`があるかを確認 => ない
3. 一番外側のスコープにも変数`xyz`は定義されていない => `ReferenceError`が発生

この内側から外側のスコープへと順番に変数が定義されているか探す仕組みのことを**スコープチェーン**と呼びます。

内側と外側のスコープ両方に同じ名前の変数が定義されている場合もスコープチェーンの仕組みで解決できます。
次のコードでは、内側のINNERブロックスコープと外側のOUTERブロックスコープに同じ名前の変数`x`が定義されています。
スコープチェーンの仕組みにより、現在のスコープに定義されている変数`x`を優先的に参照します。

{{book.console}}
```js
{
    // OUTERブロックスコープ
    const x = "outer";
    {
        // INNERブロックスコープ
        const x = "inner";
        // 現在のスコープ(INNERブロックスコープ)にある`x`を参照する
        console.log(x); // => "inner"
    }
    // 現在のスコープ(OUTERブロックスコープ)にある`x`を参照する
    console.log(x); // => "outer"
}
```

このようにスコープは階層的な構造となっており、変数を参照する際にどの変数が参照できるかはスコープチェーンによって解決されています。

## グローバルスコープ {#global-scope}

今までコードをプログラム直下に書いていましたが、ここにも暗黙的な**グローバルスコープ**（大域スコープ）と呼ばれるスコープが存在します。
グローバルスコープとは名前のとおりもっとも外側にあるスコープで、プログラム実行時に暗黙的に作成されます。

{{book.console}}
```js
// プログラム直下はグローバルスコープ
const x = "x";
console.log(x); // => "x"
```

グローバルスコープで定義した変数は**グローバル変数**と呼ばれ、グローバル変数はあらゆるスコープから参照できる変数となります。
なぜなら、スコープチェーンの仕組みにより、最終的にもっとも外側のグローバルスコープに定義されている変数を参照できるためです。

{{book.console}}
```js
// グローバル変数はどのスコープからも参照できる
const globalVariable = "グローバル";
// ブロックスコープ
{
    // ブロックスコープ内には該当する変数が定義されてない -> 外側のスコープへ
    console.log(globalVariable); // => "グローバル"
}
// 関数スコープ
function fn() {
    // 関数ブロックスコープ内には該当する変数が定義されてない -> 外側のスコープへ
    console.log(globalVariable); // => "グローバル"
}
fn();
```

<!-- textlint-disable preset-ja-technical-writing/sentence-length -->

グローバルスコープには自分で定義したグローバル変数以外に、プログラム実行時に自動的に定義される**ビルトインオブジェクト**があります。

ビルトインオブジェクトには、大きく分けて2種類のものがあります。
1つ目はECMAScript仕様が定義する`undefined`のような変数（「[undefinedはリテラルではない][]」を参照）や`isNaN`のような関数、`Array`や`RegExp`などのコンストラクタ関数です。2つ目は実行環境（ブラウザやNode.jsなど）が定義するオブジェクトで`document`や`module`などがあります。
どちらもグローバルスコープに自動的に定義されているという点で大きな使い分けはないため、この章ではどちらも**ビルトインオブジェクト**と呼ぶことにします。

<!-- Notes: global objectとbuilt-in object

- global object: https://tc39.es/ecma262/#sec-global-object
    - `global`でアクセスできる予定 https://github.com/tc39/proposal-global
    - Arrayなどもglobal objectのプロパティの一種
- builtin object: https://tc39.es/ecma262/#sec-built-in-object
    - ECMAScriptの定義したものと実装が加えたオブジェクトをまとめた用語
- 仕様どおりの定義で言えば、紹介してる`undefined`などはあくまでglobal objectのプロパティ
- `global`を使ってプロパティとしてアクセスする方法は言及していないので省略している

 -->

<!-- textlint-enable preset-ja-technical-writing/sentence-length -->

ビルトインオブジェクトは、プログラム開始時にグローバルスコープへ自動的に定義されているためどのスコープからも参照できます。

{{book.console}}
```js
// ビルトインオブジェクトは実行環境が自動的に定義している
// どこのスコープから参照してもReferenceErrorにはならない
console.log(isNaN); // => isNaN
console.log(Array); // => Array
```

自分で定義したグローバル変数とビルトインオブジェクトでは、グローバル変数が優先して参照されます。
つまり次のようにビルトインオブジェクトと同じ名前の変数を定義すると、定義した変数が参照されます。

{{book.console}}
```js
// "Array"という名前の変数を定義
const Array = 1;
// 自分で定義した変数がビルトインオブジェクトより優先される
console.log(Array); // => 1
```

ビルトインオブジェクトと同じ名前の変数を定義したことにより、ビルトインオブジェクトを参照できなくなります。
このように内側のスコープで外側のスコープと同じ名前の変数を定義することで、外側の変数が参照できなくなることを**変数の隠蔽**（shadowing）と呼びます。

この問題を回避する方法としては、むやみにグローバルスコープへ変数を定義しないことです。グローバルスコープでビルトインオブジェクトと名前が衝突するとすべてのスコープへ影響を与えますが、関数のスコープ内では影響範囲がその関数の中だけにとどまります。

ビルトインオブジェクトと同じ名前を避けることは難しいです。
なぜならビルトインオブジェクトには実行環境（ブラウザやNode.jsなど）がそれぞれ独自に定義したものが多く存在するためです。
関数などを活用して小さなスコープを中心にしてプログラムを書くことで、ビルトインオブジェクトと同じ名前の変数があっても影響範囲を限定できます。

## [コラム] 変数を参照できる範囲を小さくする {#reduce-scope}

グローバル変数に限らず、特定の変数を参照できる範囲を小さくするのはよいことです。
なぜなら、現在のスコープの変数を参照するつもりがグローバル変数を参照したり、その逆も起きることがあるからです。
あらゆる変数がグローバルスコープにあると、どこでその変数が参照されているのかを把握できなくなります。
これを避けるシンプルな考え方は、変数はできるだけ利用するスコープ内に定義するというものです。

次のコードでは、`doHeavyTask`関数の実行時間を計測しようとしています。
`Date.now`静的メソッドは現在の時刻をミリ秒にして返す関数です。
`Date.now`静的メソッドを使った**実行後の時刻**から**実行前の時刻**を引くことで、間に行われた処理の実行時間が得られます。

{{book.console}}
```js
function doHeavyTask() {
    // 計測したい処理
}
const startTime = Date.now();
doHeavyTask();
const endTime = Date.now();
console.log(`実行時間は${endTime - startTime}ミリ秒`);
```

このコードでは、計測処理以外で利用しない`startTime`と`endTime`という変数がグローバルスコープに定義されています。
プログラム全体が短い場合はあまり問題になりませんが、プログラムが長くなっていくにつれ影響の範囲が広がっていきます。
この2つの変数を参照できる範囲を小さくする簡単な方法は、この実行時間を計測する処理を関数にすることです。

{{book.console}}
```js
// 実行時間を計測したい関数をコールバック関数として引数に渡す
const measureTask = (taskFn) => {
    const startTime = Date.now();
    taskFn();
    const endTime = Date.now();
    console.log(`実行時間は${endTime - startTime}ミリ秒`);
};
function doHeavyTask() {
    // 計測したい処理
}
measureTask(doHeavyTask);
```

これにより、`startTime`と`endTime`という変数をグローバルスコープからなくせました。
また、実行時間を計測するという処理を`measureTask`という関数にしたことで再利用できます。

コードの量が増えていくにつれ、人が一度に把握できる量にも限界がやってきます。
そのため、人が一度に把握できる範囲のサイズに処理をまとめていくことが必要です。
この問題を解決するアプローチとして、変数を参照できる範囲を小さくするために、処理を関数にまとめるという手法がよく利用されます。

## 関数スコープとvarの巻き上げ {#hoisting-var}

<!-- textlint-disable eslint -->

変数宣言には`var`、`let`、`const`が利用できます。
「[変数と宣言][]」の章において、「`let`は`var`を改善したバージョン」と紹介したように、`let`は`var`を改善する目的で導入された構文です。`const`は再代入できないという点以外は`let`と同じ動作になります。そのため、`let`が使える場合に`var`を使う理由はありませんが、既存のコードや既存のライブラリなどでは`var`が利用されている場面もあるため、`var`の動作を理解する必要があります。

まず最初に、`let`と`var`で共通する動作を見ていきます。
`let`と`var`どちらも、初期値を指定せずに宣言した変数の評価結果は暗黙的に`undefined`になります。
また、`let`と`var`どちらも、変数宣言をした後に値を代入できます。

次のコードでは、それぞれ初期値を持たない変数を**宣言した後**に参照すると、変数の評価結果は`undefined`となっています。

{{book.console}}
```js
let let_x;
var var_x;
// 宣言後にそれぞれの変数を参照すると`undefined`となる
console.log(let_x); // => undefined
console.log(var_x); // => undefined
// 宣言後に値を代入できる
let_x = "letのx";
var_x = "varのx";
```

次に、`let`と`var`で異なる動作を見ていきます。

`let`では、変数を**宣言する前**にその変数を参照すると`ReferenceError`の例外が発生して参照できません。
次のコードでは、変数を宣言する前に変数`x`を参照したため`ReferenceError`となっています。
エラーメッセージから、変数`x`が存在しないからエラーになっているのではなく、実際に宣言した行より前に参照したためエラーとなっているのがわかります。[^1]

{{book.console}}
```js
console.log(x); // => ReferenceError: can't access lexical declaration `x' before initialization
let x = "letのx";
```

一方`var`では、変数を**宣言する前**にその変数を参照しても`undefined`となります。
次のコードは、変数を宣言する前に参照しているにもかかわらずエラーにはならず、変数`x`の評価結果は`undefined`となります。

{{book.console}}
```js
// var宣言より前に参照してもエラーにならない
console.log(x); // => undefined
var x = "varのx";
```

このように`var`で宣言された変数が宣言前に参照でき、その値が`undefined`となる特殊な動きをしていることがわかります。

この`var`の振る舞いを理解するために、変数宣言が**宣言**と**代入**の2つの部分から構成されていると考えてみましょう。
`var`による変数宣言は、**宣言**部分が暗黙的にもっとも近い関数またはグローバルスコープの先頭に巻き上げられ、**代入**部分はそのままの位置に残るという特殊な動作をします。

この動作により、変数`x`を参照するコードより前に変数`x`の宣言部分が移動し、変数`x`の評価結果は暗黙的に`undefined`となっています。
つまり、先ほどのコードは実際の実行時には、次のように解釈されて実行されていると考えられます。

{{book.console}}
```js
// 解釈されたコード
// スコープの先頭に宣言部分が巻き上げられる
var x;
console.log(x); // => undefined
// 変数への代入はそのままの位置に残る
x = "varのx";
console.log(x); // => "varのx"
```

さらに、`var`変数の宣言の巻き上げは、ブロックスコープを無視してもっとも近い関数またはグローバルスコープに変数をひもづけます。
そのため、次のようにブロック`{}`で`var`による変数宣言を囲んでも、もっとも近い関数スコープである`fn`関数の直下に**宣言**部分が巻き上げられます
（if文やfor文におけるブロックスコープも同様に無視されます）。

```js
function fn() {
    // 内側のスコープにあるはずの変数`x`が参照できる
    console.log(x); // => undefined
    {
        var x = "varのx";
    }
    console.log(x); // => "varのx"
}
fn();
```

つまり、先ほどのコードは実際の実行時には、次のように解釈されて実行されていると考えられます。

{{book.console}}
```js
// 解釈されたコード
function fn() {
    // もっとも近い関数スコープの先頭に宣言部分が巻き上げられる
    var x;
    console.log(x); // => undefined
    {
        // 変数への代入はそのままの位置に残る
        x = "varのx";
    }
    console.log(x); // => "varのx"
}
fn();
```

この変数の**宣言**部分がもっとも近い関数またはグローバルスコープの先頭に移動しているように見える動作のことを変数の**巻き上げ**（hoisting）と呼びます。

このように`var`は`let`、`const`とは異なった動作をしています。
`var`は巻き上げによりブロックスコープを無視して、宣言部分を自動的に関数スコープの先頭に移動するという予測しにくい問題を持っています。
この問題のもっとも簡単な回避方法は`var`を使わないことですが、`var`を含んだコードではこの動作に気をつける必要があります。

<!-- textlint-enable eslint -->

## 関数宣言と巻き上げ {#function-declaration-hoisting}

<!-- textlint-disable eslint -->

`function`キーワードを使った関数宣言も`var`と同様に、もっとも近い関数またはグローバルスコープの先頭に**巻き上げ**られます。
次のコードでは、実際に`hello`関数を宣言した行より前に関数を呼び出せます。

{{book.console}}
```js
// `hello`関数の宣言より前に呼び出せる
hello(); // => "Hello"

function hello(){
    return "Hello";
}
```

これは、関数宣言は**宣言**そのものであるため、`hello`関数そのものがスコープの先頭に巻き上げられます。
つまり先ほどのコードは、次のように解釈されて実行されていると考えられます。

```js
// 解釈されたコード
// `hello`関数の宣言が巻き上げられる
function hello(){
    return "Hello";
}

hello(); // => "Hello"
```

`function`キーワードによる関数宣言も巻き上げられます。
しかし、`var`による変数宣言の巻き上げとは異なり、問題となることはほとんどありません。
なぜなら、実際に巻き上げられた関数を呼び出せるためです。

注意点として、`var`で宣言された変数へ関数を代入した場合は`var`のルールで巻き上げられます。
<!-- textlint-enable eslint -->
<!-- textlint-disable -->
そのため、`var`で変数へ関数を代入する関数式では、`hello`変数が巻き上げにより`undefined`となるため呼び出せません（「[関数と宣言（関数式）][]」を参照）。
<!-- textlint-enable -->
<!-- textlint-disable eslint -->

{{book.console}}
```js
// `hello`変数は巻き上げられ、暗黙的に`undefined`となる
hello(); // => TypeError: hello is not a function

// `hello`変数へ関数を代入している
var hello = function(){
    return "Hello";
};
```

<!-- textlint-enable eslint -->

### [コラム] 即時実行関数 {#immediate-function}

即時実行関数（**IIFE**, _Immediately-Invoked Function Expression_）は、
グローバルスコープの汚染を避けるために生まれたイディオムです。

次のように、無名関数を宣言した直後に呼び出すことで、任意の処理を関数のスコープに閉じて実行できます。
関数スコープを作ることで`foo`変数は無名関数の外側からはアクセスできません。

<!-- textlint-disable eslint -->

{{book.console}}
```js
// 無名関数を宣言 + 実行を同時に行っている
(function() {
    // 関数のスコープ内でfoo変数を宣言している
    var foo = "foo";
    console.log(foo); // => "foo"
})();
// foo変数のスコープ外
console.log(typeof foo === "undefined"); // => true
```

関数を**式**として定義して、そのまま呼び出しています。
`function`からはじまってしまうとJavaScriptエンジンが**関数宣言**と解釈してしまうため、無害なカッコなどで囲んで**関数式**として解釈させるのが特徴的な記法です。これは次のように書いた場合と意味は同じですが、無名関数を定義して実行するため短く書くことができ、余計な関数定義がグローバルスコープに残りません。

{{book.console}}
```js
function fn() {
    var foo = "foo";
    console.log(foo); // => "foo"
}
fn();
// foo変数のスコープ外
console.log(typeof foo === "undefined"); // => true
```

ECMAScript 5までは、変数を宣言する方法は`var`しか存在しませんでした。
そのため、即時実行関数は`var`によるグローバルスコープの汚染を防ぐために使われていました。

しかしECMAScript 2015で導入された`let`と`const`により、ブロックスコープに対して変数宣言できるようになりました。
そのため、グローバルスコープの汚染を防ぐための即時実行関数は不要です。
先ほどの即時実行関数は次のように`let`や`const`とブロックスコープで置き換えられます。

<!-- textlint-enable eslint -->

{{book.console}}
```js
{
    // ブロックスコープ内でfoo変数を宣言している
    const foo = "foo";
    console.log(foo); // => "foo"
}
// foo変数のスコープ外
console.log(typeof foo === "undefined"); // => true
```

## クロージャー {#closure}

最後にこの章ではクロージャーと呼ばれる関数とスコープに関わる性質について見ていきます。
クロージャーとは「外側のスコープにある変数への参照を保持できる」という関数が持つ性質のことです。

クロージャーは言葉で説明しただけではわかりにくい性質です。
このセクションでは、クロージャーを使ったコードがどのように動くのかを理解することを目標にします。

次の例では`createCounter`関数が、関数内で定義した`increment`関数を返しています。
その返された`increment`関数を`myCounter`変数に代入しています。この`myCounter`変数を実行するたびに1, 2, 3と1ずつ増えた値を返しています。

さらに、もう一度`createCounter`関数を実行して、その返り値を`newCounter`変数に代入します。
`newCounter`変数も実行するたびに1ずつ増えていますが、`myCounter`変数とその値を共有しているわけではないことがわかります。

{{book.console}}
```js
// `increment`関数を定義して返す関数
function createCounter() {
    let count = 0;
    // `increment`関数は`count`変数を参照
    function increment() {
        count = count + 1;
        return count;
    }
    return increment;
}
// `myCounter`は`createCounter`が返した関数を参照
const myCounter = createCounter();
myCounter(); // => 1
myCounter(); // => 2
// 新しく`newCounter`を定義する
const newCounter = createCounter();
newCounter(); // => 1
newCounter(); // => 2
// `myCounter`と`newCounter`は別々の状態を持っている
myCounter(); // => 3
newCounter(); // => 3
```

このように、まるで関数が状態（ここでは1ずつ増える`count`という値）を持っているように振る舞える仕組みの背景にはクロージャーがあります。
クロージャーは直感的に理解しにくいため、まずはクロージャーを理解するために必要な「静的スコープ」と「メモリ管理の仕組み」について見ていきます。

### 静的スコープ {#static-scope}

クロージャーを理解するために、今まで意識してこなかったスコープの性質について見ていきます。
JavaScriptのスコープには、どの識別子がどの変数を参照するかが静的に決定されるという性質があります。
つまり、コードを実行する前にどの識別子がどの変数を参照しているかがわかるということです。

次のような例を見てみます。
`printX`関数内で変数`x`を参照していますが、変数`x`はグローバルスコープと関数`run`の中で、それぞれ定義されています。
このとき`printX`関数内の`x`という識別子がどの変数`x`を参照するかは静的に決定されます。

結論から言えば、`printX`関数中にある識別子`x`はグローバルスコープ（＊1）の変数`x`を参照します。
そのため、`printX`関数の実行結果は常に`10`となります。

{{book.console}}
```js
const x = 10; // ＊1

function printX() {
    // この識別子`x`は常に ＊1 の変数`x`を参照する
    console.log(x); // => 10
}

function run() {
    const x = 20; // ＊2
    printX(); // 常に10が出力される
}

run();
```

スコープチェーンの仕組みを思い出すと、この識別子`x`は次のように名前解決されてグローバルスコープの変数`x`を参照することがわかります。

1. `printX`の関数スコープに変数`x`が定義されていない
2. ひとつ外側のスコープ（グローバルスコープ）を確認する
3. ひとつ外側のスコープに`const x = 10;`が定義されているので、識別子`x`はこの変数を参照する

つまり、`printX`関数中に書かれた`x`という識別子は、`run`関数の実行とは関係なく、静的に＊1で定義された変数`x`を参照することが決定されます。
このように、どの識別子がどの変数を参照しているかを静的に決定する性質を**静的スコープ**と呼びます。

この静的スコープの仕組みは`function`キーワードを使った関数宣言、メソッド、Arrow Functionなどすべての関数で共通する性質です。

### [コラム] 動的スコープ {#dynamic-scope}

JavaScriptは静的スコープです。
しかし、動的スコープという呼び出し元により識別子がどの変数を参照するかが変わる仕組みを持つ言語もあります。

次のコードは、動的スコープの動きを説明する**疑似的な言語のコード例**です。
識別子`x`が呼び出し元のスコープを参照する仕組みである場合には、次のような結果になります。

```
// 動的スコープの疑似的な言語のコード例（JavaScriptではありません）
// 変数`x`を宣言
var x = 10;

// `printX`という関数を定義
fn printX() {
    // 動的スコープの言語では、識別子`x`は呼び出し元によってどの変数`x`を参照するかが変わる
    // `print`関数でコンソールへログ出力する
    print(x);
}

fn run() {
    // 呼び出し元のスコープで、変数`x`を定義している
    var x = 20;
    printX();
}

printX(); // ここでは 10 が出力される
run(); // ここでは 20 が出力される
```

このように関数呼び出し時に呼び出し元のスコープの変数を参照する仕組みを**動的スコープ**と呼びます。

JavaScriptは変数や関数の参照先は静的スコープで決まるため、上記のような動的スコープではありません。
しかし、JavaScriptでも`this`という特別なキーワードだけは、呼び出し元によって動的に参照先が変わります。
`this`というキーワードについては次の章で解説します。

## メモリ管理の仕組み {#memory-management}

プログラミング言語は、使わなくなった変数やデータを解放する仕組みを持っています。
なぜなら、変数や関数を定義すると定義されたデータはメモリ上に確保されますが、ハードウェアのメモリは有限だからです。
そのため、メモリからデータがあふれないように、必要なタイミングで不要なデータをメモリから解放する必要があります。

不要なデータをメモリから解放する方法は言語によって異なりますが、JavaScriptでは**ガベージコレクション**が採用されています。
ガベージコレクションとは、どこからも参照されなくなったデータを不要なデータと判断して自動的にメモリ上から解放する仕組みのことです。

JavaScriptにはガベージコレクションがあるため、手動でメモリを解放するコードを書く必要はありません。
しかし、ガベージコレクションといったメモリ管理の仕組みを理解することは、スコープやクロージャーに関係するため大切です。

どのようなタイミングでメモリ上から不要なデータが解放されるのか、具体的な例を見てみましょう。

次の例では、最初に`"before text"`という文字列のデータがメモリ上に確保され、変数`x`はそのメモリ上のデータを参照しています。
その後、`"after text"`という新しい文字列のデータを作り、変数`x`はその新しいデータへ参照先を変えています。

このとき、最初にメモリ上へ確保した`"before text"`という文字列のデータはどこからも参照されなくなっています。
どこからも参照されなくなった時点で不要になったデータと判断されるためガベージコレクションの回収対象となります。
その後、任意のタイミングでガベージコレクションによって回収されてメモリ上から解放されます。[^2]

```js
let x = "before text";
// 変数`x`に新しいデータを代入する
x = "after text";
// このとき"before text"というデータはどこからも参照されなくなる
// その後、ガベージコレクションによってメモリ上から解放される
```

次にこのガベージコレクションと関数の関係性について考えてみましょう。
よくある誤解として「関数の中で作成したデータは、その関数の実行が終了したら解放される」というのがあります。
関数の中で作成したデータは、その関数の実行が終了した時点で必ずしも解放されるわけではありません。

具体的に、「関数の実行が終了した際に解放される場合」と「関数の実行が終了しても解放されない場合」の例をそれぞれ見ていきます。

まずは、関数の実行が終了した際に解放されるデータの例です。

次のコードでは、`printX`関数の中で変数`x`を定義しています。
この変数`x`は、`printX`関数が実行されるたびに定義され、実行終了後にどこからも参照されなくなります。
どこからも参照されなくなったものは、ガベージコレクションによって回収されてメモリ上から解放されます。

{{book.console}}
```js
function printX() {
    const x = "X";
    console.log(x); // => "X"
}

printX();
// この時点で`"X"`を参照するものはなくなる -> 解放される
```

次に、関数の実行が終了しても解放されないデータの例です。

次のコードでは、`createArray`関数の中で定義された変数`tempArray`は、`createArray`関数の返り値となっています。
この、関数で定義された変数`tempArray`は返り値として、別の変数`array`に代入されています。
つまり、変数`tempArray`が参照している配列オブジェクトは、`createArray`関数の実行終了後も変数`array`から参照され続けています。
ひとつでも参照されているならば、そのデータが自動的に解放されることはありません。

{{book.console}}
```js
function createArray() {
    const tempArray = [1, 2, 3];
    return tempArray;
}
const array = createArray();
console.log(array); // => [1, 2, 3]
// 変数`array`が`[1, 2, 3]`という値を参照している -> 解放されない
```

つまり、関数の実行が終了したことと関数内で定義したデータの解放のタイミングは直接関係ないことがわかります。
そのデータがメモリ上から解放されるかどうかはあくまで、そのデータが参照されているかによって決定されます。

### クロージャーがなぜ動くのか {#why-closure-work}

ここまでで「静的スコープ」と「メモリ管理の仕組み」について説明してきました。

- 静的スコープ: ある変数がどの値を参照するかは静的に決まる
- メモリ管理の仕組み: 参照されなくなったデータはガベージコレクションにより解放される

クロージャーとはこの２つの仕組みを利用して、関数内から特定の変数を参照し続けることで関数が状態を持てる仕組みのことを言います。
<!-- クロージャーは関数閉包と呼ばれることがありますが、関数の内側に変数を閉じ込めることで、まるで関数が状態（変数）を持っているように見えます。 -->

最初にクロージャーの例として紹介した`createCounter`関数の例を改めて見てみましょう。

{{book.console}}
```js
const createCounter = () => {
    let count = 0;
    return function increment() {
        // `increment`関数は`createCounter`関数のスコープに定義された`変数`count`を参照している
        count = count + 1;
        return count;
    };
};
// createCounter()の実行結果は、内側で定義されていた`increment`関数
const myCounter = createCounter();
// myCounter関数の実行結果は`count`の評価結果
console.log(myCounter()); // => 1
console.log(myCounter()); // => 2
```

つまり次のような参照の関係が`myCounter`変数と`count`変数の間にはあることがわかります。

- `myCounter`変数は`createCounter`関数の返り値である`increment`関数を参照している
- `myCounter`変数は`increment`関数を経由して`count`変数を参照している
- `myCounter`変数を実行した後も`count`変数への参照は保たれている

<!-- 参照の方向の図 -->

> `myCounter` → `increment` → `count`

`count`変数を参照するものがいるため、`count`変数は自動的に解放されません。
そのため`count`変数の値は保持され続け、`myCounter`変数を実行するたびに1ずつ大きくなっていきます。

このように`count`変数が自動解放されずに保持できているのは「`increment`関数内から外側の`createCounter`関数スコープにある`count`変数を参照している」ためです。
このような性質のことを**クロージャー**（関数閉包）と呼びます。クロージャーは「静的スコープ」と「参照され続けている変数のデータが保持される」という2つの性質によって成り立っています。

JavaScriptの関数は静的スコープとメモリ管理という2つの性質を常に持っています。そのため、ある意味ではすべての関数がクロージャーとなりますが、ここでは関数が特定の変数を参照することで関数が状態を持っていることを指します。

先ほどの例では`createCounter`関数を実行するたびに、それぞれ`count`と`increment`関数が定義されます。そのため、`createCounter`関数を実行すると、それぞれ別々の`increment`関数が定義され、別々の`count`変数を参照します。

次のように`createCounter`関数を複数回呼び出してみると、別々の状態を持っていることが確認できます。

{{book.console}}
```js
const createCounter = () => {
    let count = 0;
    return function increment() {
        // 変数`count`を参照し続けている
        count = count + 1;
        return count;
    };
};
// countUpとnewCountUpはそれぞれ別のincrement関数(内側にあるのも別のcount変数)
const countUp = createCounter();
const newCountUp = createCounter();
// 参照している関数(オブジェクト)は別であるため===は一致しない
console.log(countUp === newCountUp);// false
// それぞれの状態も別となる
console.log(countUp()); // => 1
console.log(newCountUp()); // => 1
```

### クロージャーの用途 {#closure-usecase}

クロージャーはさまざまな用途に利用されますが、次のような用途で利用されることが多いです。

- 関数に状態を持たせる手段として
- 外から参照できない変数を定義する手段として
- グローバル変数を減らす手段として
- 高階関数の一部分として

これらはクロージャーの特徴でもあるので、同時に使われることがあります。

たとえば次の例では、`privateCount`という変数を関数の中に定義しています。
この`privateCount`変数は、外のグローバルスコープからは直接参照できません。
外から参照する必要がない変数をクロージャーとなる関数に閉じ込めることで、グローバルに定義する変数を減らせています。

{{book.console}}
```js
const createCounter = () => {
    // 外のスコープから`privateCount`を直接参照できない
    let privateCount = 0;
    return () => {
        privateCount++;
        return `${privateCount}回目`;
    };
};
const counter = createCounter();
console.log(counter()); // => "1回目"
console.log(counter()); // => "2回目"
```

また、関数を返す関数のことを高階関数と呼びますが、クロージャーの性質を使うことで次のように`n`より大きいかを判定する高階関数を作れます。
最初から`greaterThan5`という関数を定義すればよいのですが、高階関数を使うことで条件を後から定義できるなどの柔軟性があります。

{{book.console}}
```js
function greaterThan(n) {
    return function(m) {
        return m > n;
    };
}
// 5より大きな値かを判定する関数を作成する
const greaterThan5 = greaterThan(5);
console.log(greaterThan5(4)); // => false
console.log(greaterThan5(5)); // => false
console.log(greaterThan5(6)); // => true
```

<!-- textlint-disable -->
クロージャーは、変数が参照する値が静的に決まる静的スコープという性質とデータは参照されていれば保持されるという2つの性質によって成り立っています。
<!-- textlint-enable -->

JavaScriptには、関数を短く定義できるArrow Functionや高階関数であるArrayの`forEach`メソッドなどクロージャーを自然と利用しやすい環境があります。
関数を理解する上ではクロージャーを理解することが大切です。

### [コラム] 状態を持つ関数オブジェクト {#closure-vs-function-object}

JavaScriptでは関数はオブジェクトの一種です。オブジェクトであるため直接プロパティに値を代入できます。
そのため、クロージャーを使わなくても、次のように関数にプロパティとして状態を持たせることが可能です。

{{book.console}}
```js
function countUp() {
    // countプロパティを参照して変更する
    countUp.count = countUp.count + 1;
    return countUp.count;
}
// 関数オブジェクトにプロパティとして値を代入する
countUp.count = 0;
// 呼び出すごとにcountが更新される
console.log(countUp()); // => 1
console.log(countUp()); // => 2
```

しかし、この方法は推奨されていません。なぜなら、関数の外から`count`プロパティを変更できるためです。
関数オブジェクトのプロパティは外からも参照でき、そのプロパティ値は変更できます。
関数の中でのみ参照可能な状態を扱いたい場合には、それを強制できるクロージャーが有効です。

{{book.console}}
```js
function countUp() {
    // countプロパティを参照して変更する
    countUp.count = countUp.count + 1;
    return countUp.count;
}
countUp.count = 0;
// 呼び出すごとにcountが更新される
console.log(countUp()); // => 1
// 直接値を変更できてしまう
countUp.count = 10;
console.log(countUp()); // => 11
```

## まとめ {#conclusion}

この章では関数を中心にスコープについて学びました。

- 関数やブロックはスコープを持つ
- スコープはネストできる
- もっとも外側にはグローバルスコープがある
- スコープチェーンは内側から外側のスコープへと順番に変数が定義されているか探す仕組みのこと
- `var`キーワードでの変数宣言や`function`での関数宣言では巻き上げが発生する
- クロージャーは静的スコープとメモリ管理の仕組みからなる関数が持つ性質

[変数と宣言]: ../variables/README.md
[変数と宣言#let]: ../variables/README.md#let
[関数と宣言（関数式）]: ../function-declaration/README.md#function-expression
[文と式]: ../statement-expression/README.md
[undefinedはリテラルではない]: ../data-type/README.md#undefined-is-not-literal
[^1]: この仕組みはTemporal Dead Zone（TDZ）と呼ばれます。
[^2]: ECMAScriptの仕様ではガベージコレクションの実装の規定はないため、実装依存の処理となります。
