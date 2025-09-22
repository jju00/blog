# solve

- nmap 시 21,22,80 포트 열려있음
```
─$ sudo nmap -sS -Pn -n --open 10.10.146.204 -sV -sC -p 21,22,80                                                                    
Host is up (0.40s latency).
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:10.4.15.138
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-rw-r--    1 1000     1000            0 Mar 12  2023 hiya
|_-rw-r--r--    1 0        0              45 Mar 12  2023 temporary_pw.txt
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 12:8a:28:dd:91:3a:41:fd:af:6f:3a:3c:4c:43:ea:5c (RSA)
|   256 ab:36:d4:c2:a3:43:88:87:d0:89:2e:bb:b2:ce:03:51 (ECDSA)
|_  256 3a:e5:21:03:c4:de:b9:53:1a:c6:a9:66:cb:ea:1f:1b (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
| http-robots.txt: 1 disallowed entry 
|_/dev/
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.63 seconds
```

- ftp 익명 로그인 가능
```
└─$ ftp 10.10.146.204                     
Connected to 10.10.146.204.
220 (vsFTPd 3.0.3)
Name (10.10.146.204:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
```

- 현재 디렉터리에서 파일 읽기
```
ftp> ls -al                      
229 Entering Extended Passive Mode (|||64461|)
150 Here comes the directory listing.
drwxrwxr-x    2 1000     1000         4096 Mar 12  2023 .
drwxrwxr-x    2 1000     1000         4096 Mar 12  2023 ..
-rwxrw-r--    1 1000     1000         7173 Mar 12  2023 .ssh_creds.docx
-rw-rw-r--    1 1000     1000            0 Mar 12  2023 hiya
-rw-r--r--    1 0        0              45 Mar 12  2023 temporary_pw.txt
```
숨김 docx파일 하나랑 다른 두 파일이 보임. 일단 로컬로 옮겨서 확인
```
ftp> get .ssh_creds.docx
local: .ssh_creds.docx remote: .ssh_creds.docx
229 Entering Extended Passive Mode (|||61562|)
150 Opening BINARY mode data connection for .ssh_creds.docx (7173 bytes).
100% |*****************************************************************************************|  7173      511.04 KiB/s    00:00 ETA
226 Transfer complete.
7173 bytes received in 00:00 (16.58 KiB/s)
```
```
ftp> get temporary_pw.txt
local: temporary_pw.txt remote: temporary_pw.txt
229 Entering Extended Passive Mode (|||59722|)
150 Opening BINARY mode data connection for temporary_pw.txt (45 bytes).
100% |**********************|    45       63.59 KiB/s    00:00 ETA
226 Transfer complete.

─$ cat temporary_pw.txt 
100% |**********************|    45       63.59 KiB/s    00:00 ETA │Do you see a docx file ? Read the docx file.
```
텍스트 파일은 단순히 힌트 파일. docx파일 텍스트 편집기로 읽어보면 xml 파일들 있음
![[Pasted image 20250922232415.png]]

- creator랑 ssh 비번 발견
core.xml에서 creator 이름 발견
![[Pasted image 20250922232312.png]]
document.xml에서 ssh 비밀번호 발견 
```
`... Encoded password just in case Y2F0Y2htZSFAIw==`
```
base64인코딩 풀어보면 ssh 비밀번호 알 수 있음
![[Pasted image 20250922232931.png]]

- ssh harry로 접속 성공
```
find -name "user.txt"
```
![[Pasted image 20250922233347.png]]
이후 user.txt파일에서 플래그 발견 

- find로 flag.txt찾아도 안 나옴. 일단 sudo 설정된 바이너리로 gtfobins에서 명령어 보고 권한상승
![[Pasted image 20250922234127.png]]
다시 루트 쉘에서 find 쳐보면 `/root/flag.txt`에 있다. 열어보면 플래그가 base64 인코딩 되어있는데 base64가 suid 설정되어 있어서 그거로 읽거나 아니면 그냥 디코딩하기 