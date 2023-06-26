---
title: "モジュール(Node.js3)"
date: 2022-10-26T20:28:02+09:00
draft: true
---
## モジュールって?  
__意味としては「部品」、ITっぽくに言うと「それ自身単独でも機能は成り立つけど、普通は他のモノと組み合わせて使うモノ」__  
  
ここでは、
___モジュール化した処理___
と言うのを考えてみる。 
上での意味を加味すると、その処理のモジュール化=それ自体で体を為す処理また、後付けで追加機能を実装することもできる処理→
___「処理そのものが拡張性がある1個のパーツ」___
って感じ。  
ex:ラジコンのモーター、ギターのピックアップetc…　　


## モジュールを作ってみよう  
ここではモジュールの実装まで試してみる。  
実装するにあたって今回はbotを使用する。  
> bot(ボット)…ROBOT(ロボット)から生まれた言葉。一定のタスクや処理を自動化するためのアプリやプログラムのこと。Twiiterでの有名人の名言をつぶやくBOTやチャットボットとかが有名。  


botでの要件については以下のものがあげられる。  
> bot名 add ◯◯する …タスクを作る(add:加える、追加)  
> bot名 done ◯◯する …タスクを完了の状態にする  
> bot名 done ◯◯する …タスクを削除する(delete)  
> bot名 list …
__未完了__
のリスト一覧が表示される
> bot名 donelist …
__完了している__
タスク一覧が表示される

要件を考えるときに気をつけないといけないのは、  
__その要件に抜け漏れがないかどうか__
ということ。  
抜け漏れの発生=ソフトウェアが正常に動かない等実用できる状態じゃなかったなんてことも起こりうる。  
ここではそのその要件漏れがないかのチェック代わりとしてCRUDという概念も補足しておく。  
> __CRUD__  
> __Create(生成)・Read (読み取り)・Update (更新)・Delete (削除)からできた用語__  

先程のbotの要件と対応させると…  
__Create  ⇄  add__  
__Read    ⇄  list__  
__Update  ⇄  done&donelist__  
__Delete  ⇄  del__  
といった形になる。  
さらに
__オブジェクトのライフサイクル__
というものを用いて要件漏れのチェックもできるのでそれも続いて補足する。  


__オブジェクトのライフサイクル__  
各個の処理、データの集合体(オブジェクトと呼ぶ)がどんな風に生成、変更、削除されるかの様子を表したモノ。  
例えば、addなら処理とデータが生成され、delならそれらが削除されたという感じ。  

ここまで要件漏れについての前置きが長くなったがここからモジュールの実装について考えてみる。  
今回作るのはtodoという名前でタスクの管理をするnpmパッケージを作る。  
さらにいえば、todoと言うオブジェクトの中にadd, done, list, donelist, del コマンドの処理を、関数（メソッド）として実装させる。  
> 【補足】npmパッケージ…npmとはNode Package Managerの略称。字の如くNode.jsのパッケージを管理するツール。  
> またここでいうパッケージとは、もともと用意された便利機能をまとめているもの。  

```java:  
mkdir ~/workspace/todo
cd ~/workspace/todo
code .  
```  
上記の通りコマンドを順番に入力してリポジトリ(ディレクトリ)を作ってVSコードから内蔵ファイルの編集を行う。  

まず、code .のコマンドでVSコードが起動できたらDockerfile と docker-compose.yml を新しく作成し、以下の通りコピペ。  
__1.Dockerfile__  
```java:  
FROM --platform=linux/x86_64 node:14.15.4

RUN apt-get update
RUN apt-get install -y locales
RUN locale-gen ja_JP.UTF-8
RUN localedef -f UTF-8 -i ja_JP ja_JP
ENV LANG ja_JP.UTF-8
ENV TZ Asia/Tokyo
WORKDIR /todo  
```  

__2.docker-compose.yml__  
```java:  
version: '3'
services:
  app:
    build: .
    tty: true
    volumes:
      - .:/todo  
```  

編集が終わればいつものようにDockerを起動する。  
```java:  
cd ~/workspace/todo  
docker-compose up -d  
docker-compose exec app bash
```  

