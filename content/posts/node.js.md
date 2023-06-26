---
title: "Node"
date: 2022-10-09T12:07:01+09:00
draft: true
---
# Node.js 導入と運用まとめ
  ## Node.jsとは  
  >サーバーサイド(ユーザーの視覚的ではない部分(裏側)の部分の処理をしているプログラミング)で動くJavaScript
    

  ## 導入(環境構築)

  >Node.jsはHTML、CSS、 JSと違って環境構築しないといけない。(デフォルトで作成されているわけではないのでその環境を作らないといけない。)  
  使用ツール…Nodebrew(Node.jsをインストールするだけでなく、バージョン管理することもできるツール)  
  ※Homebrewを使用するという方法もあるが、Node.jsを使うだけなら使わなくていい!  
  
  ①公式HPにアクセス→GitHubのページがあるのでジャンプ。(同時進行でターミナル起動)

    GitHubページをスクロールすると、
        README.md
        nodebrew
        

        Install
        Install with curl.
        
        $ curl -L git.io/nodebrew | perl - setup←これをコピーする!!
        Or, download and setup.  

②　①でコピーしたものを(curl -L git.io/nodebrew | perl - setup)ターミナルにペーストする!  
>☆ポイント☆curlというコマンドは、httpリクエストを通してサーバーにアクセスしてダウンロード/送るコマンド入力すると、データがぞろぞろ表示されていく。すると…  


        ========================================
        Export a path to nodebrew:
         
        export PATH=$HOME/.nodebrew/current/bin:$PATH
        ========================================
        と言う感じに出てくる!  
>__そうしたら、上で出てきたexport PATH=$HOME/.nodebrew/current/bin:$PATHを__
       __そのままターミナル上に貼り付ける__  
>__☆ポイント☆ここでコピペしたモノは、Nodebrewのパスを通している。__

③Nodebrewを使ってNode.jsをインストールする

上述のGitHubのページからnodebrew install-binaryを見つけてインストール  
>コマンドは2パターン  
>__nodebrew install-binary latest…一番新しいバージョン__  
>__nodebrew install-binary stable…安定版の中で一番新しいバージョン(初心者はこっち推奨)__

    Node.jsのバージョン確認と指定

    nodebrew lsで使用バージョンが確認できる
        ☆ポイント current:noneと表示された場合は、使用するNode.JSのバージョンが指定されていない状態を指す☆
        →nodeとコマンド入力すると、REPLと言うターミナル上でNode.jsを使用できるようになる。 ctrl+cでREPLモードは解除される。

    バージョン指定:nodebrew use 16.1.1←数字は実際のバージョンの一例　使うNode.jsのバージョンを切り替えられる
    ☆Node.jsは行う行程によってバージョンを変えることがある!☆

    nodebrewのパスを自動で通すようにする
    この段階でターミナルを閉じると初期化されてしまい、毎回上述したexport PATH=$HOME/.nodebrew/current/bin:$PATH
        を入力しなければならないのでこれを解消する

    1.touch .zshrcと入力  ※ touchと.の間は１マス間を開けないと認識されない!
    2.vi .zshrcと入力　※上述の通り１マス開ける!
    3.iを入力すると編集モードになる(INSERTと表示される)→export PATH=$HOME/.nodebrew/current/bin:$PATHをコピペ、escキーを押す
    4.:wqと入力、enterでターミナルに戻れる
    5.ターミナル⇨環境設定→コマンドを実行にチェックマーク
    6.ターミナルを再起動して node -v やnodebrew -vでバージョンを確認したり 
        nodebrew lsできるか確認  

 --------------------------------------------------


