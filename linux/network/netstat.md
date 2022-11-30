# netstat

`network statistics`


```bash
# centos
yum install netstat
```

- 네트워크 상태 및 정보 확인
- 서비스 포트 상태 확인
    - 참고: [https://itwiki.kr/w/리눅스_netstat](https://itwiki.kr/w/%EB%A6%AC%EB%88%85%EC%8A%A4_netstat)
    - LISTENING: 특정 서비스가 정상적으로 열려 있는지, 서버로 들어오는 패킷을 위해 소켓을 열고 기다리는 상태
    - ESTABLISHED: 외부 서비스와 TCP 세션이 정상적으로 맺어져 있는지, 3way-handshaking이 완료 후 서버와 클라이언트가 서로 연결된 상태
    - TIME_WAIT: 서비스가 정상적으로 종료되고 있는지
    - CLOSE_WAIT: 원격 호스트는 종료된 상태이고 소켓을 종료하기 위해 기다리는 상태
    - SYN_RECEIVED: 서버 시스템이 원격 클라이언트로부터 접속 요구를 받아 클라이언트에게 응답을 하였지만 아직 클라이언트에게 확인 메시지는 받지 않는 상태
- 라우팅 테이블이나 인터페이스 패킷 통계정보 확인

# 사용법

- 리눅스 계열과 윈도 계에서의 netstat 명령어의 기본적인 사용법은 같다.
    
    ```bash
    netstat [옵션]
    ```
    

- man으로 사용법 보는거에 익숙해지기
    
    ```bash
    man netstat 
    q
    ```
    

## 리눅스 명령어 옵션

| 옵션 | 의미 |
| --- | --- |
| -a, --all | 모든 연결과 수신 대기 포트 표시, 연결(Established)되어 있거나 대기(Listening)중인 모든 포트 번호 |
| -l --listening | listening 상태인 것 출력 |
| -n, --numeric | 주소와 포트 번호를 숫자 형식으로 표시(예: http → 80) |
| -r, --route | 라우팅 테이블 표시 |
| -i, --interfaces | 인터페이스별 입출력 패킷 통계 |
| -s, --statistics | 네트워크 통계 데이터 출력 |
| -p, ----programs | PID와 프로그램 이름 출력 |
| -t, --tcp | TCP만 출력(TCP, TCPv6) |
| -4 / -6 | IPv4나 IPv6에 대해 출력 |

## 윈도 명령어 옵션

| 옵션 | 의미 |
| --- | --- |
| -a | 모든 연결과 수신 대기 모드 표시 |
| -n | 주소와 포트 번호를 숫자 형식으로 표시 |
| -r | 라우팅 테이블 표시 |
| -e | 이더넷 통계를 표시한다. -s 옵션과 함께 사용 가능 |
| -s | 프로토콜별 통계를 표시한다. 기본적으로 IP, IPv6, ICMP, ICMPv6, TCP, TCPv6, UDP 및 UDPv6에 대한 통계를 표시 |
| -p proto | proto로 지정한 프로토콜의 연결을 표시 proto는 TCP, UDP, TCPv6, UDPv6 중 하나로, -s 옵션과 함께 사용해 프로토콜별 통계를 표시할 경우, proto는 IP, IPv6, ICMP. ICMPv6, TCP, TCPv6, UDP, UDPv6 중 하나를 사용 |

# 사용

## 리눅스 서버의 사용

- ex) -ant 옵션으로 TCP에 대한 모든 연결과 수신 대기 정보를 숫자로 표기해 출력
    
    ```bash
    netstat -ant4
    ```
    

## 윈도 서버의 사용

- ex) TCP 프로토콜(-p TCP)에 대한 통계 값(-s)을 확인
    
    ```powershell
    netstat -s -p TCP
    ```
    

- 윈도에서 결괏값 필터링
    
    ```powershell
    # find는 리눅스의 grep과 동일
    netstat -an | find "LISTENING"
    netstat -na | findstr 211
    ```
    

## 🍪 부록(유용한 사용)

```bash
netstat -anp | grep CONNECTED | wc -l
netstat -anp | grep ESTABLISHED | wc -l
watch -n 1 'netstat -nap | grep :80 | grep ESTABLISHED | wc -l'

# root가 아닌 사용자가 p 옵션을 사용할 경우 -로 표시된다.
[clouder@mgt-bastion ~]$ netstat -ltp
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 localhost:smtp          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:cgms            0.0.0.0:*               LISTEN      -

# sudo su - 로 전환 후에 명령어를 다시 사용하는 것 보다 sudo !! 를 이용해 처리
[clouder@mgt-bastion ~]$ sudo !!
sudo netstat -ltp
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 localhost:smtp          0.0.0.0:*               LISTEN      1624/master         
tcp        0      0 0.0.0.0:cgms            0.0.0.0:*               LISTEN      768/php
```

# netstat로 대체할 수 있는 명령어

- ss, ip route, ip -s link, ip madder와 같은 명령어로  netstat의 다양한 옵션을 대체할 수 있다.
- 위의 명령어들이 netsat보다 상세하고 보기 쉽게 정보를 정리해주므로 대체 명령어를 쓸 것을 권고하고 있지만 여러 가지 사용상 이점 때문에 netstat는 여전히 많이 사용 된다.
