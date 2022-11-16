## tcpdump 示例

### hello world

```bash
# host
tcpdump -i any host info.cern.ch

# dst 目标端口
tcpdump -i any dst port 80

# src 出口端口
```

```log
20:19:59.573153 IP ppup.41970 > webafs706.cern.ch.http: Flags [S], seq 3210861026, win 64240, options [mss 1460,sackOK,TS val 264322131 ecr 0,nop,wscale 7], length 0
20:19:59.787386 IP ppup.41970 > webafs706.cern.ch.http: Flags [.], ack 1662193913, win 502, options [nop,nop,TS val 264322346 ecr 496757330], length 0
20:19:59.787599 IP ppup.41970 > webafs706.cern.ch.http: Flags [P.], seq 0:76, ack 1, win 502, options [nop,nop,TS val 264322346 ecr 496757330], length 76: HTTP: GET / HTTP/1.1
20:20:00.009294 IP ppup.41970 > webafs706.cern.ch.http: Flags [.], ack 1, win 502, options [nop,nop,TS val 264322567 ecr 496757544,nop,nop,sack 1 {879:880}], length 0
20:20:00.009328 IP ppup.41970 > webafs706.cern.ch.http: Flags [.], ack 880, win 501, options [nop,nop,TS val 264322568 ecr 496757545], length 0
20:20:00.009796 IP ppup.41970 > webafs706.cern.ch.http: Flags [F.], seq 76, ack 880, win 501, options [nop,nop,TS val 264322568 ecr 496757545], length 0
```

### 以 ASCII 格式打印捕获的数据包 

```bash
# -A
tcpdump -i any host info.cern.ch -A
```

```log
19:53:49.159413 IP ppup.40944 > webafs706.cern.ch.http: Flags [S], seq 4279055153, win 64240, options [mss 1460,sackOK,TS val 262751718 ecr 0,nop,wscale 7], length 0
E..<.'@.@...
......l...P..31...................
..E.........
19:53:49.374286 IP webafs706.cern.ch.http > ppup.40944: Flags [S.], seq 3913703825, ack 4279055154, win 28960, options [mss 1460,sackOK,TS val 495186947 ecr 262751718,nop,wscale 7], length 0
E..<..@.$......l
....P...Fa...32..q uv.........
......E.....
19:53:49.374360 IP ppup.40944 > webafs706.cern.ch.http: Flags [.], ack 1, win 502, options [nop,nop,TS val 262751933 ecr 495186947], length 0
E..4.(@.@...
......l...P..32.Fa............
..F.....
19:53:49.374499 IP ppup.40944 > webafs706.cern.ch.http: Flags [P.], seq 1:77, ack 1, win 502, options [nop,nop,TS val 262751933 ecr 495186947], length 76: HTTP: GET / HTTP/1.1
E....)@.@..d
......l...P..32.Fa......\.....
..F.....GET / HTTP/1.1
Host: info.cern.ch
User-Agent: curl/7.58.0
Accept: */*


19:53:49.587362 IP webafs706.cern.ch.http > ppup.40944: Flags [.], ack 77, win 227, options [nop,nop,TS val 495187160 ecr 262751933], length 0
E..4.J@.$......l
....P...Fa...3~...........
......F.
19:53:49.590085 IP webafs706.cern.ch.http > ppup.40944: Flags [P.], seq 1:879, ack 77, win 227, options [nop,nop,TS val 495187161 ecr 262751933], length 878: HTTP: HTTP/1.1 200 OK
E...j.@.$..n...l
....P...Fa...3~...........
......F.HTTP/1.1 200 OK
Date: Wed, 16 Nov 2022 11:53:49 GMT
Server: Apache
Last-Modified: Wed, 05 Feb 2014 16:00:31 GMT
ETag: "286-4f1aadb3105c0"
Accept-Ranges: bytes
Content-Length: 646
Connection: close
Content-Type: text/html

<html><head></head><body><header>
<title>http://info.cern.ch</title>
</header>

<h1>http://info.cern.ch - home of the first website</h1>
<p>From here you can:</p>
<ul>
<li><a href="http://info.cern.ch/hypertext/WWW/TheProject.html">Browse the first website</a></li>
<li><a href="http://line-mode.cern.ch/www/hypertext/WWW/TheProject.html">Browse the first website using the line-mode browser simulator</a></li>
<li><a href="http://home.web.cern.ch/topics/birth-web">Learn about the birth of the web</a></li>
<li><a href="http://home.web.cern.ch/about">Learn about CERN, the physics laboratory where the web was born</a></li>
</ul>
</body></html>

19:53:49.590136 IP ppup.40944 > webafs706.cern.ch.http: Flags [.], ack 879, win 501, options [nop,nop,TS val 262752148 ecr 495187161], length 0
E..4.*@.@...
......l...P..3~.Fe............
..G.....
19:53:49.590151 IP webafs706.cern.ch.http > ppup.40944: Flags [F.], seq 879, ack 77, win 227, options [nop,nop,TS val 495187161 ecr 262751933], length 0
E..4k.@.$......l
....P...Fe...3~...........
......F.
19:53:49.590554 IP ppup.40944 > webafs706.cern.ch.http: Flags [F.], seq 77, ack 880, win 501, options [nop,nop,TS val 262752149 ecr 495187161], length 0
E..4.+@.@...
......l...P..3~.Fe............
..G.....
19:53:49.803266 IP webafs706.cern.ch.http > ppup.40944: Flags [.], ack 78, win 227, options [nop,nop,TS val 495187375 ecr 262752149], length 0
E..4..@.$......l
....P...Fe...3......i.....
......G.

```