コンソールが起動したら新しいnpmパッケージを作る。  
以下コンテナのコンソールから  
```java:  
cd /todo
yarn init  
```  
上記の通り入力すればパッケージ作成にあたって確認項目が出てくるのでここでは基本enterキー押し続けてok  
これでpackage.jsonというファイルが出来上がる。  
その後早速package.jsonの中身を以下の通り編集する。  
```java:  
 {
   "name": "todo",
   "version": "1.0.0",
   "main": "index.js",
+  "scripts": {
+    "test": "node test.js"
+  },
   "license": "MIT"
 }  
```  
(+は追記)  
> 【補足】.jsonファイルについて…avaScript Object Notation（JSON、ジェイソン）はデータ記述言語の1つである。軽量なテキストベースのデータ交換用フォーマットでありプログラミング言語を問わず利用できる  
> 異なるプログラミング言語間でデータをやりとりするための、共通のデータ記述形式のこと。  

上記の
__scripts__
プロパティに  
```java:  
{
  "コマンド名": "実際に実行するコマンド"
}  
```  
ここでは  
```java:  
"scripts": {
    "test": "node test.js"
  },  
```  
のようにオリジナルの yarn コマンドが設定できる。  
このように設定することで、  

```
yarn コマンド名

```

とコンソールに打ち込むと  

```
実際に実行するコマンド

```

が実行されるようになる。  

つまりここでは、コンソールに  

```
yarn test

```

と打ち込むと、実際には  

```
node test.js

```

が実行されるようになる。  
__(ちなみにyarnはJSで作られたモジュールを管理するパッケージ管理システム。npmとは互換性がある)__  

## パッケージに必要なファイルの作成  
あとはパッケージで必要なファイルを追加する。  
```  
touch index.js
touch test.js  
```  
ファイルはこれで全部。  
ここからテストコードで動作確認しつつ実装していく。  
まずはindex.jsに  
__add ○○する__ で「○○ する」というタスクを作成  
__list__ で未完了のタスクの一覧を表示  
といった形で2つを実装。  
index.jsを以下の通り編集。  
```  
`'use strict';
// { name: タスクの文字列, state: 完了しているかどうかの真偽値 }
const tasks = [];

/**
 * TODOを追加する
 * @param {string} task
 */
function add(task) {
  tasks.push({ name: task, state: false });
}

/**
* TODOの一覧の配列を取得する
* @return {array}
*/
function list() {
  return tasks
    .filter(task => !task.state)
    .map(t => t.name);
}

module.exports = { add, list };  
```  
## 【補足】filter  
与えられた関数の戻り値(処理を終了する際に、呼び出し元に対して渡す値)がtrueである時だけ、その配列の要素を選別した配列を作ることができる。  
ここではfilterの動きを知るために…  
```  
node  
```  
でREPLを起動し、  
```
[1, 2, 3, 4].filter(function(n) {
  return n % 2 === 0;
});

```  
と入力する。すると…  
```
[2, 4]

```  
以上のように表示される。これは無名関数が2で割った時の余りが0になる時(n % 2 === 0)だけtrueを返すという関数であるため、配列の要素が 2 の倍数のものだけに選別される。文字通りフィルター。  
>  無名関数…コンピュータプログラム上で定義される関数のうち、名前を付けずに定義されるもの。その場ですぐに実行したり、変数に代入したり、他の関数に引数として渡したりするのに用いられることが多い。  

またアロー関数でも記述できる。  
```
'use strict';
[1, 2, 3, 4].filter(n => n % 2 === 0);

```  
と入力したら、  
```
[2, 4]

```  
と同じように表示されるはず。アロー関数は、戻り値を1つの式で表せるときにはreturnを省略できる。また、引数が1つのときは引数を囲む( )も省略可能。  
```
tasks
  .filter(task => !task.state)

```  
つまりここでは、filterを利用して、tasksという配列の中で完了していないものを選別している。filterに渡している関数は、

```
task => !task.state

```

というアロー関数になる。この関数は、オブジェクトのstateプロパティの値を反転した真偽値を返します。stateはタスクが完了していれば true、未完了 (TODO) ならば falseだったはず。!で真偽値を反転しているので、この関数はタスクが完了していれば falseを、TODO ならば trueを返す。この真偽値が filter に渡され、TODO タスクのみが選出される。  

---

## 【さらに補足】アロー関数  
その名の通り、　=>(矢)を使って関数リテラルを記述する。  
で、その関数リテラルとは、関数リテラルとはソースコードに直接べた書きされた関数のこと。  
<<関数リテラルの例>>  
```  
let getTriangle = function(base,height){
 return base * height / 2;
};
console.log('三角形の面積は' + getTriangle(10,2));//三角形の面積は10  
```  
関数リテラルは宣言した時点では名前を持たないことから　匿名関数　無名関数　と呼ばれる。
上記の例ではfunction(base,height){...};と名前のない関数を定義した上で変数getTriangleに格納。  
前置きが長くなったが、とどのつまりアロー関数は関数リテラルをシンプルに記述する方法である。  
アロー関数では、  
```  
(引数,...)=>{...関数の本体...}  
```  
以上のように記述する。先程の関数リテラルの例をアロー関数にするとこんな感じ。↓  
```  
let getTriangle = (base, height) => {
  return base * height / 2;
};
console.log('三角形の面積は' + getTriangle(10,2));//三角形の面積は10
```  
functionの代わりに=>が使われている。こんな感じで使用し、条件によっては更に簡素化も可能なので使えるに越したことはないかも。  

---
そして最後のmapに渡している関数も、無名関数をアロー関数を使った省略記法で記述している。

```
t => t.name

