---
title: SSH 安全性設定筆記
tags:
  - ssh
  - security
  - note
date: 2012-05-18 14:16:00
---


以下收錄可增加 ssh 安全性的各種技巧，主要的修改都在 `/etc/ssh/sshd_config` 中，同時，修改設定後別忘了重新載入設定值或重新啟動 sshd。

* 修改預設 port (可用多行開啟多個 port)

		Port <port>

* 僅監聽特定 ip (適用於多網卡/多 IP 的情形)

		ListenAddress 192.168.1.10

* 禁止 root 登入

		PermitRootlogin no

	管理者必須先以個人帳號登入，再 su 成 root，或利用 sudo 工作。

* 禁止使用空密碼登入

		PermitEmptyPasswords no

<!--more-->

* 僅允許或拒絕特定帳號或群組登入

		AllowUsers <user1> <user2> <user3>
		AllowGroups <group>
		DenyUsers *
		DenyGroups no-ssh

	根據實驗，對於同一帳號而言，如果同時 Allow 跟 Deny 的話，結果會是 Deny 的。

* 廢除密碼登錄，強迫使用 RSA/DSA 驗證

		RSAAuthentication yes
		PubkeyAuthentication yes
		AuthorizedKeysFile %h/.ssh/authorized_keys
		PasswordAuthentication no

	並確保 user 的 `~/.ssh` 權限為 `700`，同時將該 user 的 public key 加入其 `~/.ssh/authorized_keys` 中。Public key 的產生方式可搜尋 `ssh-keygen`。

* 僅允許 SSHv2

		Protocol 2

* 限制特定使用者、群組、主機或位址的登入行為，這裡以限制 `somebody` 與 `handsomebody` 不可使用密碼登入為例

		Match User somebody,handsomebody
		PasswordAuthentication no

	但是要怎麼結束 Match block？我只找到有資料說「In sshd_config, Match blocks must be located at the end of the file.」，如果真是這樣，那就只好認了。

* 使用 TCP wrappers 限制來源 IP

		# /etc/hosts.deny
		sshd: ALL

		# /etc/hosts.allow
		sshd: 192.168.1 1.2.3.4 # 僅允許 192.168.1.* 與 1.2.3.4 連線

* 使用 iptables 限制來源 IP

		# iptables -A INPUT -p tcp -m state --state NEW --source 1.2.3.4 --dport 22 -j ACCEPT
		# iptables -A INPUT -p tcp --dport 22 -j DROP

	設定會立即生效，但若希望重開機後還能保存，需要手動儲存 iptables 的設定。

* 時間鎖定 (這段直接引用並稍微修改[原作者](http://os.51cto.com/art/200803/68174_2.htm)的文字，有空再消化整理)

> 你可以使用不同的 iptables 參數來限制到 SSH 服務的連接，讓其在一個特定的時間範圍內可以連接，其他時間不能連接。你可以在下面的任何例子中使用 `/second`、`/minute`、`/hour` 或 `/day` 開關。
> 第一個例子，如果一個用戶輸入了錯誤的密碼，鎖定一分鐘內不允許在訪問 SSH 服務，這樣每個用戶在一分鐘內只能嘗試一次登錄
>
>		# iptables -A INPUT -p tcp -m state --syn --state NEW --dport 22 -m limit --limit 1/minute --limit-burst 1 -j ACCEPT
>		# iptables -A INPUT -p tcp -m state --syn --state NEW --dport 22 -j DROP
> 
> 第二個例子，設置 iptables 只允許主機 193.180.177.13 連接到 SSH 服務，在嘗試三次失敗登錄後，iptables 允許該主機每分鐘嘗試一次登錄
>
>		# iptables -A INPUT -p tcp -s 193.180.177.13 -m state --syn --state NEW --dport 22 -m limit --limit 1/minute --limit-burst 1 -j ACCEPT
>		# iptables -A INPUT -p tcp -s 193.180.177.13 -m state --syn --state NEW --dport 22 -j DROP

* 檢查相關檔案權限，不安全則不允許登入

		StrictModes yes

	某些相關檔案權限設定若有錯誤時，可能造成安全性風險。如使用者的 ~/.ssh/authorized_keys 權限若為 666，可能造成其他人可以盜用帳號。

* 自訂使用者登入時顯示的 banner (話說這跟安全性有什麼關係...? 大概可以用社交方式嚇跑壞人吧...= =a)

		Banner /etc/ssh/banner # 任意文字檔

* 限制 su/sudo 名單

		# vi /etc/pam.d/su
		auth       required     /lib/security/$ISA/pam_wheel.so use_uid

		# visudo
		%wheel  ALL = (ALL) ALL

		# gpasswd -a user1 wheel

* 限制 ssh 使用者名單

		# vi /etc/pam.d/sshd
		auth required pam_listfile.so item=user sense=allow file=/etc/ssh_users onerr=fail
		# echo USERNAME >> /etc/ssh_users

## 參考文章列表

* [網絡技術日誌 - SSH 安全設定](http://www.hkcode.com/linux-bsd-notes/176)
* [SSH 的一些安全小技巧 by Netman](http://www.study-area.org/tips/ssh_tips.htm) (後半段有個有趣的應用技，讓人透過 web 去暫時允許某 ip 登入 ssh，期限 5 分鐘!)
* [高级 SSH 安全技巧](http://os.51cto.com/art/200803/68174.htm)
* [鳥哥的 Linux 私房菜](http://linux.vbird.org/linux_server/0310telnetssh.php#ssh_sshdconfig)
* [Force SSH public key authentication for specific users](http://serverfault.com/questions/238711/force-ssh-public-key-authentication-for-specific-users)
