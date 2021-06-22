#ISO를 이용한 기본 로컬 레포지토리 구축 가이드
ProLinux 설치 iso를 이용하여 기본 레포지토리인 BaseOS, AppStream 레포를 로컬 레포지토리로 구축하기 위한 가이드입니다.

##폐쇄망 구축 가이드
### 0. ProLinux 설치


### 1. usb에서 파일 복사
usb가 /dev/sdc라고 가정,


#### 1.1 rufus를 이용하여 부팅 usb로 만든 경우 usb 안의 기본 레포를 로컬로 복사합니다.
```bash
$ mkdir ./mnt
$ mount /dev/sdc ./mnt
$ cp -rT ./mnt/BaseOS ./BaseOS
$ cp -rT ./mnt/AppStream ./AppStream
$ export LOCAL_REPO_ROOT=$PWD
```

#### 1.2 usb에 단순히 iso만 복사한 경우, usb의 iso를 복사한 후 iso 안의 기본 레포를 로컬로 복사합니다.
```bash
$ mkdir ./mnt
$ mount /dev/sdc ./mnt
$ cp ./mnt/ProLinux.iso ./
$ umount ./mnt
$ mount ./ProLinux.iso ./mnt
$ cp -rT ./mnt/BaseOS ./BaseOS
$ cp -rT ./mnt/AppStream ./AppStream
$ export LOCAL_REPO_ROOT=$PWD
```


### 2. repo 설정 변경
#### 2.1 기존 릴리즈 레포 비활성화
```bash
$ dnf config-manager --set-disabled BaseOS
$ dnf config-manager --set-disabled AppStream
```


#### 2.2 로컬 레포 추가
```bash
$ dnf config-manager --add-repo file://${LOCAL_REPO_ROOT}/BaseOS
$ dnf config-manager --add-repo file://${LOCAL_REPO_ROOT}/AppStream
```

