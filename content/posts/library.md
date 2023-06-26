---
title: "ライブラリ(Node.js2)"
date: 2022-10-23T16:29:23+09:00
draft: true
---
### ライブラリって?  
~~ライブラリ…図書館~~  
汎用性の高いプログラムを再利用な形でひとまとまりのしたものを表す=辞書みたいな感じ  
ライブラリは非常に便利だからこそ日頃から使い慣れていく必要がある。
ここでのポイントとして、ライブラリがあること自体を認知しておかないと当然使うことができない。  
なので、テンプレのような処理を実装したい時にライブラリが存在しているか自分で検索するように習慣づけるのが大切だったりする。  

そしてここではNode.jsのためのパッケージマネージャの１つとしてyarnを使用する。  
> __yarn__  
> どんなライブラリのパッケージがインストールされているか記録し、新しいパッケージのインストール、削除を簡単に実行できるようにするプログラム。  
> さらにインストールしたAというパッケージが別のBというパッケージを必要としてる時にBのインストールまで自動的に行うことができる。  


yarnを使用するにはwindowsやmacOSにインストールする必要があるが、今回は別途Dockerで開発用の環境を展開している。またNode.jsに特化したDockerイメージを使っている(Node.js-1の記事から使用中)ので、すでにyarnが含まれているのでインストール不要。  
開発環境を用意していない場合は初めに書いた通り、各OSにyarnを導入してみよう。  
  
## 実際にyarnを使ってみよう  
作業用のリポジトリを自分で探してきてGitHubからフォークする。  
> フォーク(fork、派生)とは該当のリポジトリをベースにしてGitHub上に別のリポジトリとして作るコト≠コピペ  
> クローン(clone)との大きな違いはリポジトリが作られる場所。クローンはローカル環境で、フォークはGitHub上になる。  


__GitHubのページの右上にForkというボタンがあるのでそれを押したらフォークされる。その前提で…__  
cd ~/workspace (※Node.js-1と同じ前提)  
git clone git@github.com:あなたのユーザー名/フォークしてきたリポジトリ.git  
cd フォークしてきたリポジトリ  
code .  
docker-compose up -d  
docker-compose exec app bash  

ここまではNode.js-1でもやった内容とほぼ同じ。  
VSコードを起動してファイルの中身が  
JSファイル…一連の実装を行うファイル  
Dockerfileとdocker-compose.yml  
は最低限必要。  
あと、.gitignore(Gitで管理しないファイルの設定ファイル)もあればなお良い。  

ではファイルも用意できたのでyarnを使ってパッケージのインストールをやってみよう。  
ここでは、axiosという、HTTPリクエストを簡単に送ることができるライブラリをインストールし、使用してみる。  
```java:  
yarn add axios
```
とコマンドと入力する。その後…  
```java:  
yarn add v1.21.1
[1/4] Resolving packages...
[2/4] Fetching packages...
[3/4] Linking dependencies...
[4/4] Building fresh packages...
success Saved lockfile.
success Saved 4 new dependencies.
info Direct dependencies
└─ axios@0.19.2
info All dependencies
├─ axios@0.19.2
├─ debug@3.1.0q
├─ follow-redirects@1.5.10
└─ ms@2.0.0
Done in 0.80s.  
```  
という風にコンソールに出力されたらインストールが成功。ちなみにaxios@0.19.2というのはバージョンを表しているので、使ってる時期等で異なることがあると思われる。またyarnの説明でも少し触れたが、__axiosをインストールした結果このライブラリが他に必要なパッケージもまとめてインストールされている。__  
ではこのままプログラム実装まで進んでみよう。  
JSファイル(ここではapp.js)を以下の通り実装する。  
```java:  
'use strict';
const axios = require('axios');
axios.get('https://www.google.com').then(res => {
  console.log(res.data);
});  
```  
これは、axiosというライブラリの API にそって記述している。パッケージを require 関数でモジュールとして取得し、axiosという変数名をつける。axios のドキュメントによれば、axios はオブジェクトで、getというメソッドを持っている。そのgetメソッドに対して HTTP リクエストを送る URL を渡し、さらにそのthenメソッドに対して HTTP レスポンスを受け取った際に実行する無名関数を渡している。  

