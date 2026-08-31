### firewall
```
# 상태확인 
sudo firewall-cmd --state

# 시작 중지 재시작
sudo systemctl start firewalld      # 시작
sudo systemctl stop firewalld       # 중지
sudo systemctl restart firewalld    # 재시작
sudo systemctl enable firewalld     # 부팅 시 자동 시작

현재 열린 방화벽 리스트
sudo firewall-cmd --list-all

# 포트 열기
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent

# 포트 닫기 
sudo firewall-cmd --zone=public --remove-port=8080/tcp --permanent

# 변경사항 반영
sudo firewall-cmd --reload
```


### 프로세스와 함께 확인
```
sudo ss -tulpn

sudo netstat -tulpn | grep 프로세스명
sudo netstat -tulpn | grep pid

## output
tcp        0      0 0.0.0.0:80        0.0.0.0:*        LISTEN      1234/nginx
tcp6       0      0 :::443             :::*             LISTEN      1234/nginx


사용 프로토콜 : tcp, tcp6

Recv-Q : 0

Send-Q : 0

Local Address : 0.0.0.0:80
서버가 바인딩한 IP:PORT

Foreign Address : 0.0.0.0.*

State : 현재소켓상태
PID/Program name : PID/프로세스명

PID/Program name : 

```

### 7777포트 또는 8888포트 연결된 ip 확인

```
# 7777포트 또는 8888포트 연결된 ip 확인
sudo ss -tnp | grep -E ':7777|:8888'

```

### 프로세스 및 연결된 ip 포트 확인 

```
# 프로세스 및 연결된 ip 포트 확인
sudo ss -tnp
```