### 不将 ip（即主机地址等）转换为名称

```bash
# -n
tcpdump -i any host info.cern.ch -n
```

```log
19:58:26.671917 IP 10.16.0.182.41114 > 188.184.21.108.80: Flags [S], seq 1599962585, win 64240, options [mss 1460,sackOK,TS val 263029230 ecr 0,nop,wscale 7], length 0
19:58:26.888685 IP 188.184.21.108.80 > 10.16.0.182.41114: Flags [S.], seq 1186268774, ack 1599962586, win 28960, options [mss 1460,sackOK,TS val 495464446 ecr 263029230,nop,wscale 7], length 0
19:58:26.888733 IP 10.16.0.182.41114 > 188.184.21.108.80: Flags [.], ack 1, win 502, options [nop,nop,TS val 263029447 ecr 495464446], length 0
19:58:26.888970 IP 10.16.0.182.41114 > 188.184.21.108.80: Flags [P.], seq 1:77, ack 1, win 502, options [nop,nop,TS val 263029447 ecr 495464446], length 76: HTTP: GET / HTTP/1.1
19:58:27.106964 IP 188.184.21.108.80 > 10.16.0.182.41114: Flags [.], ack 77, win 227, options [nop,nop,TS val 495464663 ecr 263029447], length 0
19:58:27.108875 IP 188.184.21.108.80 > 10.16.0.182.41114: Flags [P.], seq 1:879, ack 77, win 227, options [nop,nop,TS val 495464664 ecr 263029447], length 878: HTTP: HTTP/1.1 200 OK
19:58:27.108903 IP 10.16.0.182.41114 > 188.184.21.108.80: Flags [.], ack 879, win 501, options [nop,nop,TS val 263029667 ecr 495464664], length 0
19:58:27.108918 IP 188.184.21.108.80 > 10.16.0.182.41114: Flags [F.], seq 879, ack 77, win 227, options [nop,nop,TS val 495464664 ecr 263029447], length 0
19:58:27.109192 IP 10.16.0.182.41114 > 188.184.21.108.80: Flags [F.], seq 77, ack 880, win 501, options [nop,nop,TS val 263029667 ecr 495464664], length 0
19:58:27.325639 IP 188.184.21.108.80 > 10.16.0.182.41114: Flags [.], ack 78, win 227, options [nop,nop,TS val 495464883 ecr 263029667], length 0
```