```

引数tで配列の要素を取得して、選別されたタスクのnameプロパティから文字列を取得し、その文字列だけの値に変換する関数が与えられている。そのため、これでタスクの中でも TODO であるタスクの名前の一覧が取得される。  
そしてこのlist関数もモジュールのプロパティ（メソッド）として公開。

```
module.exports = { add, list };

```

ここまでで、add、list、done、donelist、delの5つの機能のうち、addとlistの2 つの実装が完了。  

大方の実装はやった感はあるけど、ここでindex.jsを動作させるためのテストをしてみる。  
test.jsを以下のように記述し、モジュールとして index.js を読み込む。
```  
'use strict';
const todo = require('./index.js');
console.log('テストが正常に完了しました');  
```  
では、コンソールで  
```  
yarn test  
```  
と実行。  
```
yarn run v1.21.1
$ node test.js
テストが正常に完了しました
Done in 0.15s.

```  
と表示されたら成功。  

## add と list のテストを実装  
それでは、実際に todo モジュールを利用して、一覧を取得できるかのテストを実装してみよう。test.jsへの実装は以下の通り。  
```  
'use strict';
const todo = require('./index.js');
const assert = require('assert');

// add と list のテスト
todo.add('ノートを買う');
todo.add('鉛筆を買う');
assert.deepStrictEqual(todo.list(), ['ノートを買う', '鉛筆を買う']);

console.log('テストが正常に完了しました');  
```  
### assert モジュールの読み込み

まず、

```
const assert = require('assert');

```

の部分では、Node.js でテストをするためのassertというモジュールを呼び出している。

### add メソッドの呼び出し

で、

```
todo.add('ノートを買う');
todo.add('鉛筆を買う');

```

の部分では、TODOを実際に2つ追加している。

ここで、

```
todo.add('ノートを買う');

```

の、todoはconst todo = require('./index.js');で読み込んだオブジェクト。

index.js では、{ add, list }というオブジェクトをモジュール(部品)として公開したので、const todo = require('./index.js'); の todo にはこのオブジェクトが代入されている。

そして、

```
todo.add('ノートを買う');

```

の addは、todoオブジェクトが持つプロパティの1つ。改めてindex.jsを見ると、{ add, list } というオブジェクトの add プロパティは関数（メソッド）だとわかる。

つまり、

```
todo.add('ノートを買う');

```

はtodoオブジェクトの中のaddメソッドを呼び出している、ということ。

したがって、

```
todo.add('ノートを買う');
todo.add('鉛筆を買う');

```

で2つのタスクを追加したので、todo.list()を呼び出せば、この2つのタスクが取得されるはず。

### list メソッドの呼び出し

実際にテストするのは、以下の通り。

```
assert.deepStrictEqual(todo.list(), ['ノートを買う', '鉛筆を買う']);

```

### deepEqual と deepStrictEqual

assert.deepStrictEqualでは、実際に実行される結果と期待する内容を引数として渡すことで、両者が一致するかどうかをテストできる。

これはassert.strictEqualとは違い、assert.deepStrictEqualでは、配列やオブジェクトの中身まで見て比較してくれる。

ちょっとここで REPL を使い、strictEqualとdeepStrictEqualの動きの差を見てみよう。
```  
node  
```  
REPLを起動。

```
'use strict';
const assert = require('assert');

```

これで、assert モジュールの読み込みをしてみる。

その後、

```
assert.strictEqual(1, 1);
assert.strictEqual([1], [1]);

```

と入力。するとstrictEqual(1, 1)では AssertionError は発生しないけど、strictEqual([1], [1])では、

```
Uncaught:
AssertionError [ERR_ASSERTION]: Values have same structure but are not reference-equal: [1]

... （中略） ...

  generatedMessage: true,
  code: 'ERR_ASSERTION',
  actual: [Array],
  expected: [Array],
  operator: 'strictEqual'
}

