# Package Repo 구축 및 사용 가이드
폐쇄망에서의 Hyper Cloud 패키지 설치를 위한 로컬 레포 구축 및 패키지매니저(dnf)를 이용한 패키지 설치 가이드입니다.

## 구성 요소 및 버전

## 폐쇄망 구축 가이드
1. 필요한 패키지를 ck-ftp에서 복사
	```bash
	$ scp -r ck-ftp@192.168.1.150:/home/ck-ftp/k8s_package/el8/redhat ./local_repo
	```
	추가적으로 필요한 패키지가 있으면 해당 디렉토리로 다운받거나 복사합니다.

2. local\_repo 디렉토리를 로컬 레포를 구축할 노드로 이동

	tar 압축
	```bash
	$ tar cvzf local_repo.tar.gz local_repo
	```

	tar 압축해제
	```bash
	$ tar xvzf local_repo.tar.gz
	```

3. 패키지들이 있는 디렉토리에 repo를 생성

	압축 해제한 디렉토리 기준
	```bash
	$ pushd ./local_repo
	$ createrepo_c ./
	$ modifyrepo_c modules.yaml ./repodata
	$ export LOCAL_REPO_PATH=$PWD
	$ popd
	```

	만약 해당 노드에 createrepo_c가 없다면 아래 명령어를 이용하여 createrepo_c 패키지를 설치합니다.
	```bash
	$ dnf localinstall ./common/createrepo/*.rpm
	```

4. 노드에 local repo를 추가
	```bash
	$ sudo cat << "EOF" | sudo tee -a /etc/yum.repos.d/localrepo.repo
	 [localrepo]
	 name=localrepo
	 baseurl=file://${LOCAL_REPO_PATH}
	 gpgcheck=0
	 EOF
	```
	또는 아래 명령어
	```bash
	$ dnf config-manager --add-repo file://${LOCAL_REPO_PATH}
	```


	추가된 repo 확인
	```bash
	$ sudo dnf repolist
	```


	필요하다면 기존 repo를 disable
	```bash
	$ dnf config-manager --set-disabled <repo-id>
	```
	`<repo-id>`는 disable할 repo의 id (repolist 기준)

## 설치 가이드
아래 명령어를 통해서 패키지를 설치할 수 있습니다.
```bash
$ sudo dnf install <pkg_name>
```
`<pkg_name>`은 설치할 패키지의 이름
만약 패키지의 버전이 여러개일 때, 특정 버전의 설치를 원하는 경우 패키지의 Name-Version-Release(NVR)을 모두 입력합니다.

ex.
```bash
$ sudo dnf install kublet-1.19.4-0
```

## 삭제 가이드
아래 명령어를 통해서 패키지를 삭제할 수 있습니다.
```bash
$ sudo dnf remove <pkg_name>
```
`<pkg_name>`은 삭제할 패키지의 이름
