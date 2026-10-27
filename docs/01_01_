
# Zabbix 프로젝트 학습

## 1장 1교시 — 모니터링 개념과 Zabbix의 필요성

### 1. 모니터링(Monitoring)이란?

서버, 네트워크, 애플리케이션 등의 상태를 지속적으로 수집하고 관찰하여 장애를 조기에 발견하고 대응하는 활동이다.

**모니터링이 필요한 이유**
- 서버 상태를 지속적으로 확인
- CPU, 메모리, 디스크 등 자원 사용량 파악
- 장애 발생 시 신속한 탐지 및 알림
- 과거 데이터를 활용한 장애 원인 분석
- 장애 발생 전 잠재적 문제 발견

**핵심:** 장애가 발생한 후 대응하는 것뿐만 아니라, 문제를 미리 발견하고 예방하기 위해 사용한다.

---

### 2. Zabbix란?

Zabbix는 서버, 네트워크 장비, 애플리케이션 등의 상태를 통합 모니터링하는 오픈소스 솔루션이다.

| 주요 기능 | 설명 |
|---|---|
| 데이터 수집 | CPU, 메모리, 디스크, 서비스 상태 수집 |
| 데이터 저장 | 수집한 데이터를 DB에 저장 |
| 시각화 | 그래프 및 대시보드 제공 |
| 장애 탐지 | Trigger 조건으로 이상 감지 |
| 알림 | Discord, 이메일 등을 통한 장애 알림 |

---

### 3. Zabbix 주요 구성요소

| 구성요소 | 역할 |
|---|---|
| Zabbix Server | 데이터 수집 및 처리, 장애 판단, 알림 |
| Zabbix Agent | 모니터링 대상 서버에서 데이터 수집 |
| Database | 수집 데이터, 설정, 이벤트 저장 |
| Web Frontend | 웹 기반 모니터링 및 관리 화면 |
| Zabbix Proxy | 여러 대상의 데이터를 중간에서 수집·전달하는 선택적 구성요소 |

**기본 동작 흐름**

```text
[WEB / WAS / DB 서버]
        |
   Zabbix Agent
        |
        | 모니터링 데이터
        v
   Zabbix Server
        |
        +----> Database (데이터 저장)
        |
        +----> Trigger (장애 조건 평가)
                    |
                    v
                  Event
                    |
                    v
                  Action
                    |
                    v
               Discord 알림

Web Frontend <----> Database
```

※ 실제 데이터 통신 방향은 Passive/Active 방식에 따라 달라진다.

---

### 4. Zabbix 핵심 용어

| 용어 | 의미 | 예시 |
|---|---|---|
| Host | 모니터링 대상 | WEB01 |
| Item | 수집할 데이터 항목 | CPU 사용률 |
| Key | 수집 항목 식별자 | system.cpu.load (CPU 부하) |
| Trigger | 장애 판단 조건 | CPU 사용률 90% 초과 |
| Event | 장애 발생 등 상태 변화 기록 | CPU 과부하 발생 |
| Action | 이벤트 발생 시 수행 작업 | Discord 알림 |
| Template | 재사용 가능한 모니터링 설정 묶음 | Linux by Zabbix agent |

**핵심 흐름**

```text
Host → Item → Trigger → Event → Action
```

**예시**

1. WEB01에서 CPU 사용률 수집
2. CPU 사용률이 90% 이상으로 일정 시간 유지
3. Trigger 조건 충족
4. 장애 Event 발생
5. Action을 통해 Discord 알림 전송

---

### 5. Passive Check와 Active Check

**Passive Check**
- Zabbix Server가 Agent에 데이터를 요청
- Agent가 요청받은 데이터를 응답
- Agent 기본 수신 포트: TCP 10050

**Active Check**
- Agent가 Server에 연결하여 수집 항목 목록을 받음
- Agent가 데이터를 수집하여 Server로 전송
- Server 기본 수신 포트: TCP 10051

| 구분 | Passive | Active |
|---|---|---|
| 요청 방식 | Server가 Agent에 요청 | Agent가 수집 및 전송 |
| 기본 수신 포트 | Agent TCP 10050 | Server TCP 10051 |
| 특징 | Server 중심 요청 | Agent 중심 수집 |

---

### 6. Zabbix와 Prometheus 비교