```

以上のようなエラーが起こる。

これは JavaScript では配列やオブジェクトを === や == 演算子で比較した場合、同じオブジェクト自身でないとfalseになるという動作から起こる。

なので、

```
1 === 1;

```

はtrueだけど、

```
[1] === [1];

```

はfalseになる。

ただし、

```
const a = [1];
a === a;

```

これは全く同じ配列オブジェクトであるためtrueとなる。ややこしい…

deepStrictEqualはこれらの問題に対処し、内部までちゃんと比較を行う。そのため、

```
assert.deepStrictEqual([1], [1]);
assert.deepStrictEqual({ p: 1 }, { p: 1 });

```

以上を実行しても AssertionError は発生しない。ここでREPL終了。

### テストの実行
```  
yarn test  
```  
以上の通りテストを実行しテストが正常に完了することを確認。

テストに通ったことで、addとlistが正しく実装された。残りもどんどんやっていく。

【補足】strict の意味について

ここまでで、strictEqualとdeepStrictEqualの違いを確認してきたが、assert.strictEqualとassert.deepStrictEqualのほかにassert.equalとassert.deepEqualという「strict のない関数」もある。

strict は「厳密な」という意味で、assert の等価判定に「厳密等価 ===」を使う。だが、strictのないものは、等価判定に「寛容な等価 ==」を使う。


# done と donelist の実装

doneとdonelistの実装とテストも作っていく。

index.js の実装は以下の通り。
```  
`'use strict';
// { name: タスクの文字列, state: 完了しているかどうかの真偽値 }
const tasks = [];

/**
 * TODOを追加する
 * @param {string} task
 */
function add(task) {
  tasks.push({ name: task, state: false });
}

/**
* TODOの一覧の配列を取得する
* @return {array}
*/
function list() {
  return tasks
    .filter(task => !task.state)
    .map(t => t.name);
}

/**
 * TODOを完了状態にする
 * @param {string} task
 */
function done(task) {
  const indexFound = tasks.findIndex(t => t.name === task);
  if (indexFound !== -1) {
    tasks[indexFound].state = true;
  }
}

/**
 * 完了済みのタスクの一覧の配列を取得する
 * @return {array}
 */
function donelist() {
  return tasks
    .filter(task => task.state)
    .map(t => t.name);
}

module.exports = {
  add,
  list,
  done,
  donelist
};  
```  
---

# done メソッドの実装

この実装は、まず配列に task が含まれているかを確認し、 もし存在すれば完了状態をtrueに変更していく。

```
/**
 * TODOを完了状態にする
 * @param {string} task
 */
function done(task) {
  const indexFound = tasks.findIndex(t => t.name === task);
  if (indexFound !== -1) {
    tasks[indexFound].state = true;
  }
}

```

### 配列の findIndex メソッド

