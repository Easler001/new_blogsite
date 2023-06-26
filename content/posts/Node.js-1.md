---
title: "集計処理を行うプログラム(Node.js1)"
date: 2022-10-23T12:04:55+09:00
draft: true
---
## 集計処理を行うプログラム

集計は文字通り数を集めて合算すること  
ここではNodeにて合算するメカニズムについてまとめる

一例として各都道府県の人口増減率について処理をしてみる  

### まずは導入から  
 
※前提としてworkspaceというディレクトリを作成している  
```cd ~/workspace/```  
```git clone git@github.com:あなたのユーザー名/adding-up.git```  
```cd adding-up```  
```git checkout main-2021```  
```code .```  
```docker-compose up -d```  
```docker-compose exec app bash```  
上記の```code .``` でVS codeが起動する。  
ファイルの中には  
> app.js (実装するJSのファイル)  
> popu-pref.csv 人口推移についてのデータが入ってる  
> Dockerfile と docker-compose.yml  これはこれまで同様　詳細は<a>https://nostalgic-noyce-f50430.netlify.app/posts/node.js/</a>  

#### コードの補足  
git checkout等について　詳細は本記事全体構成完了後に追記。  

## 今回するコト  
データ元になるCSVファイルから2010年から2015年にかけての人口増減率についてランキング作成をやってみる  
始めるにあたって、いくつかのプロセスを順序立ててやっていく。  
> ①ファイルからのデータ読み込み  
> ②2010年と2015年でデータを分ける  
> ③変化率を計算させる  
> ④変化率で並べてそれを実際に表示させる。  

イメージ的には  
> 1. ◯◯県　?%  
> 2. ◯◯県　?% …  
> となればいいが結果はどうなるかな…?  

まずはapp.jsのファイルに記述させる  
```java:'use strict';
const fs = require('fs');
const readline = require('readline');
const rs = fs.createReadStream('./popu-pref.csv');
const rl = readline.createInterface({ input: rs, output: {} });
rl.on('line', lineString => {
  console.log(lineString);
});
```  

この中で、  
```java:  
const rs = fs.createReadStream('./popu-pref.csv');
const rl = readline.createInterface({ input: rs, output: {} });  
```  
この部分は実際に書かれてるようにcsvファイルからデータを読み込ませている  
__ファイルの読込を行うStreamを生成する__  

#### Stream  
> Node.jsで入出力が起きる処理のほとんどがこれ。  
文字通り「流れ」を示している。ここでは、csvファイルのイベントに反応して情報を返す形。  
このように予めイベントが発生した時に実行する関数を設定して起きたイベントに応じて処理する一連の流れを  __イベント駆動型プログラミングという。__  

__とりあえずここでコンソールに出力してみよう__  
```java:  
node app.js  
```  

```java:
集計年,都道府県名,10〜14歳の人口,15〜19歳の人口  
2010,北海道,237155,258530  
2010,青森県,66023,67308  
2010,岩手県,62587,64637  
2010,宮城県,109287,120010  
2010,秋田県,47425,47154  
2010,山形県,54968,54898  
2010,福島県,101697,101390  
2010,茨城県,143298,144480  
2010,栃木県,94321,93535  
2010,群馬県,99068,96344  
2010,埼玉県,334605,356249  
2010,千葉県,276043,283408  
2010,東京都,492799,546573  
（省略）  
```  

①ファイルデータの読み込み　クリア  
続いてデータの選択(抽出)を行う  
 app.jsを以下の通り書き換えをしてみる  
   
```java:  
 const rl = readline.createInterface({ input: rs, output: {} });
 rl.on('line', lineString => {
-  console.log(lineString);
+  const columns = lineString.split(',');
+  const year = parseInt(columns[0]);
+  const prefecture = columns[1];
+  const popu = parseInt(columns[3]);
+  if (year === 2010 || year === 2015) {
+    console.log(year);
+    console.log(prefecture);
+    console.log(popu);
+  }
 });  
 ```  
 (＋が追加、−が削除)  
  
ここで再度
```java:  
node app.js  
```  
コンソールを確認  
```java:  
2010  
北海道  
258530  
2010  
青森県  
67308  
2010  
岩手県  
64637  
2010  
宮城県  
120010  
2010  
秋田県  
47154  
（省略）  
```  
これで2010年と2015年でデータ分けができた  
ここからはデータの計算をする  
人口増減率を出すには  
> __{(2015の人口÷2010の人口)-1}×100__
  
  で増減率を％単位で出せるはずだが果たして…?  
  とりあえずここでは増加率について出してみる  
  ここで再びapp.jsファイルを書き換えてみよう  
```java:  
 'use strict';
 const fs = require('fs');
 const readline = require('readline');
 const rs = fs.createReadStream('./popu-pref.csv');
 const rl = readline.createInterface({ input: rs, output: {} });
+const prefectureDataMap = new Map(); // key: 都道府県 value: 集計データのオブジェクト
 rl.on('line', lineString => {
   const columns = lineString.split(',');
   const year = parseInt(columns[0]);
   const prefecture = columns[1];
   const popu = parseInt(columns[3]);
   if (year === 2010 || year === 2015) {
-    console.log(year);
-    console.log(prefecture);
-    console.log(popu);
+    let value = prefectureDataMap.get(prefecture);
+    if (!value) {
+      value = {
+        popu10: 0,
+        popu15: 0,
+        change: null
+      };
+    }
+    if (year === 2010) {
+      value.popu10 = popu;
+    }
+    if (year === 2015) {
+      value.popu15 = popu;
+    }
+    prefectureDataMap.set(prefecture, value);
   }
 });
+rl.on('close', () => {
+  console.log(prefectureDataMap);
+});  
```  
 (＋が追加、−が削除) 

 この状態で  
 ```java:  
node app.js  
```  
コンソールを確認  
```java: 
Map {
  '北海道' => { popu10: 258530, popu15: 236840, change: null },
  '青森県' => { popu10: 67308, popu15: 61593, change: null },
  '岩手県' => { popu10: 64637, popu15: 57619, change: null },  
  ```  