### and

```bash
# and
tcpdump -n -i any src 10.16.0.182 and dst port 80
```

```log
20:05:54.537215 IP 10.16.0.182.41410 > 188.184.21.108.80: Flags [S], seq 4147315514, win 64240, options [mss 1460,sackOK,TS val 263477095 ecr 0,nop,wscale 7], length 0
20:05:54.752231 IP 10.16.0.182.41410 > 188.184.21.108.80: Flags [.], ack 1536620541, win 502, options [nop,nop,TS val 263477310 ecr 495912301], length 0
20:05:54.752434 IP 10.16.0.182.41410 > 188.184.21.108.80: Flags [P.], seq 0:76, ack 1, win 502, options [nop,nop,TS val 263477311 ecr 495912301], length 76: HTTP: GET / HTTP/1.1
20:05:54.969090 IP 10.16.0.182.41410 > 188.184.21.108.80: Flags [.], ack 1, win 502, options [nop,nop,TS val 263477527 ecr 495912517,nop,nop,sack 1 {879:880}], length 0
20:05:54.970104 IP 10.16.0.182.41410 > 188.184.21.108.80: Flags [.], ack 880, win 501, options [nop,nop,TS val 263477528 ecr 495912517], length 0
20:05:54.970381 IP 10.16.0.182.41410 > 188.184.21.108.80: Flags [F.], seq 76, ack 880, win 501, options [nop,nop,TS val 263477529 ecr 495912517], length 0
```

### 将原始数据包写入文件

```bash
# -w
tcpdump -n -i any src 10.16.0.182 and dst port 80 -w capture_file
```

从文件中读取数据包

```bash
# -r
tcpdump -r capture_file 
```

```log
reading from file capture_file, link-type LINUX_SLL (Linux cooked)
20:06:57.194198 IP ppup.41448 > webafs706.cern.ch.http: Flags [S], seq 2532163436, win 64240, options [mss 1460,sackOK,TS val 263539752 ecr 0,nop,wscale 7], length 0
20:06:57.407968 IP ppup.41448 > webafs706.cern.ch.http: Flags [.], ack 1364714852, win 502, options [nop,nop,TS val 263539966 ecr 495974956], length 0
20:06:57.408113 IP ppup.41448 > webafs706.cern.ch.http: Flags [P.], seq 0:76, ack 1, win 502, options [nop,nop,TS val 263539966 ecr 495974956], length 76: HTTP: GET / HTTP/1.1
20:06:57.623942 IP ppup.41448 > webafs706.cern.ch.http: Flags [.], ack 1, win 502, options [nop,nop,TS val 263540182 ecr 495975171,nop,nop,sack 1 {879:880}], length 0
20:06:57.625515 IP ppup.41448 > webafs706.cern.ch.http: Flags [.], ack 880, win 501, options [nop,nop,TS val 263540184 ecr 495975171], length 0
20:06:57.625771 IP ppup.41448 > webafs706.cern.ch.http: Flags [F.], seq 76, ack 880, win 501, options [nop,nop,TS val 263540184 ecr 495975171], length 0

```

### 端口范围

```bash
# portrange
tcpdump -n -i any src 10.16.0.182 and dst portrange 80-90
```

```log
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
20:11:24.872942 IP 10.16.0.182.41624 > 188.184.21.108.80: Flags [S], seq 778654023, win 64240, options [mss 1460,sackOK,TS val 263807431 ecr 0,nop,wscale 7], length 0
20:11:25.086901 IP 10.16.0.182.41624 > 188.184.21.108.80: Flags [.], ack 1812479679, win 502, options [nop,nop,TS val 263807645 ecr 496242632], length 0
20:11:25.087055 IP 10.16.0.182.41624 > 188.184.21.108.80: Flags [P.], seq 0:76, ack 1, win 502, options [nop,nop,TS val 263807645 ecr 496242632], length 76: HTTP: GET / HTTP/1.1
20:11:25.303906 IP 10.16.0.182.41624 > 188.184.21.108.80: Flags [.], ack 879, win 501, options [nop,nop,TS val 263807862 ecr 496242848], length 0
20:11:25.304120 IP 10.16.0.182.41624 > 188.184.21.108.80: Flags [F.], seq 76, ack 879, win 501, options [nop,nop,TS val 263807862 ecr 496242848], length 0
20:11:25.304863 IP 10.16.0.182.41624 > 188.184.21.108.80: Flags [.], ack 880, win 501, options [nop,nop,TS val 263807863 ecr 496242848], length 0

```

