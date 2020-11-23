#Package repo 구축 가이드

## 구성 요소 및 버전
* HyperCloud 패키지(ck-ftp@192.168.1.150:/home/ck-ftp/k8s_pl/install/offline/archive_20.08.03)
* ISO 파일(CentOS 7.7 :http://vault.centos.org/7.7.1908/isos/x86_64/ 또는 http://192.168.2.136/ISOs/CentOS-7-x86_64-DVD-1908.iso)

## 폐쇄망 구축 가이드
1. Install OS
    * CentOS 7.7 설치
	    * 해당 환경에 맞게 OS를 설치합니다. (IP, hostname, software selection 등)		* 

2. Repository 구축
    * HyperCloud 용 yum repository 구축
	    * HyperCloud 설치 시 필요한 패키지들로 yum Reposiroty 구축		*  

## 설치 가이드
0. [Install OS](#step-0-install-os)
1. [Create Local Repository](#step-1-local-repository-%EA%B5%AC%EC%B6%95)
2. [Add Additional Package]()

## Step 0. Install OS
* 목적 : `CentOS 7.7 설치`
* 생성 순서 : 
    * IP 설정 및 hostname 설정
	    * Network & Host name 클릭
		    ![ip-config1](/figure/network-host-select.png)
      * Configure 클릭
        ![ip-config2](/figure/configure.png)
      * IPv4 Setting 클릭 후 해당 환경에 맞게 IP 설정
        ![ip-config3](/figure/ipv4.png)
      * IPv6 Setting 클릭 후 해당 환경에 맞게 IP 설정
        ![ip-config3-(1)](/figure/ipv6.jpg)
      * Network 설정 활성화
        ![ip-config4](/figure/network-enable.png)
      * Hostname 설정 후 Apply
        ![hostname-config1](/figure/hostname.png)
      * Begin Installation 이후 설치 진행
        
* 비고 :
    * CentOS 설치 자체보다 hypercloud 설치할 때 필요한 부분만 언급하였습니다.    

## Step 1. Local Repository 구축
* 목적 : `폐쇄망일 때 yum repository 구축`
* 생성 순서 : 
    * 패키지 가져오기
      * scp -r ck-ftp@192.168.1.150:/home/ck-ftp/k8s_pl/install/offline/archive_20.08.03 .
      * cp -rT ./archive_20.08.03 /tmp/localrepo
    * CentOS Repository 비활성화
      * sudo vi /etc/yum.repos.d/CentOS-Base.repo
      * [base], [updates], [extra] repo config 에 enabled=0 추가
      * ![repo-config1](/figure/centos-configure.png)
    * Yum Repository 구축
      * sudo yum install -y /tmp/localrepo/createrepo/*.rpm
      * sudo createrepo /tmp/localrepo
      * sudo cat << "EOF" | sudo tee -a /etc/yum.repos.d/localrepo.repo
      * [localrepo]
      * name=localrepo
      * baseurl=file:///tmp/localrepo/
      * enabled=1
      * gpgcheck=0
      * EOF
    * 확인
      * sudo yum clean all && yum repolist
      * 다음과 같이 나오면 완료.
      * ![repo-config3](/figure/fin.png)
* 주의 사항
	* ceph 설치에 device를 사용할 경우의 주의 사항입니다.
	* lvm으로 묶이거나 다른 용도로 사용 중인 device는 지원되지 않으며
	* ceph osd가 deploy되는 노드에 사용할 device가 반드시 존재 및 umount 상태여야 합니다.
	* 따라서 클러스터 구성 전 device의 상태를 확인하고, 자세한 내용은 rook-ceph 설치 단계를 참고하시기 바랍니다.

## Step 2. local repository에 packages 추가 가이드 (Optional)

1. 원하는 packages를 local repo 경로에 추가
    * 추가하고자 하는 packages를 local repo 경로에 추가
	    * mv {packages} {local repo 경로}		 

    * 예시 ( kubernetes-v1.16 )    
    	    
	    * kubernetes-v1.16 다운로드 (ck-ftp@192.168.1.150:/home/ck-ftp/k8s_pl/install/offline/k8s-upgrade/1.16.15)  		
	    * scp -r ck-ftp@192.168.1.150:/home/ck-ftp/k8s_pl/install/offline/k8s-upgrade/1.16.15 . 		
	    * mv 1.16.15/*.rpm /tmp/localrepo
    
    * 예시 ( kubernetes-v1.17 )    
    	    
	    * kubernetes-v1.16 다운로드 (ck-ftp@192.168.1.150:/home/ck-ftp/k8s_pl/install/offline/k8s-upgrade/1.17.6)   		
	    * scp -r ck-ftp@192.168.1.150:/home/ck-ftp/k8s_pl/install/offline/k8s-upgrade/1.17.6 . 		
	    * mv 1.17.6/*.rpm /tmp/localrepo 
	    
2. local Repository 구축
    * yum repository 구축
	    * sudo createrepo {repository 경로}	    
    * 예시
	    * sudo createrepo /tmp/localrepo
	    * yum clean all && yum repolist




## 삭제 가이드
