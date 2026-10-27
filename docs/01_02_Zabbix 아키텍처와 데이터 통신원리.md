
# Zabbix 프로젝트 학습

## 1장 2교시 — Zabbix 아키텍처와 데이터 통신 원리

### 1. Zabbix 아키텍처

Zabbix는 여러 구성요소가 서로 통신하면서 서버 상태를 수집하고 장애를 탐지하는 구조이다.

| 구성요소 | 역할 |
|---|---|
| Zabbix Server | 데이터 수집·처리, 장애 판단, 알림 처리 |
| Zabbix Agent 2 | 모니터링 대상 서버의 데이터 수집 |
| Database | 설정, 수집 데이터, 이벤트 저장 |
| Web Frontend | 웹 기반 모니터링 및 관리 화면 |
| Zabbix Proxy | 중간에서 데이터를 수집하고 Server에 전달(선택사항) |

**기본 아키텍처**

```text
WEB01 ─ Agent 2 ─┐
                 │
WAS01 ─ Agent 2 ─┼── Zabbix Server ── Database
                 │          │              │
DB01  ─ Agent 2 ─┘          │              │
                            │              │
                         알림 처리     Web Frontend
                            │
                            v
                       Discord 알림
```

※ Agent와 Server 사이의 데이터 통신 방향은 Passive/Active 방식에 따라 달라진다.

---

### 2. 각 구성요소의 동작 원리

**① Zabbix Agent 2**
- 모니터링 대상 서버에 설치하는 프로그램
- CPU, 메모리, 디스크, 네트워크 등의 정보 수집
- Passive 및 Active 방식 지원
- 플러그인을 활용해 MySQL 등 다양한 서비스 모니터링 가능

**② Zabbix Server**
- 모니터링 데이터를 중앙에서 수집하고 처리
- Trigger 조건 평가
- 장애 이벤트 생성
- 설정된 Action에 따라 알림 처리

**③ Database**
- Host, Item, Trigger 등의 설정 저장
- 수집한 측정값(History) 저장
- 장기 집계 데이터(Trends) 저장
- 장애 이벤트 및 사용자 정보 저장

**④ Web Frontend**
- 웹브라우저에서 Zabbix 관리
- Host, Item, Trigger 설정
- 그래프 및 대시보드 조회
- 장애 상태 및 알림 설정 확인

---

### 3. Passive Check vs Active Check

#### Passive Check

Zabbix Server가 Agent에 데이터를 요청하는 방식이다.

```text
Zabbix Server ── 요청 ──> Zabbix Agent
              TCP 10050

Zabbix Server <── 응답 ── Zabbix Agent
```

**특징**
- Server가 Agent에 연결을 시작
- Agent의 기본 수신 포트는 TCP 10050
- Server에서 Agent로 접근할 수 있어야 함
- 방화벽에서 해당 통신을 허용해야 함

#### Active Check

Agent가 Server에 연결하여 수집 항목을 받아온 후 데이터를 수집·전송하는 방식이다.

```text
Zabbix Agent ── 접속 ──> Zabbix Server
              TCP 10051

1. Agent가 수집할 Item 목록 요청
2. Agent가 데이터 수집
3. Agent가 Server로 데이터 전송
```

**특징**
- Agent가 Server에 연결을 시작
- Server의 기본 수신 포트는 TCP 10051
- Agent에서 Server로 접근 가능해야 함
- Agent 측 인바운드 포트 개방을 줄일 수 있음

**비교**

| 구분 | Passive | Active |
|---|---|---|
| 연결 시작 | Server | Agent |
| 기본 수신 포트 | Agent 10050 | Server 10051 |
| 통신 방향 | Server → Agent | Agent → Server |
| 주요 특징 | Server가 직접 요청 | Agent가 수집하여 전송 |

---

### 4. Zabbix 주요 네트워크 포트

| 포트 | 서비스 | 용도 |
|---|---|---|
| TCP 10050 | Zabbix Agent | Passive 요청 수신 |
| TCP 10051 | Zabbix Server | Active 데이터 수신 등 |
| TCP 80 | HTTP | 웹 관리 화면 |
| TCP 443 | HTTPS | 암호화된 웹 관리 화면 |
| TCP 3306 | MySQL | Database 연결 |

**보안 주의사항**
- 필요한 서버 간 통신만 방화벽에서 허용
- Zabbix 포트를 인터넷 전체에 공개하지 않기
- DB와 Zabbix Server가 같은 VM에 있으면 로컬 DB 연결 사용 가능

---

### 5. Zabbix Agent 2 주요 설정

**설정 파일 경로**

