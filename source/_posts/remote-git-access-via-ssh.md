---
title: 透過 ssh 遠端存取 git repository
tags:
  - Git
  - Linux
  - 心得
  - 教學
date: 2012-03-23 03:31:00
---


好，今天來寫個我架設「偽 - git server」的作法吧！  

為什麼叫做「偽 - git server」呢？因為他不是真的跑起來一支 daemon 去負責 git 的 access，像是 gitolite 或是 gitosis 那樣。(事實上我也不知道該怎麼把那些東西架起來XD)  
由於 git 可以透過 ssh protocol 來 access，當然要善用這點！  

以下我分為單人各自使用與多人共用來講：  

<!--more-->

單人各自使用
------------

這算是簡單的應用了，只要你有一台工作站的 ssh 帳號，而且工作站上有 git (好吧，我承認我不曉得 minimal requirement 是什麼，也許是 git-shell 吧？) 的話，就可以這樣用，相當簡單！  

首先假設遠端 server/workstation 叫做 Remote，網址是 example.com，我在這台機器上的帳號為 crboy，家目錄是 /home/crboy。相當單純而常見的配置。另外本機叫做 PC，其他相同。  

那麼我們想要開始一個新專案可以這麼做：(注意主機名稱)  

	crboy@Remote:~$ mkdir my_project.git # 建立專案 repo 的目錄
	crboy@Remote:~$ cd my_project.git # 進去該目錄
	crboy@Remote:~/my_project.git$ git init --bare # 初始化這個目錄為「純repo」(這是我自己的說法，應該是不太通用，記得 bare repository 比較好)
	Initialized empty Git repository in /home/crboy/my_project.git/
	crboy@Remote:~/my_project.git$ ls # 只是看看底下產生了什麼
	branches  config  description  HEAD  hooks  info  objects  refs
	crboy@Remote:~/my_project.git$

回到本機上  

	crboy@PC:~$ git clone ssh://example.com/home/crboy/my_project.git # 把剛剛產生的空 repo 給 clone 回來，由於是空的，會收到警告
	Cloning into my_project...warning: You appear to have cloned an empty repository.
	crboy@PC:~$ cd my_project
	crboy@PC:~/my_project$ vim README # 隨便寫點東西吧
	....... # 這邊就不說了，基本的 git 操作，總之你就 commit 一些東西就對了
	crboy@PC:~/my_project$ git push origin master # 第一次 push 要指定遠端主機名稱還有 branch，之後直接 git push 即可
	Counting objects: 3, done
	Writing objects: 100% (3/3), 218 bytes, done.
	Total 3 (delta 0), reused 0 (delta 0)
	To ssh://example.com/home/crboy/my_project.git
	* [new branch]      master -> master
	crboy@PC:~/my_project$ 

去遠端主機上看看  

	crboy@Remote:~/my_project.git$ git log # 應該會看到你的 log 唷!!
	commit cbaf3fe2ab3bb5477a8f05ff79512dd6930e6177
	Author: CrBoy <email@is.hidden>
	Date:   Fri Mar 23 02:41:52 2012 +0800

		Initial commit
	crboy@Remote:~/my_project.git$

成功了！  

那麼，如果是已經有一個現有的 git repository，想要把他丟到遠端去呢？我是透過 git-remote 來幫忙。首先我們假設遠端已經開好一個空的 repository 了，名字跟剛剛一樣。在本機我們這麼做：  

	crboy@PC:~$ cd existing_project
	crboy@PC:~/existing_project$ git remote add origin ssh://example.com/home/crboy/my_project.git # 加入一台遠端主機，名字為 origin
	crboy@PC:~/existing_project$ git push origin master # push 的方式跟之前一樣
	Counting objects: 3, done.
	Writing objects: 100% (3/3), 218 bytes, done.
	Total 3 (delta 0), reused 0 (delta 0)
	To ssh://example.com/home/crboy/my_project.git
	* [new branch]      master -> master
	crboy@PC:~/existing_project$ 

