

# 📊 Zabbix 모니터링 시스템 구축 가이드

---
  Zabbix란? 서버의 생존 여부, CPU 사용량, 디스크 용량, 서비스 상태 등을 24시간 감시하고 장애를 알리는 오픈소스 모니터링 시스템입니다.
---

## 🔍 주요 모니터링 대상 및 명령어
  1. CPU: top (프로세스 및 리소스 사용량)
  2. Memory: free -h (실제 메모리 사용량 확인)
  3. Disk: df -h (디스크 남은 용량 확인)
  4. Network: eth0 등의 인터페이스 트래픽
  5. Process: 주요 서비스 및 프로세스 실행 여부

## 🛠️ Zabbix 설치 및 구성 (Ubuntu 24.04 / Zabbix 7.0 LTS)
  
  #### 1. 패키지 업데이트 및 MariaDB 설치
       
         # 시스템 패키지 업데이트
         sudo apt update && sudo apt upgrade -y

         # MariaDB 데이터베이스 설치
         sudo apt install mariadb-server -y
       
  #### 2. Zabbix 공식 저장소 등록 및 설치
         
          # Zabbix 7.0 리포지토리 다운로드 및 설치
          wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb
          sudo dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb

          # 저장소 반영을 위한 업데이트
          sudo apt update
    
          # Zabbix 서버, 프론트엔드, 에이전트 패키지 설치
          apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent
       
  #### 3. 초기 데이터베이스 설정
         
           sudo mysql -uroot -p password

           #Zabbix 데이터베이스 생성
           CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;

           #Zabbix 유저 생성 및 권한 부여 ('password' 부분을 원하는 비밀번호로 변경)
           CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'password';
           GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';

           #함수 생성 권한 신뢰 설정 활성화 (데이터 임포트 시 필요)
           SET GLOBAL log_bin_trust_function_creators = 1;
           QUIT; 
        
  
  #### 4. 초기 스키마와 데이터를 임포트합니다.
         
           # 초기 데이터 임포트 (위에서 설정한 'password' 입력)
           zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix

           # 임포트 완료 후 신뢰 설정 다시 끄기 (보안 권장사항)
           sudo mysql -uroot -p -e "SET GLOBAL log_bin_trust_function_creators = 0;"
           
  
  #### 5. /etc/zabbix/zabbix_server.conf 파일을 편집한다.
         
            DBPassword=password
         
  #### 6. Zabbix 서버 및 에이전트 프로세스를 시작한다.
         
            systemctl restart zabbix-server zabbix-agent apache2
            systemctl enable zabbix-server zabbix-agent apache2

---

## 🛠️ zabbix agent 서버 설치 및 구성

  #### 1. 루트 사용자 권한 획득

        sudo -s

  #### 2. Zabbix 저장소 설치

        wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb
        dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb
        apt update
    
  #### 3. Zabbix 에이전트 설치

        apt install zabbix-agent

  #### 4. /etc/zabbix/zabbix_agentd.conf 

        Server= zabbix 서버 IP
        ServerActive= zabbix 서버 IP
        Hostname= agent 서버 이름
  
  #### 5. Zabbix 에이전트 프로세스 시작

        systemctl restart zabbix-agent
        systemctl enable zabbix-agent


## 🛠️ zabbix 페이지 꾸미기 

  #### zabbix-agent 서버 연동하기
      # 데이터 수집 -> 호스트 -> 호스트 생성
      # 호스트 명 : zabbix-agent server.conf에 있는 호스트명
      # 템플릿 : Linux by Zabbix agent
      # 호스트 그룹 : Linux servers
      # Interfaces : 에이전트 -> 서버 IP 
      # 갱신 -> 상태에 초록색

  #### zabbix 아이템 및 트리거 커스텀 (기본 템플릿을 복사하여 사용)
      # CPU 90, 95, 100 % 를 심각도 경고, 중증 장애, 심각한 장애로 설정한다. 
        트리거 생성 -> 이름(ex: CPU 사용률 95% 이상) -> 조건식 : min(/make_by_Linux by Zabbix agent/system.cpu.util,5m)>95 -> 의존 관계(기존 CPU 90%이상 추가)

      # Memory 90, 95% 를 경고, 심각한 장애로 설정
        트리거 생성 -> 이름(ex: Memory 사용률 95% 이상) -> 조건식 : min(/make_by_Linux by Zabbix agent/vm.memory.util,5m)>95 -> 의존 관계(기존 Memory 90%이상 추가)

      # Disk 90, 95, 100% 경고, 가벼운장애, 심각한 장애로 설정
        트리거 프로토타입 -> 조건식 : min(/make_by_Linux by Zabbix agent/vfs.fs.dependent.size[{#FSNAME},pused],5m)>90 
      # zabbix agent 연결 유무
      # 서버 재부팅 유무
      # apache, tomcat, mysql 프로세스 감시 아이템 및 트리거 생성      
      # URL로 웹 접속해서 URL 접속 확인
     
  ### 대시보드

![대시보드 이미지](https://github.com/user-attachments/assets/ae20ac27-7646-49fa-820c-cbef3b701e1f)
        

## 트러블 슈팅
 
    #### 1. 한글설정

      # zabbix 다 설치하고 웹에서 접속하고 설정하려면 한글이 안된다. 
      # 이유는 서버 locale 기준으로 가져오기 때문에 한글을 설치해야한다.

         sudo apt-get install -y language-pack-ko
         sudo locale-gen ko_KR.UTF-8
         locale -a 
    
    #### 2. agent 서버와 Master 서버 연동 실패         
          
  ## 🔔 3. Discord 알림(Alert) 연동
장애 발생 시 대시보드를 보지 않고도 실시간으로 알림을 받을 수 있도록 Discord 웹훅(Webhook)을 연동했습니다.

#### 1) Discord 채널 웹훅 생성
1. 알림을 받을 디스코드 채널 설정 -> `연동` -> `웹후크 만들기` 클릭
2. 생성된 웹후크 URL 복사

#### 2) Zabbix Web 콘솔 설정
1. **미디어 유형 설정:** `경고(Alerts)` -> `미디어 유형(Media types)` -> 기본 제공된 **Discord** 선택
2. **파라미터 변경:** `URL` 항목에 방금 복사한 Discord 웹훅 URL을 붙여넣고 업데이트
3. **사용자에게 미디어 추가:** `관리(Users)` -> `사용자(Users)` -> 본인 계정 선택 -> `미디어(Media)` 탭에서 유형을 **Discord**로 지정 후 추가
4. **트리거 액션 활성화:** `경고(Alerts)` -> `작업(Actions)` -> `트리거 작업(Trigger actions)`에서 알림 규칙이 활성화되어 있는지 확인

#### 3) 결과 예시
> 🚨 **[Problem] CPU 사용률 95% 이상**
> * **Host:** shopmall
> * **Severity:** High
> * **Operational data:** 96.2 %

       
