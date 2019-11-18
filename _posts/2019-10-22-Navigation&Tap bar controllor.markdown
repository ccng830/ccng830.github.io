---
layout:     post
title:      "簡易新增Navigation Controllor 以及 Tap Bar Controllor"
date:       2019-10-22 23:55:00
author:     "cc"
header-img: "img/tag-bg.jpg"
catalog: true
tags:
    - swift
---
## 目標

>建立一個具有3個Bar Button 的介面<br/>
>

## 簡易新增Navigation 及 Tap Bar Controllor

首先在Storyboard中點選中要新增Navigation Controllor 的ViewControllor，按上方的Editor -> Embed in -> Navigation Controllor，就會在ViewControllor前方新增一頁Navigation Controllor。

<img src="/img/n&t_controllor/ntstep1.gif" width="100%">

點選Navigation Controllor，按上方的Editor -> Embed in -> Tap bar Controllor，就會在Navigation Controllor前方新增一頁Navigation Controllor。

<img src="/img/n&t_controllor/ntstep2.gif" width="100%">

接下來新增多一頁ViewControllor，點選Tap bar Controllor，按著control拖動至新增的ViewControllor，選取relationship segue中的view controllors。

<img src="/img/n&t_controllor/ntstep3.gif" width="100%">

如上述步驟產生多一頁的ViewControllor，得到如下視圖

<img src="/img/n&t_controllor/view.png" width="100%">

至於每一個Tap bar icon可以先點選要修改的Tap bar icon並在右邊Bar Item中修改。

<img src="/img/n&t_controllor/ntstep4.gif" width="100%">


    
    
    