```bash
/etc/zabbix/zabbix_agent2.conf
```

**설정 파일 편집**

```bash
sudo nano /etc/zabbix/zabbix_agent2.conf
```

**설정 예시**

```ini
Server=192.0.2.10
ServerActive=192.0.2.10
Hostname=WEB01
ListenPort=10050
```

※ 192.0.2.10은 설명용 IP이며 실제 구축 시 Zabbix Server IP를 입력한다.

| 설정 | 의미 |
|---|---|
| Server | Passive 요청을 허용할 Server/Proxy 주소 |
| ServerActive | Active 통신 대상 Server/Proxy 주소 |
| Hostname | Agent의 호스트 식별 이름 |
| ListenPort | Passive 요청 수신 포트 |

**중요:** Active Check에서는 Agent의 Hostname이 Zabbix Web UI에 등록된 기술적 Host name과 일치해야 한다.

---

### 6. Linux 네트워크 확인 명령어

```bash
# 호스트 이름 확인
hostname

# IP 및 네트워크 인터페이스 확인
ip -br addr

# 라우팅 테이블 확인
ip route

# TCP LISTEN 포트 확인
sudo ss -lntp

# 특정 포트 확인
sudo ss -lntp | grep ':10050 '

# Agent 서비스 상태 확인 (설치 후)
sudo systemctl status zabbix-agent2

# Agent 서비스 로그 확인 (설치 후)
sudo journalctl -u zabbix-agent2 -n 50 --no-pager
```

**ss 주요 옵션**

| 옵션 | 의미 |
|---|---|
| -l | LISTEN 상태 |
| -n | IP·포트 숫자 표시 |
| -t | TCP |
| -p | 관련 프로세스 표시 |

---

### 7. 장애 대응 방법

**상황: Zabbix Server에서 WEB01 데이터를 수집하지 못함**

가능한 원인:
1. Zabbix Agent 서비스 중지
2. Agent의 TCP 10050 포트 미수신
3. 잘못된 서버 IP 또는 라우팅
4. 방화벽에서 통신 차단
5. Agent 설정 오류
6. Active Check의 Hostname 불일치

**장애 확인 순서**

```text
IP 주소 확인
      ↓
라우팅 확인
      ↓
방화벽 및 통신 경로 확인
      ↓
포트 LISTEN 상태 확인
      ↓
Agent 서비스 상태 확인
      ↓
Agent 설정 파일 확인
      ↓
로그 분석 및 원인 조치
      ↓
데이터 수집 정상화 검증
```

**대표 오류**

| 오류 | 주요 확인 사항 |
|---|---|
| Connection refused | 서비스 실행 여부, 포트 수신 상태 |
| Connection timed out | 네트워크 경로, 방화벽, 응답 상태 |
| Host not found | 호스트 식별 설정, 이름 해석 등 |

---

### 8. 면접 예상 질문

**Q1. Zabbix Server와 Agent의 차이는 무엇인가요?**

Agent는 모니터링 대상 서버에 설치되어 CPU, 메모리, 디스크 등의 데이터를 수집하는 역할을 합니다.

Server는 이러한 데이터를 중앙에서 수집·처리하고 Trigger 조건을 평가하여 장애를 감지하고 알림을 처리합니다.

**Q2. Passive와 Active 방식의 차이는 무엇인가요?**

Passive 방식은 Zabbix Server가 Agent의 TCP 10050 포트로 접속하여 데이터를 요청하는 방식입니다.

Active 방식은 Agent가 Server의 TCP 10051 포트로 접속하여 수집할 항목을 받아온 뒤, 데이터를 수집해 전송하는 방식입니다.

네트워크 접근 정책과 운영 환경에 따라 적절한 방식을 선택합니다.

---

### 9. 핵심 암기 포인트

1. **Server:** 중앙 데이터 수집·처리, 장애 판단, 알림
2. **Agent 2:** 모니터링 대상의 데이터 수집
3. **Database:** 설정, 측정값, 이벤트 저장
4. **Web Frontend:** 웹에서 설정 및 모니터링
5. **Passive:** Server → Agent, TCP 10050
6. **Active:** Agent → Server, TCP 10051
7. **Server=** Passive 요청 허용 대상
8. **ServerActive=** Active 통신 대상
9. **Hostname=** Agent 호스트 식별 이름

---

**학습 진도: 1장 2교시 — Zabbix 아키텍처와 데이터 통신 원리**

<img width="1448" height="1086" alt="image" src="https://github.com/user-attachments/assets/dc90dde4-54f0-40a6-abf9-f2920a9d4cf7" />
