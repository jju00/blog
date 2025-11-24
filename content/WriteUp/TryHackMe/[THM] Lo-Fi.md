# solve

- nmap 시 22, 80 발견
```
─$ sudo nmap -sV -sC --open -Pn 10.201.114.205 -p 22,80 -oA tcpdetailed

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 77:f8:9b:9c:23:4e:af:43:24:f7:ce:21:8a:d0:f9:20 (RSA)
|   256 ce:59:74:3c:20:04:08:c0:6d:7e:e7:a6:6c:67:2c:10 (ECDSA)
|_  256 1d:76:d0:80:94:c6:eb:b6:86:fb:39:07:66:75:17:27 (ED25519)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Lo-Fi Music
```

- 웹 서버 접속시 무슨 노래 나오는 페이지인데 `page`파라미터가 눈에 띈다.
![[Pasted image 20251120075313.png]]

- 위에서 `page`파라미터는 lfi에 취약.
![[Pasted image 20251120075351.png]]

- 이때, flag는 루트 아래에 있다고 함. 따라서 플래그 파일 이름은 `flag.txt`라는 생각해서 루트 아래에서 LFI로 읽기 성공. 약간 억지스러운 게싱인 거 같기도 함
![[Pasted image 20251120081745.png]]

