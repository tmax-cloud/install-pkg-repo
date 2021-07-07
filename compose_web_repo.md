# 로컬 웹서버를 이용한 로컬 레포지토리 구축 가이드
폐쇄망과 같은 환경에서 레포지토리를 웹서버로 구축하여 사용하기 위한 가이드입니다.

##폐쇄망 구축 가이드
### 0. 로컬 레포 구축
httpd 패키지 설치를 위해서 BaseOS와 AppStream 레포에 대해서 로컬 레포를 구축합니다.
[iso를 이용한 기본 로컬 레포 구축하기](compose_local_iso.md)


### 1. httpd 패키지 설치
```bash
$ sudo dnf install httpd
```


### 2. firewall 설정 추가
```bash
$ sudo firewall-cmd --permanent --add-service=http
$ sudo firewall-cmd --permanent --add-service=https
$ sudo firewall-cmd --reload
```


### 3. httpd 서비스 시작
```bash
$ sudo systemctl enable httpd
$ sudo systemctl start httpd
```


### 4. 로컬 레포 복사 및 레포 정보 수정
로컬 레포 디렉토리를 apache 기본 디렉토리로 이동합니다.
```bash
$ mkdir -p /var/www/html/prolinux/8/os/x86_64/{BaseOS,AppStream}
$ cp -rT BaseOS /var/www/html/prolinux/8/os/x86_64/BaseOS
$ cp -rT AppStream /var/www/html/prolinux/8/os/x86_64/AppStream
```


repo 정보를 수정합니다. 기존 relelase 레포의 url을 호스트의 주소나 도메인으로 변경합니다.
```bash
$ sed -i 's/prolinux-repo.tmaxos.com/<host-ip or host domain>/' /etc/yum.repos.d/ProLinux.repo
```


### 5. 레포 정보 수정
#### 5.1 기존 로컬 레포 비활성화
아래 명령어를 이용하여 이전에 설정했던 로컬 레포를 비활성화합니다.
```bash
$ dnf config-manager --set-disabled <repo-id>
```


#### 5.2 기본 레포 활성화
아래 명령어를 이용하여 수정한 기본 레포를 활성화합니다.
```bash
$ dnf config-manager --set-enabled BaseOS
$ dnf config-manager --set-enabled AppStream
```
