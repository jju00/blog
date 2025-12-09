# EDR 이란?

> **EDR** (Endpoint Detection and Response) : 엔드포인트에서의 탐지 및 대응 솔루션

- 대상 엔드포인트에 전용 소프트웨어(=에이전트)를 도입하여 활동 로그를 상시 취득
	-> 엔드포인트 상에 **멀웨어나 랜섬웨어에 의한 수상한 움직임이 없는지** 항상 감시
- EDR 서버는 이 로그 데이터를 취합 및 정리/분석하고, 의심스러운 행동의 흔적이 발견되면 즉시 관리자에게 이 사실을 통보

## 특징

- <u>실시간</u> 위협 탐지 및 모니터링
- <u>자동화</u>된 대응 및 차단 
- <u>행위 기반</u> 수집 
	- 단순 시그니처 기반 x
	- `winword.exe → powershell.exe → certutil.exe` 이런 체인
	- MITRE ATT&CK TTP 기반 룰 적용
- 대응
	- 프로세스 kill
	- 엔드포인트 네트워크 격리
	- 파일 격리
<br>
## Elastic EDR

> **Elastic EDR**: Elastic사에서 무료로 배포하는 자사의 EDR 솔루션. SIEM도 무료로 배포하고 있다.

### 구성요소

| 구성 요소            | 역할                                | 한 줄 요약               |
| ---------------- | --------------------------------- | -------------------- |
| *Elastic Defend* | 엔드포인트에서 위협 탐지, 차단, 로그·프로세스 이벤트 수집 | “EDR 엔진”             |
| *Elastic Agent*  | Defend를 포함한 모든 Elastic 수집기 통합     | “Endpoint Installer” |
| *Fleet Server*   | Agent들을 중앙에서 관리/정책 배포             | “MDR/관리 콘솔”          |
| *Kibana App*     | EDR 탐지/분석/대시보드·타임라인               | “콘솔 UI”              |
| *Elasticsearch*  | 이벤트 저장·인덱싱·검색                     | “데이터 레이크”            |
1. 엔드포인트에 **Elastic Agent**가 설치됨
2. 그 안에 활성화된 **Elastic Defend** 모듈이 실제 EDR 역할을 수행
3. **Defend**는 프로세스/파일/메모리/네트워크 이벤트를 감시하고 악성 행위를 탐지, 차단
4. 이렇게 수집된 이벤트와 경고는 **Fleet** 서버를 통해 중앙으로 전달됨
5. 모든 데이터는 **Elasticsearch**에 저장·인덱싱됨
6. **Kibana**의 Security 콘솔을 통해 이벤트 분석, 타임라인 시각화 등 UI에서 분석할 수 있다.
<br>
### 요구사항

- VMware, ESXi, VirtualBox 등의 하이퍼바이저
- Ubuntu, Debian, Kali Linux 등의 가상머신 1대
- 윈도우 가상머신 1대

| 역할               | OS            | 구성 및 용도                               |
| ---------------- | ------------- | ------------------------------------- |
| Elastic Stack 서버 | Ubuntu 22.04  | Elasticsearch + Kibana + Fleet Server |
| Victim PC        | Windows 10/11 | Elastic Agent 설치. EDR 테스트 대상          |
<br>
### 설치

> *1. Elastic Cloud의 템플릿대로 설치*
> *2. 로컬 내 가상머신에서 구성*
>    
> 설치 방법은 위와 같이 나눌 수 있는데, 추후 EDR evasion 등 추가 학습을 고려했을 때 로컬 가상머신 내 설치가 자유도 측면에서 이점이 있을 거 같아서 로컬 vm내 설치를 선택하였다.

#### 1. 가상머신 2대 준비

![[Pasted image 20251210031917.png]]
vmware pro로 진행하였고, 위의 사진과 같이 한 대는 windows 11 iso를 다운받아 windows victim host를 구성. 다른 한 대는 탐지결과를 분석하기 위한 Elastic Stack서버를 구성하였다. 

#### 2. Elastic Stack 설치