### 打印每个数据包的头部等数据

```bash
# -X
tcpdump -X -n -i any src 10.16.0.182 and dst port 80 
```

```log
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
20:15:57.544614 IP 10.16.0.182.41814 > 188.184.21.108.80: Flags [S], seq 4007826866, win 64240, options [mss 1460,sackOK,TS val 264080103 ecr 0,nop,wscale 7], length 0
	0x0000:  4500 003c b148 4000 4006 ac89 0a10 00b6  E..<.H@.@.......
	0x0010:  bcb8 156c a356 0050 eee2 95b2 0000 0000  ...l.V.P........
	0x0020:  a002 faf0 dd18 0000 0204 05b4 0402 080a  ................
	0x0030:  0fbd 8ae7 0000 0000 0103 0307            ............
20:15:57.757579 IP 10.16.0.182.41814 > 188.184.21.108.80: Flags [.], ack 350341011, win 502, options [nop,nop,TS val 264080316 ecr 496515301], length 0
	0x0000:  4500 0034 b149 4000 4006 ac90 0a10 00b6  E..4.I@.@.......
	0x0010:  bcb8 156c a356 0050 eee2 95b3 14e1 c793  ...l.V.P........
	0x0020:  8010 01f6 dd10 0000 0101 080a 0fbd 8bbc  ................
	0x0030:  1d98 38e5                                ..8.
20:15:57.757814 IP 10.16.0.182.41814 > 188.184.21.108.80: Flags [P.], seq 0:76, ack 1, win 502, options [nop,nop,TS val 264080316 ecr 496515301], length 76: HTTP: GET / HTTP/1.1
	0x0000:  4500 0080 b14a 4000 4006 ac43 0a10 00b6  E....J@.@..C....
	0x0010:  bcb8 156c a356 0050 eee2 95b3 14e1 c793  ...l.V.P........
	0x0020:  8018 01f6 dd5c 0000 0101 080a 0fbd 8bbc  .....\..........
	0x0030:  1d98 38e5 4745 5420 2f20 4854 5450 2f31  ..8.GET./.HTTP/1
	0x0040:  2e31 0d0a 486f 7374 3a20 696e 666f 2e63  .1..Host:.info.c
	0x0050:  6572 6e2e 6368 0d0a 5573 6572 2d41 6765  ern.ch..User-Age
	0x0060:  6e74 3a20 6375 726c 2f37 2e35 382e 300d  nt:.curl/7.58.0.
	0x0070:  0a41 6363 6570 743a 202a 2f2a 0d0a 0d0a  .Accept:.*/*....
20:15:57.978531 IP 10.16.0.182.41814 > 188.184.21.108.80: Flags [.], ack 879, win 501, options [nop,nop,TS val 264080537 ecr 496515515], length 0
	0x0000:  4500 0034 b14b 4000 4006 ac8e 0a10 00b6  E..4.K@.@.......
	0x0010:  bcb8 156c a356 0050 eee2 95ff 14e1 cb01  ...l.V.P........
	0x0020:  8010 01f5 dd10 0000 0101 080a 0fbd 8c99  ................
	0x0030:  1d98 39bb                                ..9.
20:15:57.978803 IP 10.16.0.182.41814 > 188.184.21.108.80: Flags [F.], seq 76, ack 880, win 501, options [nop,nop,TS val 264080537 ecr 496515516], length 0
	0x0000:  4500 0034 b14c 4000 4006 ac8d 0a10 00b6  E..4.L@.@.......
	0x0010:  bcb8 156c a356 0050 eee2 95ff 14e1 cb02  ...l.V.P........
	0x0020:  8011 01f5 dd10 0000 0101 080a 0fbd 8c99  ................
	0x0030:  1d98 39bc                                ..9.

```