今の実装で、配列から task を探索するために、配列の [findIndexという組み込み関数（メソッド）を使っている。findIndexは、配列の要素を順番に見ていき、渡した関数が true を返す要素が見つかったらそのインデックスを返す。見つからなかった場合は、-1を返す。true を返す要素が配列中に複数存在する場合には、最初に見つかった要素のインデックスを返し、それ以外は -1を返す。ここでは配列の各要素をtとして、そのプロパティnameがtaskと一致すれば、発見したとみなす。

---

# donelist の実装

ここではlist関数のfilterの条件が真偽で反転した実装となっている。

```
/**
 * 完了済みのタスクの一覧の配列を取得する
 * @return {array}
 */
function donelist() {
  return tasks
    .filter(task => task.state)
    .map(t => t.name);
}

```
<<具体例>>  
```
**`> [
    { name: '鉛筆を買う', state: false },
    { name: 'ノートを買う', state: false },
    { name: '勉強をする', state: true }
  ].filter(task => task.state);

[{ name: '勉強をする', state: true }]`**

**`> [
    { name: '勉強をする', state: true }
  ].map(t => t.name);

['勉強をする']  
```  

### モジュールとして公開するオブジェクトの更新

そして最後に追加した関数をモジュールとして公開。

```
module.exports = {
  add,
  list,
  done,
  donelist
};

```

# done と donelist のテスト

doneとdonelistもtest.jsに追加!  

```  
`'use strict';
const todo = require('./index.js');
const assert = require('assert');

// add と list のテスト
todo.add('ノートを買う');
todo.add('鉛筆を買う');
assert.deepStrictEqual(todo.list(), ['ノートを買う', '鉛筆を買う']);

// done と donelist のテスト
todo.done('鉛筆を買う');
assert.deepStrictEqual(todo.list(), ['ノートを買う']);
assert.deepStrictEqual(todo.donelist(), ['鉛筆を買う']);

console.log('テストが正常に完了しました');  
```  


先ほどのテストに引き続いてdoneとdonelistのテストを行っている。「鉛筆を買う」を完了にして、それぞれの一覧の状態をテストしている。

```
// done と donelist のテスト
todo.done('鉛筆を買う');
assert.deepStrictEqual(todo.list(), ['ノートを買う']);
assert.deepStrictEqual(todo.donelist(), ['鉛筆を買う']);

```

### テストの実行
```  
yarn test  
```  

テストが正常に完了すればOK

# del の実装

最後に、delの処理を実装する。index.jsを以下のように実装。
```  
`'use strict';
// { name: タスクの文字列, state: 完了しているかどうかの真偽値 }
const tasks = [];

/**
 * TODOを追加する
 * @param {string} task
 */
function add(task) {
  tasks.push({ name: task, state: false });
}

/**
* TODOの一覧の配列を取得する
* @return {array}
*/
function list() {
  return tasks
    .filter(task => !task.state)
    .map(t => t.name);
}

/**
 * TODOを完了状態にする
 * @param {string} task
 */
function done(task) {
  const indexFound = tasks.findIndex(t => t.name === task);
  if (indexFound !== -1) {
    tasks[indexFound].state = true;
  }
}

/**
 * 完了済みのタスクの一覧の配列を取得する
 * @return {array}
 */
function donelist() {
  return tasks
    .filter(task => task.state)
    .map(t => t.name);
}

/**
 * 項目を削除する
 * @param {string} task
 */
function del(task) {
  const indexFound = tasks.findIndex(t => t.name === task);
  if (indexFound !== -1) {
    tasks.splice(indexFound, 1);
  }
}

module.exports = {
  add,
  list,
  done,
  donelist,
  del
};  
```  

詳しく見てみる。

```
/**
 * 項目を削除する
 * @param {string} task
 */
function del(task) {
  const indexFound = tasks.findIndex(t => t.name === task);
  if (indexFound !== -1) {
    tasks.splice(indexFound, 1);
  }
}

```

この実装では、まず配列の組み込み関数findIndexを使って、配列からtaskを文字通り探している。taskが見つかった場合にはそのインデックスが、見つからなかった場合には-1がindexFoundに代入される。(既視感)  

そして、次のif文でindexFoundが-1でない場合、すなわち task が見つかった場合の処理を記述する。

### 配列の splice メソッド

if文のブロックの中では、配列から要素を削除するために、配列のspliceという組み込み関数を使っている。spliceでは、第 1 引数に削除する位置の始まりのインデックス、第 2 引数に削除する要素の個数を指定する。ここでは、インデックスがindexFoundである要素 1 つを削除したいから、

```
tasks.splice(indexFound, 1);

```

としている。

### モジュールとして公開するオブジェクトの更新

これまで同様、新しく追加したdelをモジュールの関数として公開。

```
module.exports = {
  add,
  list,
  done,
  donelist,
  del
};

```

# del のテスト

最後に、削除機能を含めたテストとしてtest.jsを以下のように実装する。  
```  
`'use strict';
const todo = require('./index.js');
const assert = require('assert');

// add と list のテスト
todo.add('ノートを買う');
todo.add('鉛筆を買う');
assert.deepStrictEqual(todo.list(), ['ノートを買う', '鉛筆を買う']);

// done と donelist のテスト
todo.done('鉛筆を買う');
assert.deepStrictEqual(todo.list(), ['ノートを買う']);
assert.deepStrictEqual(todo.donelist(), ['鉛筆を買う']);

// del のテスト
todo.del('ノートを買う');
todo.del('鉛筆を買う');
assert.deepStrictEqual(todo.list(), []);
assert.deepStrictEqual(todo.donelist(), []);

console.log('テストが正常に完了しました');  
```  
以下のようにここまで足したタスクを両方共除去し、一覧が空になっていることをテストしていく。

```
// del のテスト
todo.del('ノートを買う');
todo.del('鉛筆を買う');
assert.deepStrictEqual(todo.list(), []);
assert.deepStrictEqual(todo.donelist(), []);

```

では、早速…  
```  
yarn test  
```  
以上を実行して、テストが正常に完了すればOK  
だいぶ長くなったがタスク管理を行うパッケージのtodoはこれで完成!








