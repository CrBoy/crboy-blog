---
title: 神奇的 Git Subtree
tags:
  - Git
date: 2016-09-29 03:01:12
---


一直以來只知道 git 可以透過 git-submodule 管理巢狀 git 專案。也就是一個大專案中有小專案，而這兩個專案要分別用 git 做版本控制的狀況。這會發生在專案有可以抽出來跟其他專案共用的部分，或是專案需要引入一個外部現有的 git 專案的時候。舉例來說，我的 vimrc 裡面有 snippets，這個 snippets 是值得被另外拉出來成為一個專案的。又或是像 hexo，我要自訂 theme，所以我 fork 了 icarus 這個 theme 來改，但需要把它放進我的大 hexo 專案中。

用 submodule 的做法其實滿麻煩的，它的原理是：大專案紀錄某目錄是一個 submodule，所以大專案就不 track 該目錄的變更，而是 track 該目錄的 HEAD commit。該目錄的內容變更則交給他自己 track。也就是說，如果修改了小專案的內容，需要先去小專案裡面把修改 commit 進去後，再去大專案把小專案的新 commit id 給 commit 進去。光用講的都覺得混亂...

有天發現了 git-subtree 是個可以解決類似問題的工具，但差別在於 subtree 不需要事先指定哪個目錄是子專案，只要在需要的時候處理即可。什麼是需要的時候呢？

1. 要把子專案切出去的時候（通常只會切一次）
2. 要把子專案在本地端的變更丟出去的時候
3. 要把一個專案放進來變成子專案的時候（通常只會放一次）
4. 要把子專案在外面的變更拉進來的時候

除了「需要的時候」以外，子專案就像是（其實根本就是）大專案的一部分，用原有的方式去操作 git 即可，相當單純！

我覺得以我找到的資料來說，[Git SubTree 共編 Library](http://yutin.logdown.com/posts/188306-git-subtree-total-addendum-library) 這篇文章是講得比較清楚易懂的，有些別的文章會舉例錯誤或不知所云....XD 不過我認為需要搭配 [man page](https://github.com/git/git/blob/master/contrib/subtree/git-subtree.txt) 服用會比較能理解。

下面就以 hexo 為例，練習把原本用 git-submodule 管理的 theme（crboy-icarus）改成用 git-subtree 管理，並推到相同的 remote repository 去吧。

首先把 theme 的 origin 記下來，然後刪除這個 submodule

```
### 記下原有的 remote，方便後面操作（非必要）
$ cat .gitmodules
[submodule "themes/crboy-icarus"]
    path = themes/crboy-icarus
	url = https://github.com/CrBoy/hexo-theme-icarus

### 移除 submodule 並 commit
$ git submodule deinit themes/crboy-icarus
$ git rm themes/crboy-icarus
$ git commit

### 將原有的 remote 加入，並新增為 subtree
$ git remote add theme-crboy-icarus https://github.com/CrBoy/hexo-theme-icarus
$ git subtree add -P themes/crboy-icarus theme-crboy-icarus master
```

完成之後，原本的 submodule 現在已經成為上層專案的一部份了，任何修改都可以直接 commit 進上層專案就好。如果要把變更送到 remote theme-crboy-icarus 去，可以這樣做：

```
$ git subtree push -P themes/crboy-icarus theme-crboy-icarus master
```

如果這個子專案在別地方被更新了，那可以這樣把更新拉進來：

```
$ git subtree pull -P themes/crboy-icarus theme-crboy-icarus master
```

總結一下：用 subtree 當然是也有一些缺點的，例如子專案的東西會在母專案裡也存一份，所以會讓 git repo 變大。但這問題在現代來說不是很嚴重，整體而言我認為 subtree 比 submodule 方便得多，我會傾向使用 subtree。至於 submodule 的使用情境，目前只想到要 include 一個外部專案，而且我確定完全不會去改它的時候才比較可能使用吧。
