---
title: "HTTPのメソッドについて"
date: 2022-10-28T20:36:16+09:00
draft: true
---
### そもそもHTTPとは  

__通信するときの「約束事」__  

お互い言語を揃えないと通じ合えないって感じ。  
> ex:日本語と英語だとディスコミニケーションだからどっちかに合わせる  

HTTPはホームページのファイルを受け渡す時に使う約束事を指してる。  

ここではHTTPのメソッドに焦点を当ててより深めにアプローチしてみよう。  

別記事でサーバーとクライアントについて記載したが、例えばHTTPサーバーを実装したら、サーバーとクライアント(サイト側とサイトを見る人のようなイメージ)との間でやりとり、ここでは  
__クライアントから情報をサーバーに渡すことが出来るようになる。__  

これを踏まえた上でPOSTというメソッドについてまず確認してみる。  

## HTTPメソッドとは  

まず、そもそもの話HTTPメソッドとは何なのかというところから。  
__HTTPメソッドとはHTTPリクエストの種類を指す。__  

> 【補足】HTTPリクエスト…文字通りだけどwebブラウザがwebサーバに「このページちょうだい」ってお願いすること  

HTTPのHTTP1.1のリクエストは8種類あり、そのうち4つはモジュールでも扱ったCRUDに対応している。  
GET    ⇄    Read  
POST   ⇄    Create  
PUT    ⇄    Update/Create  
SELETE ⇆    Delete  

# POSTメソッドの実装  
今回は実際にデータを受け取ってみたいので、上記のうちのPOSTメソッドを扱うコードを作成してみる。  
使えそうなページをGithubからクローンしてみたらVScodeにてファイル構成を確認。  
中身として、  
- docker-compose.yml
- Dockerfile
- index.js
- package.json  

index.jsを以下の通り編集する。  
```  
'use strict';
 const http = require('http');
 const server = http
   .createServer((req, res) => {
    const now = new Date();
    console.info('[' + now + '] Requested by ' + req.socket.remoteAddress);
     res.writeHead(200, {
       'Content-Type': 'text/plain; charset=utf-8'
     });
    switch (req.method) {
      case 'GET':
        res.write('GET ' + req.url);
        break;
      case 'POST':
        res.write('POST ' + req.url);
       let rawData = '';
        req
          .on('data', chunk => {
            rawData = rawData + chunk;
          })
          .on('end', () => {
            console.info('[' + now + '] Data posted: ' + rawData);
          });
        break;
      default:
        break;
    }
     res.end();
   })
   .on('error', e => {
     console.error('[' + new Date() + '] Server Error', e);
   })
   .on('clientError', e => {
     console.error('[' + new Date() + '] Client Error', e);
 const port = 8000;
 server.listen(port, () => {
   console.info('[' + new Date() + '] Listening on ' + port);
 });
```  
## 【補足】switch文  
 if文と似ている。いわゆる条件分岐で使う構文。
 違いとして…
> __if文は、上から順番に条件を判定していき、条件が合致したら「指定の処理」を実行する。__  
> __どれかの条件に合致するとそれ以降の条件判定は行わない。__  
> __switch文は、ある特定のデータの値のパターンをチェックする。__  
> __特定のデータに対して、多くの比較値で条件分岐する場合は、if文よりもswitch文の方が簡潔に書くことが出来る。__  

例として、以下のコードでは変数nがcase文で指定された1の時と2の時で、実行される行が変わる。またbreakという文が実行されるかreturn文が実行されることで、case文で指定された処理を終了する。  
```  
switch (n) {
  case 1:
    // n === 1 の時、この行を実行
    break; // ここで処理を中断
  case 2:
    // n === 2 の時、この行を実行
    break; // ここで処理を中断
  default:
    break; // n が 1 でも 2 でもない場合はすぐ中断し、なにもしない
}  
```  
では、動作を確認してみる。  
```  
cd ~/workspace/node-js-http-3013
docker-compose up -d
docker-compose exec app bash  
node  
```  
コンソールにアクセスして、REPLを起動させる。  
以下をコピペして実行してみる。  
```  
function greet(name) {
  switch (name) {
    case 'taro':
      return 'こんにちは';
    case 'john':
      return 'hello';
    default:
      console.log('登録なし');
      break;
  }
  return '';
}
greet('taro');
greet('john');
greet('pochi');

```  
すると、  
```
> greet('taro');
'こんにちは'
> greet('john');
'hello'
> greet('pochi');
登録なし
''

```  
と表示される。  
冒頭の説明のようにif文と同じように条件の判定を行うことができる。  

これらを踏まえて、index.jsの中身について深掘りしてみる。  
```
switch (req.method) {
  case 'GET':
    res.write('GET ' + req.url);
    break;
  case 'POST':
    res.write('POST ' + req.url);
    let rawData = '';
    req
      .on('data', chunk => {
        rawData = rawData + chunk;
      })
      .on('end', () => {
        console.info('[' + now + '] Data posted: ' + rawData);
      });
    break;
  default:
    break;
}

```  
ここでは、reqオブジェクトから、HTTP メソッドの文字列req.methodを得て、処理を分岐させている。  
'GET'メソッドならば、そのURLをコンテンツとしてレスポンスに返して終わりになる。'POST'メソッドの際もまずは、そのURLをコンテンツとしてレスポンスに返している。  
そして、POSTの際には追加して送られてくるデータがあるので、その処理を記述している。  
```
let rawData = '';
req
  .on('data', chunk => {
    rawData = rawData + chunk;
  })
  .on('end', () => {
    console.info('[' + now + '] Data posted: ' + rawData);
  });

```  
reqと書かれているリクエストオブジェクトも、Streamと同様にイベントを発行するオブジェクト。そのため、データを受けとった際にはdataというイベントが発生する。データは細切れな状態でchunk変数に入れて受け取り、それを元のrawData という文字列に繋げる。全て受信したら、文字列データをinfoログとして出力している。  
ではコードを実行してみよう。tmuxでウィンドウを2つ立ち上げてアクセスする。  
```  
tmux  
node index.js  
```  
別ウィンドウはctrl+bを押した後cを押すとウィンドウが切り替わる。  
この別ウィンドウでGETメソッドにアクセスする。  
```  
curl http://localhost:8000/messages  
```  
以上のようにURLのパスの後ろに/messagesと加えてアクセス。  
```
GET /messages

```  
とクライアント側のコンソールに表示されたらok  
更にPOSTメソッドでのアクセスを行うために、データを送ってみよう。  
```  
curl -d 'message=こんにちは' http://localhost:8000/messages  
```  
その後、  
```
POST /messages

```  
と表示されたらこれもok  
次にサーバー側のコンソールを、Ctrl+b→0を押して確認してみる。  
```
[Tue Feb 12 2020 16:37:51 GMT+0900 (GMT+09:00)] Requested by ::ffff:127.0.0.1
[Tue Feb 12 2020 16:37:51 GMT+0900 (GMT+09:00)] Data posted: message=こんにちは

```  
これで、クライアントからサーバーに対してデータを送信できた。

POSTメソッドを使うと更に大きなファイルであっても送信ができる。

【補足】curl -d フォームの送信にて使用。(-dは--dataも可)  
```  
例:$ curl --data FORM_ID=VALUE http://www.example.com/  
```  



