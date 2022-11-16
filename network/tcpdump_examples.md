## tcpdump 示例

### 以 ASCII 格式打印捕获的数据包 

```bash
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



## 参考

- https://gist.github.com/tuxfight3r/9ac030cb0d707bb446c7 tcp flags
