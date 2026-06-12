

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
         
           mysql -uroot -p password

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
        





       
