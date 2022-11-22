# Permission

---

# 1. 권한이란

- 권한 설정 방법(명령어, 설정모드), 권한 관련 파일 및 디렉토리 명령어
- 리눅스의 모든 파일과 디렉토리는 권한(퍼미션)을 가지고 있다. (파일 및 디렉토리에 접근을 제어)
- 리눅스 파일시스템 상세 권한에 대한 정보를 저장하는 부분이 있다.
- 퍼미션들은 시스템 상에 존재하는 파일들에 대한 읽기, 쓰기, 실행에 대한 접근 여부를 결정
- ls -l 명령어로 확인 가능 (첫번째 필드, -rwxr-wr—)
    
    ```bash
    [ec2-user@jenkins-server ~]$ ls -al
    drwx------ 6 ec2-user ec2-user     193 Nov 15 10:18 .
    drwxr-xr-x 5 root     root          49 Nov 15 10:13 ..
    -rw------- 1 ec2-user ec2-user    2638 Nov 15 10:28 .bash_history
    -rw-r--r-- 1 ec2-user ec2-user      18 Jul 15  2020 .bash_logout
    -rw-r--r-- 1 ec2-user ec2-user     193 Jul 15  2020 .bash_profile
    -rw-r--r-- 1 ec2-user ec2-user     231 Jul 15  2020 .bashrc
    drwxrwxr-x 2 ec2-user ec2-user       6 Oct 28 05:09 jenkins_home
    drwx------ 2 ec2-user ec2-user      29 Oct 28 04:26 .ssh
    drwxr-xr-x 2 root     root           6 Nov 15 10:07 test
    -rw------- 1 ec2-user ec2-user    1021 Nov 15 10:18 .viminfo
    ```
    

- 이러한 퍼미션은 다중 사용자 환경을 제공하는 리눅스 환경에서는 가장 기초적인 접근 통제 방법
- 어떠한 사용자가 File & Directory에 관련된 작업(Read & Write & Excute)을 할 수 있거나 없도록 하는 것이 유닉스계열에서 말하는 권한

## 1.1 type

- -는 file d는 directory
- 심볼릭링크 s, 장치파일 c/b, 파이프 p

```bash
**d**rwxrwxr-x 2 ec2-user ec2-user       6 Oct 28 05:09 jenkins_home
```

## 1.2 access mode

- r read, w write, x execute

# 2. 권한 설정

```bash
d rwx rw- r-x 2 ec2-user ec2-user       6 Oct 28 05:09 jenkins_home

# d: type
# rwx: 소유자 허가권 user
# rw-: 그룹 허가권 group
# r-x: 다른 사용자 허가권 other
```

## 2.1 class modes

| reference | class | description |
| --- | --- | --- |
| u | owner | file’s owner |
| g | group | user who are members of the file's group |
| o | other |  |
| a | all | all three of the above, same as ugo, 운영체제의 등록되어 있는 모든 사용자들 |

## 2.2 octal modes

| —- | —x | -w- | -wx | r— | r-x | rw- | rwx |
| --- | --- | --- | --- | --- | --- | --- | --- |
| none | execute only | write only | write and execute | read only | read and execute | read and write | read, write and execute |
| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |

## $ chmod

- change mode
- chmod [권한] [파일 또는 디렉토리의 이름]
- [권한]을 입력할 때는 심볼릭 모드와 옥탯(8진수) 모드 2가지 방식을 이용하여 입력 가능하다.
- 사용 예시
    
    ```bash
    touch perm.txt
    
    # other의 read권한을 빼기
    chmod o-r perm.txt
    
    # read모드 추가
    chmod o+r perm.txt
    
    # 소유자의 read 권한을 뺀다
    chmod u-r perm.txt
    
    # 모든 사용자둘이 rwx를 갖는다
    chmod a=rwx perm.txt
    
    chmod go+rwx perm.txt
    
    # execute only
    chmod 111 perm.txt
    
    # read, write and execute
    chmod 777 perm.txt
    ```
    

### umask

- 파일이나 디렉토리를 생성할 떄 기본적으로 권한 값은 파일의 경우 644(rw-r—r—) 디렉토리의 경우 755(rwxr-xr-x)로 설정된다.
- 이 값은 umask 값에 의해서 결정되는 값으로 파일은 666에서 umask값을 not 연산을 수행 후 둘을  and 연산, 디렉토리는 777에서 같은 방식으로 구해진다.
- umask 값은 umask 명령어를 이용해서 변경 가능하며 재부팅하면 초기화 된다.
    - umask : 파일 또는 디렉토리를 생성할 때 권한 값을 결정하는
    - 재부팅 후에도 적용 시키려면 사용자 초기화 파일에 설정해야 한다.
    
    ```bash
    [ec2-user@jenkins-server ~]$ umask
    0002
    
    [ec2-user@jenkins-server ~]$ mkdir perm
    [ec2-user@jenkins-server ~]$ touch perm.txt
    [ec2-user@jenkins-server ~]$ 
    [ec2-user@jenkins-server ~]$ ls -l |grep perm
    drwxrwxr-x 2 ec2-user ec2-user       6 Nov 17 01:38 perm
    -rw-rw-r-- 1 ec2-user ec2-user       0 Nov 17 01:38 perm.txt
    
    [root@mgt-bastion ~]# umask -S
    u=rwx,g=rx,o=rx
    ```
    