早速実行。
```  
`node app.js`  
```  
多分GoogleのトップページのHTMLが一気に表示される。  
このようにyarn addでnode_modulesディレクトリにインストールされたnpmパッケージは自動的に読み込まれ、そのディレクトリ内で利用できる。便利だね…  

## # npm パッケージを作ってみよう  
npmパッケージは自分でも作れる!  
npmサイトに登録するという手段については今回使わず自分で作成する手順を確認してみる。  

### sumライブラリの作成  
Dockerコンテナのコンソールで、次のようにsumパッケージを置くディレクトリを作ってみる。  
```  
cd /yarn-training
mkdir sum
cd sum  
```  
そのまま、  
```  
yarn init  
```  
npm パッケージとして初期化するためのチュートリアルが始まる。ちなみにyarn initのinitはinitialize（初期化）の略です。  
~~そういえばgit initとかあったな~~  
以下確認画面がenterキー連打で  
```
success Saved package.json
Done in 10.10s.

```  
のような感じで表示されたらok  
これで、package.jsonという npm パッケージの情報ファイルが書き込まれた。  
ここからパッケージsumの実装。これは、  
__「ライブラリそのものの実装」になる。__
```  
touch index.js  
```  
でindex.jsを作成したら、sumフォルダをVSコードで開き、index.jsを編集  
```  
'use strict';

/**
 * 数値の配列を受け取って、その要素の合計を返す関数
 */
function add(numbers) {
  let result = 0;
  for (const num of numbers) {
    result = result + num;
  }
  return result;
}

module.exports = {
  add: add
};  
```  
中身について…  
```
function add(numbers) {
  let result = 0;
  for (const num of numbers) {
    result = result + num;
  }
  return result;
}

```  
この add 関数は、数値の配列を受け取り、全てを足し合わせる関数。  
```
module.exports = {
  add: add
};

```  
これについては、npmに限らず、特定の関数をモジュールとしてファイルの外部から使えるようにするのに、使えるようにしたい関数やオブジェクトをmodule.exportsに代入する。  
具体的に言うと、  
module.exportsに、  
```
{
  add: add
}

```  
が代入されている。  
このオブジェクトが、外部から使えるsumモジュールになる。

ここでadd: addの、左側のaddはオブジェクトのプロパティ（メソッド）の名前で、右側のaddは上で定義した関数。

つまり、上で定義したaddという関数をメソッドに持つオブジェクトがsumモジュールになる。  
## 自作sumライブラリを使ったアプリケーション
今度は上で作ったsumパッケージを使うアプリケーションを作成してみる。今度は
__「ライブラリそのものの実装」ではなく「ライブラリを使った実装」となる。__  
```  
cd /yarn-training
mkdir sum-app
cd sum-app  
```  
上記のようにsum-appディレクトリを作成する。  
これまで同様  
```  
yarn init  
```  
enter連打でok  
### 自作sumライブラリのインストール  
```  
yarn add ../sum  
```  
パッケージのインストール。その後、  
```
success Saved lockfile.
success Saved 1 new dependency.
info Direct dependencies
└─ sum@1.0.0
info All dependencies
└─ sum@1.0.0
Done in 0.15s.

```  
と表示されたらnode_modulesにさっき作ったsumパッケージがインストールされたことになる。  
### ### 自作 sum ライブラリを使ったプログラムの実装  
続いてDockerコンテナのコンソールで、

cd /yarn-training/sum-app

sum-appディレクトリに移動。  
```  
touch app.js  
```  
app.jsを作成。sum-appフォルダをVSコードで開き、app.jsを編集。
```  
'use strict';
const s = require('sum');
console.log(s.add([1, 2, 3, 4]));  
```  
sumモジュールを変数sに代入している。自作のsumモジュールはaddのメソッドなのでsはaddのメソッドを持ったオブジェクト。  
console.logでは1から4を足し合わせてるようだ。  
ということで…  
```  
node index.js  
```  
その結果、  
```  
10  
```  
1+2+3+4 = 10となっているので成功!!