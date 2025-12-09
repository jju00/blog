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



vmware pro로 진행하였고, 위의 사진과 같이 한 대는 windows 11 iso를 다운받아 windows victim host를 구성. 다른 한 대는 탐지결과를 분석하기 위한 Elastic Stack서버를 구성하였다. 







### 레퍼런스

- https://www.레드팀.com/homelab/edr
- https://otterhacker.github.io/Malware/EDR/Elastic%20EDR.html


