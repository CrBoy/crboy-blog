---
title: 「cannot open shared object file」的解決方案
tags:
  - Linux
  - Trouble Shooting
  - 教學
date: 2012-05-23 15:20:00
---


有時候執行一些指令，會碰到下面這樣的訊息：

	error while loading shared libraries: libiconv.so.2: cannot open shared object file: No such file or directory

在這個例子中，我們的程式找不到 `libiconv.so.2` 這個 dynamic library。那麼應該怎麼解決呢？

首先必須找到系統中的 `libiconv.so.2`，下面四條指令選一條作就好：

	$ locate libiconv.so.2
	$ whereis libiconv.so.2
	$ find /usr /lib -name libiconv.so.2
	$ find / -name libiconv.so.2 2>/dev/null

這時候就可以找出函式庫位置，例如：`/usr/local/lib/libiconv.so`

<!--more-->

萬一沒有找出來的話，就要自行安裝了。[這篇文章](http://www.techsww.com/tutorials/libraries/libiconv/installation/installing_libiconv_on_ubuntu_linux.php)有教該如何安裝 libiconv，如果是其他函式庫的話可以自行變通。

找到位置之後，應該怎麼做呢？如果我們有 root 權限的話，可以把函式庫所在路徑寫入 `/etc/ld.so.conf`，再重新產生 ld 的 cache 即可，如下：

	# echo "/usr/local/lib" >> /etc/ld.so.conf # 注意! 要用 >> 而非 >！
	# ldconfig

在某些系統中，`/etc/ld.so.conf` 可能會預設加上 `include /etc/ld.so.conf.d/*.conf` 這樣的敘述，那們我們也可以把路徑放在那個目錄下：

	# echo "/usr/local/lib" >> /etc/ld.so.conf.d/usr_local_lib.conf
	# ldconfig

這樣一來就可以讓程式順利搜尋到需要的 library 囉！

**BUT!! 玩 Linux 最重要的就是這個 BUT!!**

如果我們沒有 root 權限的話該怎麼辦呢？這時候只好很可憐的自己搞定了...如果需要自行編譯 library 的話，可以照著類似的步驟做，但是 prefix (也就是安裝位置) 要設定在自己的家目錄下，例如：

	$ ./configure --prefix=$HOME/libiconv

但這邊假設系統中已經有 `/usr/local/lib/libiconv.so.2` 這個檔案，進行以下步驟：

	$ echo "export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH" >> ~/.bashrc

原理很簡單，就是透過設定 `LD_LIBRARY_PATH` 來讓程式找得到需要的 library 檔案。修改後需要重新登入才會生效喔！

但是...基本上還是建議不要用後面這個方法啦，找你的系統管理員協助設定 `/etc/ld.so.conf` 才是好主意。這裡有篇文章：[Why LD_LIBRARY_PATH is bad](http://xahlee.org/UnixResource_dir/_/ldpath.html)，專門在討論為什麼不要隨意使用 `LD_LIBRARY_PATH`。
