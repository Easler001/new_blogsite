---
title: "テンプレートエンジン(Node.js5)"
date: 2022-10-30T11:46:00+09:00
draft: true
---
## テンプレートエンジン  
ここまでで HTML フォームから HTTP サーバーへ情報を受け渡せるようになった。  
今度は HTTP サーバーを Node.js で実装していることを活かして、動的に様々なアンケートを表示させてみる。  

使用するフォルダをクローンしてVSコードで確認。ここについてはこれまでと同様。  
ファイルの構成は  
- docker-compose.yml
- Dockerfile
- form.html
- index.js
- package.json  

を前提とする。  
ここではHTMLを動的に表示させるためテンプレートエンジンを利用する。  
> テンプレートエンジン…テンプレートと文字列とプログラムを組み合わせることで、 静的なユーザーインタフェースのデータである HTML を動的に出力できるライブラリ  

Dockerfileに以下を追記する。  
```
  FROM --platform=linux/x86_64 node:14.15.4
  
  RUN apt-get update
  RUN apt-get install -y locales vim tmux
  RUN locale-gen ja_JP.UTF-8
  RUN localedef -f UTF-8 -i ja_JP ja_JP
  ENV LANG ja_JP.UTF-8
  ENV TZ Asia/Tokyo
  RUN yarn add pug@2.0.4
  RUN yarn global add pug-cli
  WORKDIR /app  
```  

> Pugについて…Node.jsのテンプレートエンジンの一つ
> Pug は、HTML を閉じタグが不要なタグの宣言や入れ子構造に沿った適切なインデントを使うことで、簡潔に表現できる。上記のDockerfileではこのPugのインストール用のコマンドを記載してある。その後、  

```  
echo "node_modules" >> .gitignore
git add .gitignore  
```  
ここでnode_modules をリポジトリ管理しないように .gitignore を作成・編集  
> .gitignoreはGitの管理に含めないファイルを指定するためのファイル。  
> 管理しないファイルがある時に使用する。  

```  
docker-compose up -d
docker-compose exec app bash  
```  
ここで、今まで通りコンソールを起動。  

その後、  
```  
echo "h1 Pug!" | pug  
```  
と入力する。  
実行後、  
```  
<h1>Pug!</h1>  
```  
と言う形で表示されたらPugのインストールは完了。  
ここからはhtmlファイルをPugの形式に合わせて編集する。  
```  
<!DOCTYPE html>
<html lang="jp">
<head>
	<meta charset="UTF-8">
	<title>アンケート</title>
</head>

<body>
	<h1>どちらが食べたいですか？</h1>
	<form method="post" action="/enquetes/yaki-shabu">
		名前: <input type="text" name="name">
		<input type="radio" name="yaki-shabu" value="焼き肉" /> 焼き肉
		<input type="radio" name="yaki-shabu" value="しゃぶしゃぶ" /> しゃぶしゃぶ
		<button type="submit">投稿</button>
	</form>
</body>

</html>  
```  

さらにPugファイルを編集するためにVSコードについて設定を少し弄る。  
macなら基本設定タブから設定を押す。  
①Tab sizeの設定  
検索フィールドにtab sizeと入力  
「Editor: Tab Size」の項目の数字を2にする。→これでtabキーで入力される文字数が2になる  
②Insert Spaces の設定  
検索フィールドに insert spaces と入力  
「Editor: Insert Spaces」の項目のチェックをつける。→これでTab キーでスペースが入力されます。  
③Render Whitespace の設定  
検索フィールドにrender whitespaceと入力  
「Editor: Render Whitespace」の項目で「boundary」を選択  
この設定により、スペースなどの本来見えない文字が、ドットなどで表示されるようになる。  

ここでform.pugを読み込めるようにindex.jsを編集する。  

```  
'use strict';
const http = require('http');
const pug = require('pug');
const server = http
  .createServer((req, res) => {
    const now = new Date();
    console.info('[' + now + '] Requested by ' + req.socket.remoteAddress);
    res.writeHead(200, {
      'Content-Type': 'text/html; charset=utf-8'
    });

    switch (req.method) {
      case 'GET':
        res.write(pug.renderFile('./form.pug'));
        res.end();
        break;
      case 'POST':
        let rawData = '';
        req
          .on('data', chunk => {
            rawData = rawData + chunk;
          })
          .on('end', () => {
            const qs = require('querystring');
            const answer = qs.parse(rawData);
            const body = answer['name'] + 'さんは' +
              answer['yaki-shabu'] + 'に投票しました';
            console.info('[' + now + '] ' + body);
            res.write('<!DOCTYPE html><html lang="ja"><body><h1>' +
              body + '</h1></body></html>');
            res.end();
          });
        break;
      default:
        break;
    }
  })
  .on('error', e => {
    console.error('[' + new Date() + '] Server Error', e);
  })
  .on('clientError', e => {
    console.error('[' + new Date() + '] Client Error', e);
  });
const port = 8000;
server.listen(port, () => {
  console.info('[' + new Date() + '] Listening on ' + port);
});  
```  
そして、  
```  
node index.js  
```  
と入力したら、http://localhost:8000/にアクセス。  
アンケートのフォームが表示されたら成功。  


これをさらに発展させて複数のアンケートを動的に表示させる  
```  
'use strict';
const http = require('http');
const pug = require('pug');
const server = http
  .createServer((req, res) => {
    const now = new Date();
    console.info('[' + now + '] Requested by ' + req.socket.remoteAddress);
    res.writeHead(200, {
      'Content-Type': 'text/html; charset=utf-8'
    });

    switch (req.method) {
      case 'GET':
        if (req.url === '/enquetes/yaki-shabu') {
          res.write(pug.renderFile('./form.pug', {
            path: req.url,
            firstItem: '焼き肉',
            secondItem: 'しゃぶしゃぶ'          }));
        } else if (req.url === '/enquetes/rice-bread') {
          res.write(pug.renderFile('./form.pug', {
            path: req.url,
            firstItem: 'ごはん',
            secondItem: 'パン'
          }));
        }
        res.end();
        break;
      case 'POST':
        let rawData = '';
        req
          .on('data', chunk => {
            rawData = rawData + chunk;
          })
          .on('end', () => {
            const qs = require('querystring');
            const answer = qs.parse(rawData);
            const body = answer['name'] + 'さんは' +
              answer['favorite'] + 'に投票しました';
            console.info('[' + now + '] ' + body);
            res.write('<!DOCTYPE html><html lang="ja"><body><h1>' +
              body + '</h1></body></html>');
            res.end();
          });
        break;
      default:
        break;
    }
  })
  .on('error', e => {
    console.error('[' + new Date() + '] Server Error', e);
  })
  .on('clientError', e => {
    console.error('[' + new Date() + '] Client Error', e);
  });
const port = 8000;
server.listen(port, () => {
  console.info('[' + new Date() + '] Listening on ' + port);
});  
```  
上記では焼肉orしゃぶしゃぶ、ごはんorパンの2つのアンケート  
その後、  
```  
node index.js  
```  
にて起動。それから、  
http://localhost:8000/enquetes/yaki-shabu  
http://localhost:8000/enquetes/rice-bread  
2つのアンケートが眼精していたらok  
ごはんとパンのアンケートに答えると

```
吉村さんはごはんに投票しました

```

以上のように表示され、コンソールのログに、

```
[Mon Dec 23 2020 10:11:22 GMT+0000 (UTC)] 投稿: 吉村さんはごはんに投票しました

```

以上のように表示される。  

これで、テンプレートエンジンを利用して動的なアンケートページが作成できるようになった。