- 의존성 패키지들을 먼저 설치해준다.
```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
# 도커 설치
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 도커 그룹에 유저 추가 (현재 유저로도 docker 실행하게)
sudo usermod -aG docker $USER
newgrp docker
```
설치 확인
![[Pasted image 20251210041727.png]]

- https://github.com/peasead/elastic-container 프로젝트 클론 후 설치
```bash
cd ~
git clone https://github.com/peasead/elastic-container.git
cd elastic-container
```

- 비밀번호 변경
```bash
sudo vi .env
```
![[Pasted image 20251210042054.png]]
여기 changme라 되어있는 2개를 변경

- 스크립트 실행하여 약 10~15분간 도커 이미지를 다운받고, 도커를 실행한다. 즉, 도커를 통해 Elastic Stack 서버 (Elasticsearch + Kibana +Fleet Server) 를 띄우게 됨.
```bash
sudo ./elastic-container.sh start
```
설치가 성공적으로 끝나면 대충 Kibana가 실행되었다는 알람이 뜬다.
![[Pasted image 20251210043910.png]]

#### 3. Elastic Defend 추가 

- 이제, Windows 호스트에서 우분투 도커로 돌아가는 Kibana에 접속하여 아까 바꾼 비번으로 로그인하였다.
![[Pasted image 20251210044330.png]]

- 왼쪽 메뉴 integrations -> Elastic Defend를 클릭 -> Add Endpoint and cloud security 선택
![[Pasted image 20251210044650.png]]
여기서 이름, 설명은 대충 입력해준다. 2번은 디폴트 상태로 냅둔다.
![[Pasted image 20251210044848.png]]

#### 4. Elastic Agent 설치 및 등록

- Endpoint and Cloud Security 설치가 끝나면 `Add Elastic Agent to your hosts` 버튼을 눌러 Agent 생성과 설정을 진행한다.
![[image 2.png]]

- Fleet server URL 설정을 하였다. 이는 fleet 서버 url이 빈 상태라 Agent가 접속할 URL을 설정해주어야 하는 것이고, `https://ubuntuip:8220`으로 하였다. 포트가 8220인 이는, 도커를 띄울때의 디폴트 fleet이 8220이기 때문.
![[Pasted image 20251210045302.png]]
![[Pasted image 20251210045452.png]]
이때, 저기서 2번 파워쉘 명령어는 치면 안 된다. 왜냐면 ubuntu에서 이미 Elastic stack을 설치해서 fleet이 있는 상탠데, 저거 치면 fleet을 windows에 또 설치하는 거기 때문.
따라서, 저렇게 fleet server host url만 등록한 뒤, 다시 Add agent 창으로 돌아온다.

- Add agent로 다시 돌아와서, Agent 설치를 마무리한다.
![[Pasted image 20251210050530.png]]
여기서 2번에 나오는 파워쉘 명령을 관리자로 실행해준다. 그리고 맨 마지막줄에는 `--insecure` 플래그를 넣어준다.
```powershell
$ProgressPreference = 'SilentlyContinue'
Invoke-WebRequest -Uri https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-9.0.8-windows-x86_64.zip -OutFile elastic-agent-9.0.8-windows-x86_64.zip 
Expand-Archive .\elastic-agent-9.0.8-windows-x86_64.zip -DestinationPath .
cd elastic-agent-9.0.8-windows-x86_64
.\elastic-agent.exe install --url=https://192.168.200.136:8220 --enrollment-token=ZThLcUJKc0JYVDVSODRyQ0NHTGw6Wl9reVdwbEVJUEp0ZE1BZTVuVjk3UQ== --insecure
```
![[Pasted image 20251210052706.png]]
위와 같이 Agent, Defend가 모두 설치된 것을 확인
<br>
## 실행을 통한 EDR 탐지 확인 및 분석

현재는 기본 탐지 룰 세팅이 활성화 되어 있는 상태이다.







### 레퍼런스

- https://www.레드팀.com/homelab/edr
- https://otterhacker.github.io/Malware/EDR/Elastic%20EDR.html


