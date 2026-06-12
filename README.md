# Zabbix

Zabbix란? "서버가 살아있는지, CPU가 얼마나 사용 중인지, 디스크가 꽉 찼는지, 서비스가 죽었는지 24시간 감시하는 시스템"\

어떤 것을 모니터링 할까 ?
  1. CPU : top
  2. Memory : free -h
  3. Disk : df -h
  4. Network : eth0 트래픽
  5. Process 실행 여부

Zabbix 설치
  1. Zabbix 서버 준비
     ```bash
         sudo apt update
     ```
  2. MariaDB 설치
     ```bash
         sudo apt install mariadb-server -y
     ```
  3. Zabbix 공식 사이트 참고 해서 Zabbix 저장소 설치
     ```bash
         wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb 
         dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb
         apt update
     ```
  4. Zabbix 서버, 프런트엔드, 에이전트를 설치한다.
      ```bash
           apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent
      ```
  5. 초기 데이터베이스 생성
       ```sql
           mysql -uroot -p password
           mysql> set global log_bin_trust_function_creators = 0;
           mysql> quit;
       ```
  6. /etc/zabbix/zabbix_server.conf 파일을 편집한다.
       1) DBPassword에서 비밀번호 설정
  
  7. Zabbix 서버 및 에이전트 프로세스를 시작한다.
       ```bash
           systemctl restart zabbix-server zabbix-agent apache2
           systemctl enable zabbix-server zabbix-agent apache2
       ```





       
