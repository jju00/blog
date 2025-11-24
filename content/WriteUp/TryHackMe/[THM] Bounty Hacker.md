# solve

- nmap 정보 수집 시 21, 22, 80 열려있음
![[Pasted image 20251013003229.png]]

- ftp 익명 로그인 허용
![[Pasted image 20251013003153.png]]

- ftp파일들 옮기기 권한 존재. locks.txt 열어보면 비밀번호 사전으로 보임
![[Pasted image 20251013003511.png]]

- task.txt 열어보면 lin이라는 유저 이름을 알 수 있음.
![[Pasted image 20251013010208.png]]

- lin 유저의 비밀번호를 locks.txt 사전으로 ssh 브루트포스 후 발견
```
hydra -l lin -P locks.txt ssh://10.201.71.112 -V -s 22
```
![[Pasted image 20251013010151.png]]

- lin으로 로그인 후 user.txt 열어서 첫 번째 플래그 획득
![[Pasted image 20251013010342.png]]

- `sudo -l`시 tar 바이너리를 sudo로 실행할 수 있다는 걸 확인
![[Pasted image 20251013010603.png]]

- 실행하여 root 권한 획득 및 root.txt읽어서 플래그 획득
![[Pasted image 20251013010648.png]]




