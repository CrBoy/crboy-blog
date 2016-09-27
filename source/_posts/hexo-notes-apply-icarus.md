---
title: Hexo 學習筆記 - 套用 Icarus 佈景
date: 2016-03-21 18:04:54
tags:
  - Hexo
---


挑了一下主題，暫時就用 Icarus 這個樣式吧XD

不過在 config 的時候遇到問題，怎麼改都沒效。後來找到[這篇文章](http://wuchong.me/blog/2015/03/12/support-jacman-to-Hexo-3/)，裡面寫到這段話：

> hexo-config 方法会去 theme 主题下的 \_config.yml 中查找参数中的配置项

原來是會讀取 **theme 底下的 `_config.yml`**啊！難怪我改 Hexo 的 `_config.yml`（根目錄那一個）沒辦法起作用。不過奇怪的是，像是 `customize.profile.author` 之類的設定，在 Hexo 的 config 中修改，會起作用。不曉得 Hexo 在這邊是怎麼處理的...

