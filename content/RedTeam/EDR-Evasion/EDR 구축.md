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

## Elastic EDR

> **Elastic EDR**: Elastic사에서 무료로 배포하는 자사의 EDR 솔루션. SIEM도 무료로 배포하고 있다.

### 요구사항

- VMware, ESXi, VirtualBox 등의 하이퍼바이저
- Ubuntu, Debian, Kali Linux 등의 가상머신 1대
- 윈도우 가상머신 1대

|역할|OS|용도|
|---|---|---|
|Elastic Stack 서버|Ubuntu 22.04|Elasticsearch + Kibana + Fleet Server|
|Victim PC|Windows 10/11|Elastic Agent 설치. EDR 테스트 대상|

### 설치







### 레퍼런스

- https://www.레드팀.com/homelab/edr
- https://otterhacker.github.io/Malware/EDR/Elastic%20EDR.html