另外順便提一下，因為剛剛在測試的時候耍蠢，收到以下這個訊息：  

	fatal: '/home/crboy/my_project.git' does not appear to be a git repository
	fatal: The remote end hung up unexpectedly

如果你跟我一樣，請你不要驚慌，因為這只是你忘記把遠端 repo 的那個目錄 initialize 了.....XD  

有沒有發現上面我根本沒有提到要安裝什麼東西，還是要架什麼 server？因為他真的不需要....只要本來 ssh 是會通的，git 可以用，那上面的動作就都沒問題了(吧XD)。  

但是，有時候我們還是得跟別人合作 project 呀！如果有特殊考量，不能用 Github 或是 bitbucket.org 這類服務的話，又該怎麼辦呢？我這邊有個不怎麼完美的作法，但至少是堪用了....  

多人共用
--------

多人要共用就複雜多了，因為牽扯到權限問題。本來以前的我很天真的要 partner 直接 clone 我的 repo 就好，結果後來要 push 的時候才發現他沒有 writing 權限，沒辦法 push 啊啊啊啊XD 於是乎，我只好把我的 repo，裡面所有檔案的 group 都改成對方的 group，然後再把權限加上 g+w (chmod -R g+w <repo>)........好，他現在可以 push 了。  

結果過沒多久，又發現出包了啊XDDDD 檢查之後才發現，他新增的一些檔案，owner 會是他，所以我沒辦法改寫，我新增的檔案， group 還是我自己，所以他還是不能 push。雖然可以一直狂修權限，但是這根本就是比水溝還髒的解法= =  

於是乎，我去想辦法解決這個問題。  

本來是想架 server 的，不過後來忘記在哪裡看到這個技巧，覺得很方便，又不用多開 daemon，就採用了！運作上還不錯，缺點大概是每次設定上比較麻煩，但是這可以寫支 script 來處理就是XD (根本就是抗拒接受更完整的方案)  

這個技巧，說穿了很簡單。就是「建立公用帳號，並讓每個人都透過公用帳號來 push/pull」。哇靠，這樣不是很不安全嗎？ CrBoy 你這個惡魔怎麼可以介紹這種鳥方法！呃....各位先冷靜，我再說明得詳細一點。  

這個方法要：  

* 建立公用帳號 git #當然你可以叫他別的名字啦
* 設定 git 的 login shell 為 git-shell
* 在 ~git/.ssh/ 底下新增 authorized_keys
* 把所有需要 git remote access 的人的 public key 都放進 ~git/.ssh/authorized_keys

大概這樣吧.....如果對於 ssh-key 沒有概念的話，呃...可能要請你看看網路上的其他文章了...很簡單的，連我都會 :)

因為我實在懶得重做一次了，所以請各位看倌照著上面的提示做吧！只要對 linux 跟 ssh 有點概念跟使用經驗的話，應該是沒問題的！做完之後，就可以在 git 這個帳號的家目錄底下建立 repo 了，要建幾個都可以XD。至於 clone 或是 push/pull 的方法，都跟上面單人使用的時候類似，唯一不同的是多人都可以存取同一個 repo 而且不會發生鬧鬼的問題唷！另外補充一下，連線的時候要指定 username 為 git 唷~當然 ssh 的 key 也得設定好才行！

但這個方法是有缺點的，其中幾個很明顯的缺點，如：

*   不能控制誰可以存取哪個 repo
*   沒有 web 介面可以看到 repo 資訊
*   要新增 user 需要請管理者更新 authorized_keys
*   要新增 repo 需要請管理者協助
*   只能走 ssh，不能走像是 http, https, 或是 git 之類的 protocol。很多人是不知道怎麼用 ssh 的....orz

所以這個 solution 只適合用在小型團隊，很少開專案或是加減人的狀況囉！

對了，抱怨一下，我實在受不了 blogger 寫技術文章真的有夠麻煩.....快崩潰了= =