ここで一番最後のcloseのイベントを以下のように追加実装させる  
```java:  
rl.on('close', () => {
  for (const [key, value] of prefectureDataMap) {
    value.change = value.popu15 / value.popu10;
  }
  console.log(prefectureDataMap);
});  
```  
__value.change = value.popu15 / value.popu10;これで割り算をしてる__  
これで変化率が出てくるはず  
```java:  
Map {
  '北海道' => { popu10: 258530, popu15: 236840, change: 0.9161025799713767 },
  '青森県' => { popu10: 67308, popu15: 61593, change: 0.9150918167231236 },
  '岩手県' => { popu10: 64637, popu15: 57619, change: 0.8914244163559571 },
```  
最後に並び替え!  
app.jsのclose部分を書き換え 
```java: 
 rl.on('close', () => {
-  console.log(prefectureDataMap);
+  for (const [key, value] of prefectureDataMap) {
+    value.change = value.popu15 / value.popu10;
+  }
+  const rankingArray = Array.from(prefectureDataMap).sort((pair1, pair2) => {
+    return pair2[1].change - pair1[1].change;
+  });
+  console.log(rankingArray);
 });  
 ```  
 (＋が追加、−が削除)  
 これで最後…  
 ```java:  
[ [ '愛知県',
  { popu10: 361670, popu15: 371756, change: 1.0278873005778748 } ],
[ '大阪府',
  { popu10: 416930, popu15: 426504, change: 1.0229630873288083 } ],
[ '富山県',
  { popu10: 47585, popu15: 48442, change: 1.0180098770620993 } ],
[ '神奈川県',
  { popu10: 421017, popu15: 426358, change: 1.0126859485483959 } ],  
  ```  
ランキングがこれでできた!!あとは綺麗に表示させるとか色々構文の中弄ってみても良さそう  

### おまけ  
綺麗に出力させてみる  
app.jsを書き換えてみる (下部分のclose部分のみ) 
```java:  
 rl.on('close', () => {
   for (const [key, value] of prefectureDataMap) {
     value.change = value.popu15 / value.popu10;
   }
   const rankingArray = Array.from(prefectureDataMap).sort((pair1, pair2) => {
     return pair2[1].change - pair1[1].change;
   });
-  console.log(rankingArray);
+  const rankingStrings = rankingArray.map(([key, value]) => {
+    return (
+      key +
+      ': ' +
+      value.popu10 +
+      '=>' +
+      value.popu15 +
+      ' 変化率:' +
+      value.change
+    );
+  });
+  console.log(rankingStrings);
 });  
 ```  
 追加部分で見やすさが追加されてる感じ  
 コンソール出力  
 ```java:  
 node app.js  
 ```  
 すると…  
 ```java:  
 [ '愛知県: 361670=>371756 変化率:1.0278873005778748',
'大阪府: 416930=>426504 変化率:1.0229630873288083',
'富山県: 47585=>48442 変化率:1.0180098770620993',
'神奈川県: 421017=>426358 変化率:1.0126859485483959',  
```  
いい感じ。👍  
最後の最後…  
増減率を試そう  
app.jsを弄ってみよう(close部分を筆者オリジナルで弄ってみた)  
```java:  
rl.on('close', () => {
    for (const [key, value] of prefectureDataMap) {
        //{(2015の人口÷2010の人口)-1}×100を表す//
            value.change = ((value.popu15 / value.popu10)-1)*100;
          }
          const rankingArray = Array.from(prefectureDataMap).sort((pair1, pair2) => {
            return pair2[1].change - pair1[1].change;
          });
          //console.log(rankingArray);//
          const rankingStrings = rankingArray.map(([key, value]) => {
                return (
                  key +
                  ': ' +
                  value.popu10 +
                  '=>' +
                  value.popu15 +
                  ' 変化率:' +
                  value.change +
                  //%単位で導出//
                  ' % '
                );
              });
              console.log(rankingStrings);
        });  
        ```  
```java:  
[
  '愛知県: 361670=>371756 変化率:2.7887300577874807 % ',
  '大阪府: 416930=>426504 変化率:2.296308732880825 % ',
  '富山県: 47585=>48442 変化率:1.8009877062099333 % ',
  '神奈川県: 421017=>426358 変化率:1.2685948548395887 % ',
  '滋賀県: 72773=>73562 変化率:1.0841933134541737 % ',
  '香川県: 43947=>44342 変化率:0.8988099301431296 % ',
  '岐阜県: 101669=>101755 変化率:0.0845882225653849 % ',
  '兵庫県: 268710=>268336 変化率:-0.13918350638234545 % ',
  '長野県: 98872=>98617 変化率:-0.25790921595598704 % ',
  '静岡県: 169029=>168579 変化率:-0.26622650551089144 % ',  
  (以下略)  
  ```  
  これでやりたいことができた!  
  満足

