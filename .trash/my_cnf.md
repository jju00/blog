---
cheat: " "
---
#유저파일/설정 

>`/home/<유저>/.my.cnf`: MySQL이나 MariaDB 클라이언트 설정 파일. `[client]`문에 유저이름과 비밀번호가 하드코딩 되어 있는 경우가 있다. mysql 서버 설정 파일은 `/etc/mysql/my.cof`


**그 외**

1. **호스트 내 파일 읽기**
	-> 읽어볼만한 파일들: [[HTTP LFI 취약점 공격]] 참고
		- 유저들의 `.ssh`나 `.bashrc`에 비밀번호 저장되어 있는 경우 많음: `su - user`로 로그인 시도
	
	- ssh 관련 파일 먼저: 아래는 기본 이름들
		- 개인키: `home/유저/.ssh/id_rsa ` -> 이거 복붙해서 해당 유저로 ssh 로그인
		- 공개키: `home/유저/.ssh/id_rsa.pub`
		
	> *이때, 다운받은 개인 키에 passphrase가 걸려있으면 ssh2john, johntheripper로 크랙 가능*