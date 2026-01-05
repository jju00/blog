# solve

- nmap 시 80 하나가 끝
```
─$ nmap -sV -sC --open 10.49.135.241 -oA tcpdetailed
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-05 13:27 EST
Nmap scan report for 10.49.135.241
Host is up (0.16s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Werkzeug httpd 2.0.3 (Python 3.10.2)
|_http-title: Corridor
|_http-server-header: Werkzeug/2.0.3 Python/3.10.2

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.84 seconds
```

- 웹 사이트 들어가니 이상한 방 사진이 존재. 소스코드 확인 시 해시로 추정되는 문자열들이 존재함.
![[Pasted image 20260106034359.png]]

- 위 문자열들만 해시파일로 저장한 후, hashcat을 이용해 크랙한 결과
```
hashcat -m 0 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt
```
```
└─$ hashcat -m 0 --show hashes.txt
c4ca4238a0b923820dcc509a6f75849b:1
c81e728d9d4c2f636f067f89cc14862c:2
eccbc87e4b5ce2fe28308fd9f2a7baf3:3
a87ff679a2f3e71d9181a67b7542122c:4
e4da3b7fbbce2345d7772b0674a318d5:5
1679091c5a880faf6fb5e6087eb1b2dc:6
8f14e45fceea167a5a36dedd4bea2543:7
c9f0f895fb98ab9159f51fd0297e236d:8
45c48cce2e2d7fbdea1afc51c7c6ad26:9
d3d9446802a44259755d38e6d163e820:10
6512bd43d9caa6e02c990b0a82652dca:11
c20ad4d76fe97759aa27a0c99bff6710:12
c51ce410c124a10e0db5e4b97fc2af39:13
```

- 1 ~ 13까지 있음. 따라서 0을 md5로 암호화한 후에 엔드포인트명으로 입력해서 접속
```
└─(03:44:39)──> echo -n "0" | md5sum
cfcd208495d565ef66e7dff9f98764da  -
```

- 플래그 획득
![[Pasted image 20260106034733.png]]