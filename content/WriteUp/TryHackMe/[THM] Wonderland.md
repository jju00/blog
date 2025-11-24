# solve

- nmap 시 22,80 열려있음

- gobuster 시 `/r`디렉터리 발견. 이걸 따라가다보면 `/r/a/b/b/i/t`까지 페이지가 있다.
![[Pasted image 20250930134014.png]]
![[Pasted image 20250930133817.png]]
![[Pasted image 20250930141504.png]]
![[Pasted image 20250930141525.png]]
![[Pasted image 20250930141545.png]]
![[Pasted image 20250930141600.png]]
![[Pasted image 20250930141939.png]]

- 소스코드에서 숨겨진 key를 발견. alice로 ssh 로그인
```
<p style="display: none;">alice:HowDothTheLittleCrocodileImproveHisShiningTail</p>
```
![[Pasted image 20250930144003.png]]

- root/user.txt 발견
![[Pasted image 20250930154923.png]]

- sudo로 실행가능한 거 확인하니, python3.6과 저 파일을 rabbit의 권한으로 실행 가능
![[Pasted image 20250930144252.png]]

- 현재 디렉터리에는 쓰기 권한 존재
![[Pasted image 20250930150832.png]]

- 저 .py 열어보면 맨 위 import random있음. 따라서 random.py라는 파일을 만들어두고, random 모듈 대신 읽도록 설정. random.py에는 rabbit으로 bash 쉘을 띄우는 내용 넣음 - rabbit권한으로 스크립트가 실행되므로, rabbit 권한으로 rabbit 쉘을 얻기 성공
```
cat > random.py <<'PY'
import pty, os
# 인터랙티브한 bash 얻기
pty.spawn("/bin/bash")
PY
```
![[Pasted image 20250930151918.png]]

- rabbit 폴더 보면 teaParty라는 elf 바이너리가 존재. 뭐 입력해도 seg fault가 남
![[Pasted image 20250930152343.png]]

- kali로 파일 옮겨서 분석. strings 긁어보면 저런 게 있다. 이때, date 실행파일이 절대 경로 없이 실행되기 때문에, path 하이재킹에 취약하다.
![[Pasted image 20250930160510.png]]

- 따라서, date라는 파일을 하나 만들고, 파일 안에는 새로운 권한으로(teaparty가 실행될 때의 EUID 권한) bash쉘을 띄우도록 함 - 즉, 부모 프로세스가 가진 _effective UID(EUID)_ 를 물려받음
```
#! /bin/bash
/bin/bash
```
![[Pasted image 20250930161450.png]]

- hatter로 쉘이 띄워짐. 안에는 hatter의 password 파일이 존재
![[Pasted image 20250930161552.png]]
![[Pasted image 20250930162230.png]]

- capability를 확인 시, perl을 실행가능
![[Pasted image 20250930163538.png]]

- 권한상승 성공
![[Pasted image 20250930164008.png]]







