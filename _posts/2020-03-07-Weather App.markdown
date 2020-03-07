---
layout:     post
title:      "串接天氣API，開發一款根據天氣狀況實時切換背景的天氣App (1)"
date:       2020-03-07
author:     "cc"
header-img: "img/tag-bg.jpg"
catalog: true
tags:
    - API
---
## 簡介

開發前我們先觀看一下成品，
<img src="/img/weatherApp/Weather App testing.gif" width="60%">

演示如下，在頁面上方顯示地點，溫度，天氣狀況等基本資訊，下方是一個TableView，顯示濕度，日出/日落時間等資訊，並且下拉能夠更新整個。
由於該App是根據天氣變化而切換背景，因此右上角的按鈕是用以測試其他不同背景用的，按下後會彈出一個列表以供切換背景。

## OpenWeatherMap API

本次作品串接的API為OpenWeatherMap API，注冊後即可使用免費版本，
><a href="https://openweathermap.org/api">https://openweathermap.org/api</a>

如下圖所示，本次使用了Current weather data，
<img src="/img/weatherApp/1.png" width="100%">

其使用方法是：
>http://api.openweathermap.org/data/2.5/weather?q={地方}&appid={自己的Key}

注冊後可以如下圖般查看到自己的Key
<img src="/img/weatherApp/2.png" width="100%">

先利用Jsoneditor觀察一下API的資料，得到如下圖般所示，
><a href="http://jsoneditoronline.org/">http://jsoneditoronline.org/</a>
<img src="/img/weatherApp/3.png" width="100%">

由於該API是免費版，所以只提供較少的資料，但作為練習已經很足夠了！
我們所需要的資料分別是
weather內的id，description；
main內的temp，feels_like，temp_max，temp_min，humidity；分別是現時，體感，最高，最低溫及濕度
name； 地方名
sys內的sunrise，sunset； 日出時間，日落時間

取得id，就能判斷現時的大致情況，參考網站內的文件，如下圖所示
><a href="https://openweathermap.org/weather-conditions">https://openweathermap.org/weather-conditions</a>
<img src="/img/weatherApp/4.png" width="100%">

可以知道2xx的表示雷暴，3xx及5xx都表示下雨，6xx為下雪等等，因此我們可以根據ID的首位，來判斷現時大致情況，用以控制背景變化。

而其它的data利用如下圖所示：
<img src="/img/weatherApp/5.png" width="100%">

大致分析完API的data後，我們下一節就開始Swift部分。

