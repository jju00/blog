# solve

1. 머신 연결 확인
	![[Pasted image 20250630091212.png]]

2. nmap 스캔
	![[Pasted image 20250630091149.png]]
	-> `sC`로 준 스크립트가 robots.txt에서 admin.html이 disallowed된 페이지라는 것을 발견

3. admin.html로 접속
	![[Pasted image 20250630091321.png]]
	-> 페이지를 보면 XXXX.hv.html을 찾아보라고 한다. XXXX는 랜덤 4자리 숫자
	![[Pasted image 20250630091505.png]]
	-> 따라서 0000 ~ 9999까지 중에서 존재하는 페이지를 찾아야 한다 
		= 디렉터리 브루트포싱 진행 

4. 디렉터리 브루트포싱 - wordlist 생성
	-> gobuster 툴 이용. 워드리스트가 필요하기 때문에 수동으로 생성한다.
	```
	for i in {0000..9999} ; do echo $i.hv.html >> wordlist.txt; do
	```
	를 통해서 스크립팅 하여 0000~9999까지 XXXX.hv.html 형식으로 저장된 10000개의 문자열을 wordlist로 저장한다.
	![[Pasted image 20250630092401.png]]

5. 디렉터리 브루트포싱 - gobuster 실행
	```bash
	gobuster dir -u 10.10.46.128 -w wordlist.txt
	```
	![[Pasted image 20250630092827.png]]
	-> 하나가 200ok로 나온다.
	![[Pasted image 20250630092806.png]]
	-> 하지만 들어가보면 아무것도 안 뜨는데, 이때 `ctrl + u`로 소스코드를 한 번 보기
	![[Pasted image 20250630093215.png]]
	-> 그러면 문제의 답인 secret 유저의 비밀번호와 flag를 얻게 된다.

6. 파일시스템에 숨겨진 플래그 찾기
	우선, 아까 정보수집에서 확인한 열린 포트는 22,80 2개였고 22가 ssh니, 얻은 secret 계정으로 ssh에 로그인
	![[Pasted image 20250630093543.png]]
	![[Pasted image 20250630094013.png]]
	![[Pasted image 20250630100159.png]]
	-> roothint 열어보기. 이후 `/etc/apache2/secrets.txt`의 flag 파일 열기