### 打印系统上可用的网络接口列表

```bash
tcpdump -D
```

```log
1.ens160 [Up, Running]
2.any (Pseudo-device that captures on all interfaces) [Up, Running]
3.lo [Up, Running, Loopback]
4.nflog (Linux netfilter log (NFLOG) interface)
5.nfqueue (Linux netfilter queue (NFQUEUE) interface)
```

### 更详细的输出

```bash
tcpdump -vvv -A -n -i any host info.cern.ch
```

```log
tcpdump: listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
20:31:21.326903 IP (tos 0x0, ttl 64, id 35705, offset 0, flags [DF], proto TCP (6), length 60)
    10.16.0.182.42398 > 188.184.21.108.80: Flags [S], cksum 0xdd18 (incorrect -> 0x1095), seq 3017036436, win 64240, options [mss 1460,sackOK,TS val 265003885 ecr 0,nop,wscale 7], length 0
E..<.y@.@..X
......l...P..R....................
...m........
20:31:21.538313 IP (tos 0x0, ttl 36, id 33538, offset 0, flags [DF], proto TCP (6), length 60)
    188.184.21.108.80 > 10.16.0.182.42398: Flags [S.], cksum 0x5c20 (correct), seq 2700029494, ack 3017036437, win 28960, options [mss 1460,sackOK,TS val 497439080 ecr 265003885,nop,wscale 7], length 0
E..<..@.$......l
....P.....6..R...q \ .........
..Qh...m....
20:31:21.538362 IP (tos 0x0, ttl 64, id 35706, offset 0, flags [DF], proto TCP (6), length 52)
    10.16.0.182.42398 > 188.184.21.108.80: Flags [.], cksum 0xdd10 (incorrect -> 0xf942), seq 1, ack 1, win 502, options [nop,nop,TS val 265004097 ecr 497439080], length 0
E..4.z@.@.._
......l...P..R....7...........
...A..Qh
20:31:21.538491 IP (tos 0x0, ttl 64, id 35707, offset 0, flags [DF], proto TCP (6), length 128)
    10.16.0.182.42398 > 188.184.21.108.80: Flags [P.], cksum 0xdd5c (incorrect -> 0xaa9d), seq 1:77, ack 1, win 502, options [nop,nop,TS val 265004097 ecr 497439080], length 76: HTTP, length: 76
	GET / HTTP/1.1
	Host: info.cern.ch
	User-Agent: curl/7.58.0
	Accept: */*
	
E....{@.@...
......l...P..R....7.....\.....
...A..QhGET / HTTP/1.1
Host: info.cern.ch
User-Agent: curl/7.58.0
Accept: */*


20:31:21.750224 IP (tos 0x0, ttl 36, id 56093, offset 0, flags [DF], proto TCP (6), length 52)
    188.184.21.108.80 > 10.16.0.182.42398: Flags [.], cksum 0xf935 (correct), seq 1, ack 77, win 227, options [nop,nop,TS val 497439292 ecr 265004097], length 0
E..4..@.$......l
....P.....7..R......5.....
..R<...A
20:31:21.752279 IP (tos 0x0, ttl 36, id 56322, offset 0, flags [DF], proto TCP (6), length 930)
    188.184.21.108.80 > 10.16.0.182.42398: Flags [P.], cksum 0x7db2 (correct), seq 1:879, ack 77, win 227, options [nop,nop,TS val 497439293 ecr 265004097], length 878: HTTP, length: 878
	HTTP/1.1 200 OK
	Date: Wed, 16 Nov 2022 12:31:21 GMT
	Server: Apache
	Last-Modified: Wed, 05 Feb 2014 16:00:31 GMT
	ETag: "286-4f1aadb3105c0"
	Accept-Ranges: bytes
	Content-Length: 646
	Connection: close
	Content-Type: text/html
	
	<html><head></head><body><header>
	<title>http://info.cern.ch</title>
	</header>
	
	<h1>http://info.cern.ch - home of the first website</h1>
	<p>From here you can:</p>
	<ul>
	<li><a href="http://info.cern.ch/hypertext/WWW/TheProject.html">Browse the first website</a></li>
	<li><a href="http://line-mode.cern.ch/www/hypertext/WWW/TheProject.html">Browse the first website using the line-mode browser simulator</a></li>
	<li><a href="http://home.web.cern.ch/topics/birth-web">Learn about the birth of the web</a></li>
	<li><a href="http://home.web.cern.ch/about">Learn about CERN, the physics laboratory where the web was born</a></li>
	</ul>
	</body></html>
E.....@.$..i...l
....P.....7..R.....}......
..R=...AHTTP/1.1 200 OK
Date: Wed, 16 Nov 2022 12:31:21 GMT
Server: Apache
Last-Modified: Wed, 05 Feb 2014 16:00:31 GMT
ETag: "286-4f1aadb3105c0"
Accept-Ranges: bytes
Content-Length: 646
Connection: close
Content-Type: text/html

<html><head></head><body><header>
<title>http://info.cern.ch</title>
</header>

<h1>http://info.cern.ch - home of the first website</h1>
<p>From here you can:</p>
<ul>
<li><a href="http://info.cern.ch/hypertext/WWW/TheProject.html">Browse the first website</a></li>
<li><a href="http://line-mode.cern.ch/www/hypertext/WWW/TheProject.html">Browse the first website using the line-mode browser simulator</a></li>
<li><a href="http://home.web.cern.ch/topics/birth-web">Learn about the birth of the web</a></li>
<li><a href="http://home.web.cern.ch/about">Learn about CERN, the physics laboratory where the web was born</a></li>
</ul>
</body></html>

20:31:21.752297 IP (tos 0x0, ttl 64, id 35708, offset 0, flags [DF], proto TCP (6), length 52)
    10.16.0.182.42398 > 188.184.21.108.80: Flags [.], cksum 0xdd10 (incorrect -> 0xf3df), seq 77, ack 879, win 501, options [nop,nop,TS val 265004310 ecr 497439293], length 0
E..4.|@.@..]
......l...P..R...1............
......R=
20:31:21.752312 IP (tos 0x0, ttl 36, id 56323, offset 0, flags [DF], proto TCP (6), length 52)
    188.184.21.108.80 > 10.16.0.182.42398: Flags [F.], cksum 0xf5c5 (correct), seq 879, ack 77, win 227, options [nop,nop,TS val 497439293 ecr 265004097], length 0
E..4..@.$......l
....P....1...R............
..R=...A
20:31:21.752531 IP (tos 0x0, ttl 64, id 35709, offset 0, flags [DF], proto TCP (6), length 52)
    10.16.0.182.42398 > 188.184.21.108.80: Flags [F.], cksum 0xdd10 (incorrect -> 0xf3dc), seq 77, ack 880, win 501, options [nop,nop,TS val 265004311 ecr 497439293], length 0
E..4.}@.@..\
......l...P..R...1............
......R=
20:31:21.963230 IP (tos 0x0, ttl 36, id 15741, offset 0, flags [DF], proto TCP (6), length 52)
    188.184.21.108.80 > 10.16.0.182.42398: Flags [.], cksum 0xf41a (correct), seq 880, ack 78, win 227, options [nop,nop,TS val 497439505 ecr 265004311], length 0
E..4=}@.$.<]...l
....P....1...R............
..S.....

```

## 参考

- https://gist.github.com/tuxfight3r/9ac030cb0d707bb446c7 tcp flags
