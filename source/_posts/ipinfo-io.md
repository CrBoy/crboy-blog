---
title: 發現一個滿方便的 ip 資訊查詢：ipinfo.io
date: 2016-09-27 18:45:49
tags:
---

以前都用 orange.tw 在查 ip，雖然自己也寫了一個 ip.crboy.net，不過因為實在不想花錢養一台機器，所以就把服務給停掉了。

剛剛發現 ipinfo.io 這個網站，用瀏覽器開的話相關說明也都有了。但如果直接用 `curl` 送 request 的話，就可以拿到自己的 ip 的相關資訊（包含地點、國家等）。

```
$ curl -D- ipinfo.io
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Tue, 27 Sep 2016 10:33:36 GMT
Server: nginx/1.8.1
Set-Cookie: first_referrer=; Path=/
X-Content-Type-Options: nosniff
Content-Length: 219
Connection: keep-alive

{
  "ip": "119.14.25.111",
  "hostname": "host-111.25-14-119.dynamic.totalbb.net.tw",
  "city": "Taipei",
  "region": "",
  "country": "TW",
  "loc": "25.0392,121.5250",
  "org": "AS9416 Hoshin Multimedia Center Inc."
}
```

也可以查詢特定 ip 的資訊：

```
$ curl -D- ipinfo.io/8.8.8.8
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Tue, 27 Sep 2016 10:36:40 GMT
Server: nginx/1.8.1
Set-Cookie: first_referrer=; Path=/
X-Content-Type-Options: nosniff
Content-Length: 160
Connection: keep-alive

{
  "ip": "8.8.8.8",
  "hostname": "No Hostname",
  "city": "",
  "region": "",
  "country": "US",
  "loc": "37.7510,-97.8220",
  "org": "AS15169 Google Inc."
}
```

甚至 fetch 特定欄位資料：

```
$ curl -D- ipinfo.io/8.8.8.8/country
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Content-Type: text/html; charset=utf-8
Date: Tue, 27 Sep 2016 10:37:18 GMT
Server: nginx/1.8.1
Set-Cookie: first_referrer=; Path=/
Content-Length: 3
Connection: keep-alive

US
```
```
$ curl -D- ipinfo.io/country
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Content-Type: text/html; charset=utf-8
Date: Tue, 27 Sep 2016 10:37:22 GMT
Server: nginx/1.8.1
Set-Cookie: first_referrer=; Path=/
Content-Length: 3
Connection: keep-alive

TW
```

還有一些好玩的....像是：

```
$ curl -D- ipinfo.io/127.0.0.1
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Tue, 27 Sep 2016 10:39:36 GMT
Server: nginx/1.8.1
Set-Cookie: first_referrer=; Path=/
X-Content-Type-Options: nosniff
Content-Length: 37
Connection: keep-alive

{
  "ip": "127.0.0.1",
  "bogon": 1
}
```

```
$ curl -D- ipinfo.io/255.255.255.255
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Tue, 27 Sep 2016 10:40:46 GMT
Server: nginx/1.8.1
Set-Cookie: first_referrer=; Path=/
X-Content-Type-Options: nosniff
Content-Length: 72
Connection: keep-alive

{
  "ip": "255.255.255.255",
  "hostname": "No Hostname",
  "bogon": 1
}
```

`bogon: 1` 根據[維基百科上 Bogon filtering](https://en.wikipedia.org/wiki/Bogon_filtering) 的描述，看來是有擋掉不應該被查詢的 ip。不過....

```
$ curl -D- ipinfo.io/192.168.1.1
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Tue, 27 Sep 2016 10:39:56 GMT
Server: nginx/1.8.1
Set-Cookie: first_referrer=; Path=/
X-Content-Type-Options: nosniff
Content-Length: 184
Connection: keep-alive

{
  "ip": "192.168.1.1",
  "hostname": "ip-192-168-1-1.ap-southeast-1.compute.internal",
  "city": "Momoyama-cho",
  "region": "Kyoto",
  "country": "JP",
  "loc": "34.9500,135.7830"
}
```

看起來還是有些特別的 ip 可以查XD 原來這台主機放在 AWS EC2 的日本機房嗎......（思）
