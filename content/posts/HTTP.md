---
title: "HTTP通信とは何なのか"
date: 2022-10-09T10:37:31+09:00
draft: false
---
# HTTP通信とは
  HTTPと呼ばれる約束事に従って行われる通信のこと
　　　　≒インターネットっぽい通信のこと


	 ☆通信プロトコル(約束事)ex.話す時日本語と英語じゃ伝わらないから言語を統一しよう。


  ## HTTPは「ホームページのファイルを受け渡しするときに使う約束事」

	ex:ホームページを見る流れ
　　　　　①あのページを閲覧したい!→②Webブラウザがそのホームページのファイルがあるサーバ  

　　　　　(Webサーバ)に「ページちょうだい」とお願いする→③そうしたらWebサーバからWebブラウザ  

　　　　　にそのページが渡される→④ページを受け取ったWebブラウザがそれを画面に表示する。


	③④の流れでのやり取りで使うお約束がHTTP

	これら踏まえてHTTPの約束事に従って行う通信=HTTP通信