| 구분 | Zabbix | Prometheus |
|---|---|---|
| 주요 목적 | 인프라 통합 모니터링 | 메트릭 중심 모니터링 |
| 서버 모니터링 | Agent 활용 | Exporter 활용 |
| 네트워크 장비 | SNMP 지원 | SNMP Exporter 활용 |
| Kubernetes | 지원 | 널리 활용 |
| 알림 | 자체 Action 기능 | Alertmanager 등 연계 |

**프로젝트 적용**
- Zabbix: WEB / WAS / DB 서버 상태 및 장애 모니터링
- Prometheus: 향후 Kubernetes 메트릭 모니터링 학습에 활용 가능

---

### 7. Linux 기본 모니터링 명령어

```bash
# CPU 및 프로세스 상태 확인
top

# 서버 가동 시간과 Load Average 확인
uptime

# 메모리 사용량 확인
free -h

# 파일시스템 사용량 확인
df -h

# 로그 디렉터리 용량 확인
du -sh /var/log

# Apache 서비스 상태 확인
systemctl status apache2

# HTTP 응답 헤더 확인
curl -I http://localhost

# TCP LISTEN 포트 확인
sudo ss -lntp

# 5초마다 메모리 상태 갱신
watch -n 5 free -h
```

| 명령어 | 설명 |
|---|---|
| top | CPU, 메모리, 프로세스 상태 확인 |
| uptime | 서버 가동 시간 및 부하 확인 |
| free -h | 메모리 사용량 확인 |
| df -h | 파일시스템 사용량 확인 |
| du -sh | 디렉터리 전체 용량 확인 |
| systemctl status | 서비스 실행 상태 확인 |
| curl -I | HTTP 응답 헤더 확인 |
| ss -lntp | TCP LISTEN 포트 및 프로세스 확인 |
| watch -n 5 | 명령어를 5초마다 반복 실행 |

---

### 8. 장애 대응 시나리오

**상황: Marble 쇼핑몰 접속 지연**

Zabbix에서 다음 지표가 확인되었다고 가정한다.

- WEB CPU 사용률: 94%
- HTTP 응답 시간: 8.2초

**장애 대응 순서**

1. Zabbix에서 장애 발생 시간과 지표 확인
2. `top`으로 CPU 사용 프로세스 확인
3. Apache 로그에서 요청 및 오류 확인
4. WAS 응답 시간과 DB 상태 점검
5. 근본 원인 분석 및 조치
6. Zabbix에서 정상 상태 복구 확인

**주의:** CPU 사용률이 높다는 사실만으로 CPU가 장애의 근본 원인이라고 단정해서는 안 된다.

---

### 9. 면접 예상 질문

**Q1. Zabbix란 무엇이며 왜 사용하나요?**

Zabbix는 서버, 네트워크, 애플리케이션 등의 상태를 중앙에서 모니터링하는 오픈소스 솔루션입니다.

CPU, 메모리, 디스크 등 자원 사용량과 서비스 상태를 수집하고, 설정한 임계치를 초과하면 장애 알림을 발생시킬 수 있습니다.

이를 통해 장애를 조기에 발견하고 과거 데이터를 활용하여 장애 원인을 분석할 수 있습니다.

**Q2. Zabbix Agent와 Server의 차이는 무엇인가요?**

Agent는 모니터링 대상 서버에서 CPU, 메모리, 디스크 등의 데이터를 수집하는 역할을 합니다.

Server는 이러한 데이터를 수집·처리하고 Trigger 조건을 평가하여 장애를 감지하고 알림을 처리하는 역할을 합니다.

**Q3. Passive와 Active 방식의 차이는 무엇인가요?**

Passive 방식은 Zabbix Server가 Agent에 데이터를 요청하고 Agent가 응답하는 방식입니다.

Active 방식은 Agent가 Server로부터 수집할 항목 목록을 받아 데이터를 수집하고 Server로 전송하는 방식입니다.

---

### 10. 핵심 요약

1. **Monitoring:** 서버 및 서비스 상태를 지속적으로 관찰하는 활동
2. **Zabbix:** 데이터 수집, 저장, 시각화, 장애 탐지 및 알림 솔루션
3. **Server / Agent:** Server는 중앙 처리, Agent는 대상 시스템 데이터 수집
4. **Host → Item → Trigger → Event → Action:** 장애 탐지 및 알림 흐름
5. **Passive / Active:** Server 요청 방식과 Agent 전송 방식

---

**학습 진도:** 1장 1교시 — 모니터링 개념과 Zabbix 필요성