### 권한 관련 명령어

|  | 파일 | 디렉토리 |
| --- | --- | --- |
| r | more, cat, head, tail | ls |
| w | 편집기로 내용 수정 | touch, mkdir, mv, cp, rm |
| x | 파일 이름 | cd |

```bash
# perm이라는 디렉토리안에 perm.txt라는 파일이 생성됐다
mkdir perm; cd perm; echo 'hi' > perm.txt
# r의 권한이 없다면, ls -al로 조회할수없다
chmod o-r perm.txt

# w의 권한이 없다면 디렉토리에 
# 파일을 만들 수도 이름을 바꾸는 것도 파일을 수정하는 것도 할 수 없다
# x는 디렉토리에 cd명령을 통해 들어갈 수 있느냐 없느냐와 관련

# perm안에 모든 디렉토리에 해당하여 other의 w권한을 추가
chmod -R o+w perm
```

## $ chown

```bash
# 소유자를 변경
chown [USER] [FILE]

# 그룹을 변경
chown [:GROUP] [FILE]

# 소유자와 그룹 모두 변경
chown [USER]:[GROUP] [FILE]
chown [USER].[GROUP] [FILE]

# 하위 폴더 소유자 모두 변경 Recursive
chown -R [USER]:[GROUP] [FILE]
```

# 3. 특수 권한

## 3.1 SetUID, SetGID

- SetUID: 나머지 사용자가 파일을 실행할 때 소유자의 권한으로 접근할 수 있게 해주는 권한
- SetGID:  나머지 사용자가 파일을 실행할 때 관리 그룹의 권한으로 접근할 수 있게 해주는 권한

- SetUID
    - ls -l 명령어를 이용해서 확인했을 때 특수권한이 부여된 파일은 소유자의 권한 부분 중 실행 권한 부분이 x가 아닌 s로 나타난다.
    - 나머지 사용자가 파일을 실행하려고 한다면, 소유자(root)권한으로 실행된다.
        
        
        | 소유자(사용자) | 그룹 | 소유자(u) | 관리그룹(g) | 나머지(o) |
        | --- | --- | --- | --- | --- |
        | root | admin | r w s | r w x | r w x |
    - chmod 4xxx [파일이름] 또는 chmod u+s [파일이름]  명령어로 설정할 수 있다.

- SetGID
    - ls -l 명령어를 이용해서 확인했을 때 특수권한이 부여된 파일은 관리 그룹의 권한 부분 중 실행 권한 부분이 x가 아닌 s로 나타난다.
    - 나머지 사용자가 파일일 실행하려고 한다면, 관리그룹(admin)권한으로 실행된다.
        
        
        | 소유자(사용자) | 그룹 | 소유자(u) | 관리그룹(g) | 나머지(o) |
        | --- | --- | --- | --- | --- |
        | root | admin | r w x | r w s | r w x |
    - chmod 2xxx [ 파일이름] 또는 chmod g+s [파일이름] 명령어로 설정할 수 있다.

- 특수 권한 파일 왜 만들까
    
    ```bash
    # SetUID 권한을 가진것을 탐색
    [ec2-user@jenkins-server ~]$ find / -perm -4000
    ...
    /usr/bin/chage
    /usr/bin/gpasswd
    /usr/bin/newgrp
    /usr/bin/mount
    /usr/bin/su
    /usr/bin/umount
    /usr/bin/crontab
    /usr/bin/sudo
    /usr/bin/passwd
    /usr/bin/staprun
    /usr/bin/at
    /usr/sbin/mount.nfs
    ...
    
    # 비밀번호가 저장되는 파일, 아무권한도 없다
    [ec2-user@jenkins-server ~]$ ls -l /etc/shadow
    ---------- 1 root root 856 Nov 15 10:13 /etc/shadow
    
    # 사용자는 비밀번호를 변경하는 명령어를 실행하는 동안은 SetUID로 root권한을 부여받는다
    [ec2-user@jenkins-server ~]$ ls -l /usr/bin/passwd
    -rwsr-xr-x 1 root root 27776 Feb 13  2020 /usr/bin/passwd
    ```
    

## 3.2 StickyBit

- 디렉토리에 부여하는 권한
- 디렉토리를 마치 자유게시판처럼 사용할 수 있게 해주는 권한
- 일반적으로 /tmp 디렉토리에 부여
- ls -l 명령어를 이용해서 확인했을 때 특수권한이 부여된 파일은 나머지 사용자의 권한 부분 중 실행권한 부분이 x가 아닌 t로 나타난다.
    
    ```bash
    [ec2-user@jenkins-server ~]$ ls -ld /tmp
    drwxrwxrwt 8 root root 172 Nov 17 01:56 /tmp
    ```
    
- chmod 1xxx [파일이름] 또는 chmod o+t [파일이름] 명령어로 설정할 수 있다.
- StickyBit가 부여된 디렉토리 내에서는 누구나 자신의 파일을 생성하거나 수정, 삭제가 가능하다.
- 하지만 다른 사용자의 파일을 수정하거나 삭제할 수 없다(관리자는 가능)