# Node.jsの運用

 ## Nodeの準備  
 今回の運用ではDocker Hubを使用する。

 >__ターミナルでディレクトリを用意する_  
 __mkdir -p ~/workspace/nodejs-study__  
 __cd ~/workspace/nodejs-study__

 >___ターミナルからvsコードをダイレクトで起動させる___  
 ___code .___  
 ☆方法(事前設定必須)☆  
 1.vsコードでコマンドパレットを開く。(Command + Shift + P)  
 2.Shellと打って検索  (シェル コマンド:PATH内に'code'コマンドをインストールします　と出てくるはず)  
 3.インストール  
 4.インストール完了のメッセージが出たらok!

 【補足】Dockerについて  
  今回Docker hubというものを使用するが、Docker desktopとかDockerそのものといったものが一体何が違うのかというところも再確認。  
  
   Docker…コンテナ型の仮想環境を作成、配布、実行するためのプラットフォーム=仮想環境を構築するためのツール、コンテナ(仮想環境)でアプリケーションを動かしていく。  
   
   Docker hub…上記のコンテナのイメージ(Dockerコンテナを作るためのレシピみたいなもの)がおいてある場所(例えば今回のようにNode.jsを使ってみたいならNode.jsをDocker hubのページで検索すれば出てくる)  
   
   Docker desktop…

__Docker Hubで環境を構築する__  
https://hub.docker.com/  
今回についてはアカウント登録不要なので以下のチャートにそって進行  
>1.検索フィールドが丈夫にあるので'node'と入力  
2.「node」というDockerイメージが表示される  
3.先ほど用意したcd ~/workspace/nodejs-studyの中にDockerfileというファイルを作成する。  
※Dockerfileには.html等の拡張子はない!  
4.作ったDockerfileに以下を入力orコピペ

FROM --platform=linux/x86_64 node:14.15.4  

RUN apt-get update  
RUN apt-get install -y locales  
RUN locale-gen ja_JP.UTF-8  
RUN localedef -f UTF-8 -i ja_JP ja_JP  
ENV LANG ja_JP.UTF-8  
ENV TZ Asia/Tokyo  
WORKDIR /nodejs-study  


Dockerコンテナを起動するために便利なymlファイルを作成する。  
>~/workspace/nodejs-studyの中にdocker-compose.yml という名前のファイルを作成  
以下ファイル内に入力(インデント((字下げ)も厳密に見られるので入力時に注意))

version: '3'  
>services: 
  app:  
    build: .  
    tty: true  
    volumes:  
      - .:/nodejs-study



# Dockerコンテナの起動

nodeイメージからDockerコンテナを構築して起動する。  

>cd ~/workspace/nodejs-study  フォルダに移動  
docker-compose up -d  起動  

docker-compose exec app bash(起動してコンソールにアクセスする)  
node --version(使用しているnodeのバージョンを確認する)  

終了  
>exit  コンソールから抜ける  
docker-compose down  
-------------------------------  

# http接続  
>mkdir -p ~/workspace/nodejs-http  
cd ~/workspace/nodejs-http  
code .  
Nodeの準備の時と似た形。  
フォルダを作成してvsコード上で確認する。  

Dockerfileの用意 方法は上記同様のため省略。以下を入力orコピペ  
FROM --platform=linux/x86_64 node:14.15.4  

>RUN apt-get update  
RUN apt-get install -y locales tmux vim  
RUN locale-gen ja_JP.UTF-8  
RUN localedef -f UTF-8 -i ja_JP ja_JP  
ENV LANG ja_JP.UTF-8  
ENV TZ Asia/Tokyo  
WORKDIR /app  

ymlファイルの作成  
docker-compose.ymlという名前でファイル作成 以下コピペ  

>version: '3'  
services:  
  app:  
    build: .
    tty: true  
    ports: 
      - 8000:8000  
    volumes:  
      - .:/app  

# コンテナ起動  
>cd ~/workspace/nodejs-http  
docker-compose up -d
ここまでは上とほぼ同じ  
入力ができる状態の確認ができたら以下を入力  
docker-compose exec app bash　　

ここでDockerコンテナのコンソールが起動できたら次の段階に進む。  

# Node.jsプロジェクトの初期化  
ここからは Docker コンテナのコンソールでコマンドを実行していく。











