---
layout:     post
title:      "利用Google Spreadsheet JSON & Imgur 開發App (1)"
date:       2020-03-02
author:     "cc"
header-img: "img/tag-bg.jpg"
catalog: true
tags:
    - API
---
## 簡介

開發iOS App時，我們都希望App內顯示的資料 (文字、數據、圖片等等) 都來自網絡上，因此更新數據就來得更加容易。而本文所介紹的方法適合單純地從網路抓資料顯示，不需要修改或上傳資料，原理是利用 Google Spreadsheet & imgur 當後台，然後將它變成 JSON 讓 iOS App 串接。
 
接下來我們以製作簡單的寵物小精靈圖鑑為例子。

## 建立Google Spreadsheet

搜尋Google Spreadsheet(Google 試算表)，即下面的link。

><a href="https://docs.google.com/spreadsheets/u/0/">Google Spreadsheet</a>

按左下角，新增Spreadsheet
<img src="/img/googleJson/img1.png" width="100%">

輸入如圖中的資料，圖片可透過免費的 imgur作上傳，上傳後copy圖片的網址。
><a href="https://imgur.com/">Imgur</a>
<img src="/img/googleJson/img2.png" width="100%">

然後如圖按檔案 --> 發佈到網絡
<img src="/img/googleJson/img3.png" width="100%">
## 取得 spreadsheet 的 id

編輯 Spreadsheet 時，先複製它的網址，/d/ 後的字串是 Spreadsheet 的 id。
<img src="/img/googleJson/img4.png" width="100%">

因此，ID為1ybZ6Kk1wIKIZnQ_nPtRNNO5iX8Q5U9pz8qWD2jrf1l8

透過 gsx2json 提供的 API。

>http://gsx2json.com/api?id=spreadsheet 的 id&columns=false

將上述得到的id copy到以上位址，如本例即為

>http://gsx2json.com/api?id=1ybZ6Kk1wIKIZnQ_nPtRNNO5iX8Q5U9pz8qWD2jrf1l8&columns=false

若訪問該網址可看到如下圖所示：
<img src="/img/googleJson/img5.png" width="100%">

我們可以透過Json Editor Online令資料顯示得更加直觀

><a href="http://jsoneditoronline.org/">Json Editor Online</a>

將所有資料copy後，貼到左側，再按下「Copy > 」的按鈕
<img src="/img/googleJson/img6.png" width="100%">

我們可以rows中包含了的內容就是儲存了小精靈資料的Array

>下一個教程將建立一個接收以上資料的 iOS 範例。
