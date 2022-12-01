# User와 Group

---

- 참고
    - [https://www.youtube.com/watchv=3V0y9Il1tsk&list=PL0d8NnikouEXVn9FfoX2XVlGgEArLDiLZ&index=1&t=2s](https://www.youtube.com/watch?v=3V0y9Il1tsk&list=PL0d8NnikouEXVn9FfoX2XVlGgEArLDiLZ&index=1&t=2s)

# 1. Login

```bash
useradd clouder 
echo 'clouder' | passwd --stdin clouder

cp -a /etc/sudoers /etc/sudoers.bak 
echo "clouder  ALL=(ALL)       NOPASSWD: ALL" >> /etc/sudoers
```

## 1.1 루트 사용자의 로그인

- 기본적으로 root 사용자는 로그인 할 수 없도록 설정되어 있다.
- root 사용자의 비밀번호를 설정하여 root로 바로 접속 할 수도 있지만,
- su 명령어를 사용하여 root 사용자로 전환하여 로그인 하거나
- su - 를 사용하여 root 사용자의 환경변수도 로드하면서 로그인하거나
- sudo 명령어로를 통하여 root 권한으로 해당 명령어를 실행할 수도 있다.

## 1.2 루트 계정 관리 방안

- 루트 사용자는 초기 환경 설정 시에만 이용하고 SSH를 통해 root 사용자 로그인이 되지 않도록 설정해야한다.
    
    ```bash
    vi /etc/ssh/sshd_config
    
    # 설정 추가
    PermitRootLogin no
    ```
    

- PAM을 통하여 root 사용자로 로그인할 수 없도록 설정
- root 계정으로 부득이 로그인하더라도 환경변수 TMOUT을 설정하여 자동 로그아웃되도록 설정
    
    ```bash
    [root@mgt-bastion ~]# echo $TMOUT
    7200
    ```
    

## $ sudo

- super user do
- 일반 사용자가 root 권한을 잠시 빌려 명령을 실행
- 권한만 잠시 빌리는 것이기 때문에 근본적으로 명령을 내리는 주체는 현재 사용자
- ~~생성, 수정, 삭제 등 이력이 남는 작업을 했을 때, root 유저가 아닌 해당 유저의 이름이 남게 된다.~~
- sudo 명령어를 사용할 때는 root 사용자가 아닌 현재 로그인한 일반 사용자의 비밀번호를 요구한다.
- **sduo 명령어를 사용할 수 있는 사용자 혹은 그룹은 `/etc/sudoers`에 등록이 되어 있어야 한다.**

### /etc/sudoers

```bash
계정명  호스트명=(실행 계정명)  [NOPASSWD:] 명령어
```

| 항목 | 의미 |
| --- | --- |
| 계정명 | 명령어 실행 권한을 줄 계정명이나 그룹명, 모두에게 줄 경우 ALL |
| 호스트 | 실행할 대상 서버명이나 IP, 모든 서버가 대상이라면 ALL |
| 실행 계정명 | 명령어를 실행할 때 어떤 계정의 권한을 갖는지 설정, 생략시 root로 실행 |
| NOPASSWD | 설정할 경우 명령어를 실행할 때 계정 암호를 물어보지 않음 |
| 명령어 | 실행을 허용하는 명령어의 경로, ALL일 경우 모든 명령어를 허용 |

```bash
# 파일 수정 전 권장사항
# ls -al /etc/sudoers
# -r--r-----. 1 root root 4366 Nov 15 10:24 /etc/sudoers
# cp -a /etc/sudoers /etc/sudoers.bak 
# echo "clouder  ALL=(ALL)       NOPASSWD: ALL" >> /etc/sudoers

# ec2-user만 shtudown을 허용할 경우
ec2-user ALL=(ALL) /sbin/shutdown

# 그룹일 경우 %를 붙어 설정
# wheel 그룹에 속한 계정들은 sudo 사용 가능
%wheel ALL=(ALL) ALL

# wheel 그룹에 속한 계정들은 sudo 명령어 사용시 패스워드 묻지 않음
%wheel ALL=(ALL) NOPASSWD: ALL
```

## $ su

- substitute user 또는 switch user
- 현재 사용자를 로그아웃하지 않은 상태에서 다른 사용자의 계정으로 전환하는 명령어
- exit 명령어로 원래 사용자로 돌아온다.
- 로그인하려는 대상 계정의 비밀번호를 요구한다.
- sudo  명령어로 일시적으로 슈퍼유저 권한을 사용할 수 있지만,
- 계속 관리자 권한이 필요한 경우(매번 명령어를 입력할 때마다 sudo를 붙이기 번거롭기 때문에)
- sudo su, sudo -s로 root 계정을 반영구적으로 빌릴 수 있다.
    
    ```bash
    # root 계정의 패스워드 필요
    [clouder@ansible-server ~]$ su -
    Password:
    
    # 현재로그인한 계정의 패스워드 필요
    [clouder@ansible-server ~]$ sudo su -
    We trust you have received the usual lecture from the local System
    Administrator. It usually boils down to these three things:
        #1) Respect the privacy of others.
        #2) Think before you type.
        #3) With great power comes great responsibility.
    [sudo] password for clouder:
    ```
    

- su  : root 계정의 패스워드 필요
- sudo su : 현재 로그인 된 계정의 패스워드

### su [-] username

- Dash의 유무
    - su - uesrname: 입력한 사용자의 사용자 초기화 파일 적용
    - su username: 현재 사용자의 환경을 유지, 사용자 초기화 파일 적용 X

- su - 는 다른 사용자의 계정으로 완전히 전환하고, 전환한 사용자의 환경설정을 불러온다.
- su 명령어는 현재 계정의 환경변수를 유지한채 대상계정으로 전환한다.
- 반면에 su - 명령어는 다른 사용자의 계정으로 전환하고 환경변수까지 그 계정의 상태로 완전히 전환한다.
    
    ```bash
    [vagrant@ansible-server ~]$ su - iteyes
    Password: # iteyes의 비밀번호를 요구
    Last login: Tue Nov 15 10:37:56 UTC 2022 on pts/0
    [iteyes@ansible-server ~]$ pwd
    /home/iteyes
    
    [iteyes@ansible-server ~]$ su eunsun
    Password:
    [eunsun@ansible-server iteyes]$ pwd
    /home/iteyes
    
    [eunsun@ansible-server ~]$ sudo -s # --shell
    [root@ansible-server eunsun]$ pwd
    /home/eunsun
    ```
    

- 옵션
    
    
    | 옵션 | 설명 |
    | --- | --- |
    | -c, --command | 지정한 사용자로 쉘이 실행할 명령어를 지정 |
    | -, -l, --login | 사용자가 직접 로그인했을 때와 동일하게 환경변수 등이 설정되고 홈 디렉토리로 이동 |
    | -s, --shell | 명시된 쉘을 사용 |

## $ who

- who : 접속정보
- who am i : 현재 터미널에 대한 접속 정보
- whoami : 로그인 명
    
    ```bash
    [eunsun@ansible-server ~]$ who
    vagrant  pts/0        2022-11-15 10:19 (10.0.2.2)
    [eunsun@ansible-server ~]$
    [eunsun@ansible-server ~]$ who am i
    vagrant  pts/0        2022-11-15 10:19 (10.0.2.2)
    [eunsun@ansible-server ~]$
    [eunsun@ansible-server ~]$ whoami
    eunsun
    ```
    

## $ last

- 로그인 내역 확인
- /var/log/wtmp 파일을 참조해서 로그인했던 정보를 출력해주는 명령어
- 자신의 시스템에서 접속한 정보를 확인할 수 있다.(ip확인 가능)
    
    ```bash
    [eunsun@ansible-server ~]$ last
    vagrant  pts/0        10.0.2.2         Tue Nov 15 10:19   still logged in
    vagrant  pts/0        10.0.2.2         Tue Nov 15 07:09 - 09:46  (02:37)
    vagrant  pts/0        10.0.2.2         Tue Nov 15 06:59 - 07:02  (00:03)
    reboot   system boot  3.10.0-1127.el7. Tue Nov 15 06:01 - 10:58  (04:56)
    ```
    

# 2. User

- 리눅스의 사용자는 크게 3가지로 구분할 수 있다.
    1. 루트(root) 계정
        - 루트 계정의 UID는 0이다.
        
    2. 시스템 계정
        - uid가 1 ~ 499를 갖는 계정
        
        | root | 시스템에서 모든 권한을 가지고 있는 최고 권한 사용자
        시스템 내의 보호되는 파일이나 퍼미션 등의 제한 사항에 대해 대부분 영향을 받지 않는다. |
        | --- | --- |
        | bin | 시스템 구동중인 바이너리 파일을 관리하기 위한 계정 |
        | daemon | 백그라운드 프로세스에 대한 작업을 제어하기 위한 시스템 계정 |
        | adm | 시스템 로깅이나 특정 작업을 관리하는 시스템 계정 |
        | lp | 프린트를 위한 계정 |
        | gdm | gnome display 관리 서비스 계정 |
    
    1. 사용자 계정
        - 레드햇 또는 CentOS 6버전 계열의 UID는 500이상의 값을 갖는다.
        - 그러나 레드헷 또는 CentOS 7버전과 오픈수세, 데비안의 UID는 1000이상의 값을 갖는다.
        - 일반 사용자 계정의 UID 범위는 /etc/login.defs에 정의되어 있다.

## 2.1 UID

- 시스템 인스톨 시 기본으로 설정되는 시스템 계정은 보통 0 ~ 99(root 계정 포함)의 범위를 갖는다.
- 시스템 계정은 레드헷 계열은 100 ~ 499를 갖고 데비안 계열인 경우 100 ~ 999 범위를 갖는다.
- 이 범위는 배포판마다 상이할 수 있으므로 확인 필요하다.

## 2.2 시스템 계정 관리 방안

- 서비스별로 권한을 분리하여 시스템 계정을 생성?
- 리눅스를 설치하면 기본으로 설치되는 시스템 계정을 보면 메일 관리는  mail 사용자가 담당하고, ssh 서비스 관리는 ssh 계정이 담당하고 있다.

## 2.3 사용자관련 파일

### /etc/passwd

- 사용자의 기본 정보를 저장하고 있는 파일
    
    ```bash
    [ncloud@mgt-bastion ~]$ cat /etc/passwd
    **root:x:0:0:root:/root:/bin/bash**
    bin:x:1:1:bin:/bin:/sbin/nologin
    ...
    **ncloud:x:11000:11000:ncloud sudo-account:/home1/ncloud:/bin/bash**
    ...
    
    # 사용자, 사용자 홈디렉토리만 보기
    # cut -f1,6 -d: /etc/passwd
    ```
    
    | 로그인 명 | 각 사용자는 유일한 이름을 가지고 있어야함(중복허용X) |
    | --- | --- |
    | 사용x | - (본 사용 목적 : 패스워드 저장 부분)
    - 현재는 사용 x, /etc/shadow 파일에 따로 저장 |
    | UID | - 로그인명은 중복이 x, but UID는 중복이 가능
    - 시스템 사용자를 식별하는 번호
    - root 의 권한을 가질 수도 있음
    - 일반 사용자는 1000번 이상의 사용
    - 일반적으로 1000보다 작은 UID는 예약 UID |
    | GID | 해당 사용자가 속해 있는 주 그룹의 GID |
    | 주석 | - 일반적으로 사용자의 전체 이름을 써준다.
    - 또는 연락처 부서 등 로그인명만으로 사용자의 구분이 힘들 때 자세한 설명을 쓰는 부분 |
    | 사용자 홈 디렉토리 | 사용자의 홈 디렉토리 |
    | 로그인 쉘 | 사용자가 로그일할 때 실행되는 쉘을 지정 |

### /etc/shadow

- 사용자의 패스워드를 저장하고 있는
    
    
    | 로그인명 | 사용자의 로그인 명 |
    | --- | --- |
    | 패스워드 필드 | - 암호화돼서 저장, 대칭키 암호화 알고리즘(DES)을 이용한 Crypt 함수
    - $로 필드로 구분(패스워드 생성시 사용된 salt 값을 저장
    - *LK* : 해당 사용자는 로그인이 할 수 없는 lock이 걸린 사용자를 의미
    - 공백 : 패스워드 입력 없이 로그인 할 수 있음
    - Windows의 암호저장 파일경로: C:Windows/System32/config/SAM |
    | 마지막 변경일 | 1970/1/1 기준으로 패스워드를 변경한 날짜까지 count |
    | MIN | 패스워드 변경 후 최소 사용 기간(패스워드 변경 후 설정 MIN이 지날 때까지 재변경 불가) |
    | MAX | - 패스워드 변경 후 최대 사용 기간
    - 지정된 MAX값 이내에 변경 패스워드 사용 가능 |
    | WARN | 패스워드 만기일 이전에 사용자에게 경고 메시지를 전달할 날짜 지정 |
    | Incative | - 비활성화 사용자
    - 휴면 계정 설정(설정 날짜만큼 로그인 활동이 없으면 Lock 계정(휴면계정)으로 변경됨 |
    | Expire | - 패스워드 만기일 설정
    - 해당 날짜에 자동으로 계정 locking, 로그인 불가(관리자에 의해 lock 해제 필요) |
    | Reserved | 예약필드, 현재는 사용 X |
    
    ```bash
    [root@ansible-server ~]# cat /etc/shadow |column -s ":" -t
    root             $1$m.FEVNiS$OYiaRNHMHzS85/wnDHccI.         0  99999  7
    bin              *                                   18353  0  99999  7
    ...
    sync             *                                   18353  0  99999  7
    shutdown         *                                   18353  0  99999  7
    halt             *                                   18353  0  99999  7
    ...
    games            *                                   18353  0  99999  7
    ftp              *                                   18353  0  99999  7
    nobody           *                                   18353  0  99999  7
    systemd-network  !!                                  18382
    dbus             !!                                  18382
    polkitd          !!                                  18382
    rpc              !!                                  18382  0  99999  7
    ...
    vagrant          $1$gPNBpA.5$5pr.KtXhOx6S/Hc69TUZZ.         0  99999  7
    ```
    

- 패스워드 생성
    - password → H0 → 암호문1 → crypt(Salt 값 추가 + 암호문1) → 암호문2
    - salt 값 : 운영체제에서 random하게 만들어 내는 값
    - 같은 암호를 사용하는 두 사용자의 암호값 혼란을 막기 위해

- 패스워드 정책 및 기본값 설정
    - 패스워드 정책 설정 파일 : **/etc/security/pwquality.conf**
    - 패스워드 기본값 설정 파일 : **/etc/login.defs**
    - 복잡도, 변경주기, 히스토리 등
    - 대/소문자, 숫자, 특수문자를 혼용하여 8글자 이상의 패스워드를 사용
    - 동일 문자를 연속 4회 이상 사용하기 금지
    - 패스워드 히스토리를 관리해 2~3개 이상 동일 패스워드를 사용 금지
    - 패스워드 변경 주기를 설정(패스워드 유효기간을 90일 이하로 설정)
    - 사전에 나오는 쉬운 단어나 이름은 패스워드로 사용하지 못하도록 설정
    - 기본 설정된 패스워드는 사용하지 못하도록 설정
    - 초기 부여된 패스워드는 사용자 최초 접속 시 변경하도록 설정

- authconfig
    
    ```bash
    # 최소길이
    authconfig |grep passminlen
    
    # 대소문자
    authconfig |grep enablereqlower
    authconfig |grep enablerequpper
    
    # 명령어로 설정파일에 옵션 추가
    authconfig --passminlen=10 --update
    tail -15 /etc/security/pwquality.conf
    
    authconfig --enablereqlower --update
    ```
    

### /etc/group

- 그룹에 대한 정보를 저장하고 있는
    
    ```bash
    [root@mgt-acc-001 ~]# cat /etc/group
    # 그룹이름 : 패스워드 : GID : 사용자 목록
    **root:x:0:**
    ...
    **ncloud:x:11000:**
    ```
    

### /etc/default/uaserdd

- 위의 파일을 직접 열어보거나 useradd 명령어의 -D 옵션을 사용하여 파일의 내용 확인
- useradd 명령으로 사용자 생성 시 사용되는 기본 설정 값이 저장된 환경 설정 파일

# 3. 사용자 및 그룹 추가

```bash
# 존재하지 않는 사용자를 추가할 때 그룹 지정
useradd -G [group_name] [user_name]

# 새로운 그룹을 만든다
groupadd [group_name]
sudo groupadd developer

# 그룹정보 확인
# /etc/group

# 그룹에 사용자 추가
usermod -a -G [group_name] [user_name]
sudo usermod -a -G developer egoing

# change directory group in linux
chown eunsun.developer .
chown eunsun.developer .
chmod g+x .

# mogaha:x:1100:1100::/app/mogaha:/bin/bash
groupadd -g 1100 esb
useradd -u 1100 -g 1100 -d /app/mogaha mogaha -m

# 하위 폴더 소유자 모두 변경하기(Recursive)
chown -R mogaha:esb folder
```

## $ useradd

- 사용자 추가
- 사용자를 생성하기 위해서는 useradd 명령어나 adduser 명령어를 사용한다.
- CentOS 계열에서는 adduser는 useradd에 대한 링크이기 때문에 useradd와 adduesr는 차이가 없지만
- Ubuntu에서는 차이가 존재한다.
    - Ubuntu에서 useradd 명령을 사용할 경우 -m옵션으로 홈 디렉터리를 지정하고 passwd를 통한 패스워드 지정이 필요하다. 그러나 adduser를 사용하면 대화식으로 사용자 정보를 입력하여 생성할 수 있다.
- useradd [option] username
    
    
    | -u | UID를 지정(normally set unique) |
    | --- | --- |
    | -g | 주 그룹의 GID를 설정 |
    | -G | 보조 그룹의 GID 설정 |
    | -d | 홈 디렉토리 |
    | -m | 홈 디렉토리 지정 시 홈 디렉토리가 존재하기 않을 경우 해당 디렉토리를 생성하도록하는 옵션 |
    | -s | 로그인 쉘 설정 |
    | -c | 주석(사용자에 대한 설명) |
    | -f | 휴먼 계정 전환 날짜 지정 |
    | -e | 만료일 지정 |

## $ usermod

- 사용자 계정 정보 변경
- usermod [option] username
    
    
    | -u | UID를 지정(normally set unique) |
    | --- | --- |
    | -g | 주 그룹의 GID를 설정 |
    | -G | 보조 그룹의 GID 설정 |
    | -D | 기본 설정값 확인 (uesradd -D)
    uesradd -D -s /etc/sh → 기본 설정값 변경 |
    | -d | 홈 디렉토리 |
    | -s | 로그인 쉘 설정 |
    | -c | 주석(사용자에 대한 설명) |
    | -f | 휴먼 계정 전환 날짜 지정 |
    | -e | 만료일 지정 |

## $ userdel

- 사용자 계정 삭제
- userdel [option] uesrname
    
    
    | -r | 해당 사용자가 사용하던 홈 데릭토리도 같이 삭제 |
    | --- | --- |

## $ groupadd

| -g | 그룹의 GID 지정 |
| --- | --- |
| -o | GID 중복 설정 가능하게 하는 옵션 |

## $ groupmod

| -g | 그룹의 GID 지정 |
| --- | --- |
| -o | GID 중복 설정 가능하게 하는 옵션 |

## $ groupdel

- groupdel

## $ passwd

- passwd [option] username
    
    ```bash
    echo 'eunsun' |passwd --stdin eunsun
    ```
    

- 해당 사용자의 패스워드 변경
- 사용자명을 입력하지 않으면, 현재 사용자의 패스워드로 변경
    
    
    | -n | 패스워드 변경 후 최소 사용 기간 |
    | --- | --- |
    | -x | 패스워드 변경 후 최대 사용 기간 |
    | -w | 패스워드 만기일 이전에 사용자에게 경고 메시지를 전달할 날짜 지정 |
    | -f | 최초 로그인시에 강제로 패스워드를 변경하도록 설정 |
    | -d | shadow  파일의 패스워드 필드 값을 제거
    passwd 입력 없이 로그인 가능 |
    | -l | 해당 사용자의 상태를 locking으로 변경 |
    | -S | 해당 사용자의 현재 패스워드 설정 조회 |

# 사용자 초기화 파일

- 적용되는 범위가 다른 사용자 초기화 파일
- /etc/profile : 시스템 전역에 걸쳐 환경을 설정하는 파일, 모든 사용자가 적용되는 파일
    - umask도 profile에 설정되어 있다. 재부팅하면 초기화 되는 이유
    - 재부팅해도 적용되게 하고 싶으면 파일 자체를 수정

- ~/.profile : 개별 사용자의 홈 디렉토리에 있는 파일, 해당 사용자의 설정을 변경할 때 사용
- ~ /.bashrc : 개별 사용자의 홈 디렉토리에 있는 파일, 해당 사용자의 쉘 관련 설정을 변경할 때 사용
- 환경변수, 쉘 프롬포트 모양(명령어 앞에 붙는 내용), 별명 기능(alias), 쉘 옵션 정의 등 설정 기능

```bash
# 사용자 초기화 파일
.
..
.bash_logout
.bash_profile
.bashrc
.kshrc
.mozilla

# /etc/skel 의 파일 목록과 같다
# /etc/skel 이곳에 사용자 초기화 파일같은 기본값들이 있다.
# 사용자 초기화 파일들은 여기 있는 파일들이 복사되어 만들어진다.
```
