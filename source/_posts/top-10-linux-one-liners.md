---
title: 10 個最酷的 Linux 單行指令
tags:
  - Linux
date: 2010-04-06 02:10:00
---


本文內容修改自 [http://linuxtoy.org/archives/top-10-one-liners.html](http://linuxtoy.org/archives/top-10-one-liners.html)

下面是來自 [Commandlinefu](http://commandlinefu.com/) 網站由用戶投票表決出的 10 個最酷的 Linux 單行指令。

1\. `sudo !!`
以 root 帳戶執行上一道指令。

2\. `python -m SimpleHTTPServer`
利用 Python 建立一個簡單的 Web 伺服器，可透過 `http://$HOSTNAME:8000` 瀏覽。

3\. `:w !sudo tee %`
在 vim 中無需權限儲存編輯的文件。

4\. `cd -`
更改到上一次瀏覽的目錄。

5\. `^foo^bar`
將上一道指令中的 foo 替換為 bar，並執行。

6\. `cp filename{,.bak}`
快速備份或複製文件。

7\. `mtr google.com`
`traceroute` + `ping`。

8\. `!whatever:p`
搜尋命令歷史，但不執行。

9\. `ssh-copy-id user@host`
將 ssh keys 複製到 `user@host` 以啟用無密碼 SSH 登錄。

10\. `ffmpeg -f x11grab -s wxga -r 25 -i :0.0 -sameq /tmp/out.mpg`
把 Linux 桌面錄製為影片。
