#wargame 

# solve

1. 정보 수집
![[Pasted image 20250725175041.png]]


2. 취약점 진단, 공격
-> 디렉터리 브루트포싱
![[Pasted image 20250725175844.png]]
![[Pasted image 20250725175809.png]]
php니까 php 리버스쉘 이용
![[Pasted image 20250725180126.png]]
테스트해보니까 png가 업로드 가능
![[Pasted image 20250725182642.png]]
-> 확장자랑 content-type 변경 후 업로드 성공. 이후 포트 열고 `../uploads/php-reverse-shell.php5`로 가면 리버스쉘이 세션을 붙인다. 
![[Pasted image 20250725183807.png]]
-> 플래그


3. 권한 상승
![[Pasted image 20250725183430.png]]
![[Pasted image 20250725184051.png]]
![[Pasted image 20250725184105.png]]
-> 그 중에서, GTFO에서 SUID가 있으면 악용 가능한 바이너리가 python이라 이걸 이용

```
/usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```
![[Pasted image 20250725184605.png]]
-> 권한 상승 성공
![[Pasted image 20250725184638.png]]