---
layout:     post
title:      "CocoaMqtt 安裝教學"
date:       2019-10-20 11:00:00 
author:     "cc"
header-img: "img/tag-bg.jpg"
catalog: true
tags:
    - swift
    - MQTT
---
## 安裝CocoaPod

我們首先要安裝CocoaPod，然後利用CocoaPod下載CocoaMQTT。

終端機 terminal 中輸入

    sudo gem install cocoapods
    
<img src="/img/cocoamqtt/terminal.png" width="65%">
等待數分鐘後安裝完畢。


## 安裝CocoaMQTT

打開Xcode，開一個新的專案，本次例子中取名為CocoaPod_test，並存於documents中。

新增專案時，User Interface點選Storyboard

<img src="/img/cocoamqtt/storyboard.png" width="100%">

建好後將它關掉，可利用快捷鍵 Command + Q關掉。回到終端機 terminal，使用cd(change directory) 指令以導航至Xcode 專案的資料夾中。以本例子為例，本例子存於documents中，因此在終端機 terminal輸入：

    cd ~/documents/CocoaPod_test/

然後，我們需要在專案的根目錄下新增一個 Podfile 文件，被安裝的 pods 都會寫入至 Podfile 以作紀錄及追踪更新。日後當你呼叫 CocoaPods 去安裝或更新現存的 pods，CocoaPods 便會前往 Podfile 搜找指令。
要建立 Podfile 是相當簡單，在終端機 terminal輸入：

    pod init

運行完我們可以看到專案資料夾中，多了一個Podfile

<img src="/img/cocoamqtt/podfile.png" width="100%">

雙擊打開Podfile輸入，在use_framework!下一行加入

    use_framework!
    pod 'CocoaMQTT'

<img src="/img/cocoamqtt/pod_cocoamqtt.png" width="100%">

儲存並關閉，在terminal中輸入：

    pod install

<img src="/img/cocoamqtt/pod_install.png" width="100%">

安裝完畢後，打開專案的資料夾，會發現多了一個CocoaPod_test.xcworkspace 的檔案，雙擊打開

<img src="/img/cocoamqtt/xcws.png" width="100%">
<img src="/img/cocoamqtt/xcws2.png" width="100%">

點選ViewControllor.swift，現在在專案中可以使用CocoaMQTT module了。

    import CocoaMQTT

<img src="/img/cocoamqtt/cocoamqtt.png" width="100%">




    
    
    

