# OS 및 Package Repo 구축 가이드

## 구성 요소 및 버전
* HyperCloud 패키지(ck-ftp@192.168.1.150:/home/ck-ftp/k8s_package/el8/redhat)
* ISO 파일(CentOS 8.2 :https://vault.centos.org/8.2.2004/isos/x86_64/CentOS-8.2.2004-x86_64-dvd1.iso 또는 http://192.168.2.136/centos/8.2.2004/isos/CentOS-8.2.2004-x86_64-dvd1.iso)

## 폐쇄망 구축 가이드
1. Install OS
    * CentOS 8.2 설치
	    * 해당 환경에 맞게 OS를 설치합니다. (IP, hostname, software selection 등)		 

2. Repository 구축
    * HyperCloud 용 dnf repository 구축
	    * HyperCloud 설치 시 필요한 패키지들로 dnf Reposiroty 구축 	  
3. 주의사항
    * BIOS 세팅
        * HyperThreading 기능을 켜서 사용하기
		* ![image (10)](https://user-images.githubusercontent.com/45585638/106423281-79bfec80-64a3-11eb-97da-b4f01aeeba43.png)
    * ceph 설치에 device를 사용할 경우의 주의 사항입니다.
        * lvm으로 묶이거나 다른 용도로 사용 중인 device는 지원되지 않으며
        * ceph osd가 deploy되는 노드에 사용할 device가 반드시 존재 및 umount 상태여야 합니다.
        * 따라서 클러스터 구성 전 device의 상태를 확인하고, 자세한 내용은 rook-ceph 설치 단계를 참고하시기 바랍니다.
## 설치 가이드
0. [Install OS](#step-0-install-os)
1. [Create Local Repository](#step-1-local-repository-%EA%B5%AC%EC%B6%95)
2. [Add Additional Package](#step-2-local-repository%EC%97%90-packages-%EC%B6%94%EA%B0%80-%EA%B0%80%EC%9D%B4%EB%93%9C-optional)

## Step 0. Install OS
* 목적 : `CentOS 8.2 설치`
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
      * scp -r ck-ftp@192.168.1.150:/home/ck-ftp/k8s_package/el8/redhat/common .
      * scp ck-ftp@192.168.1.150:/home/ck-ftp/k8s_package/el8/redhat/k8s/k8s1.19/*.rpm . (k8s version 확인 필요, k8s1.15, k8s1.16, k8s1.17, k8s1.18, k8s1.19)     
      * scp ck-ftp@192.168.1.150:/home/ck-ftp/k8s_package/el8/redhat/kernel/kernel8.2/*.rpm . (kernel version 확인 필요 kernel7.6, kernel7.7, kernel7.8, kernel7.9)
      * cp -rT ./common /tmp/localrepo
      * mv ./*.rpm /tmp/localrepo
    * CentOS Repository 비활성화
      * sudo vi /etc/yum.repos.d/CentOS-BaseOS.repo
      * [BaseOS] repo config 에 enabled=0 추가
      * sudo vi /etc/yum.repos.d/CentOS-AppStream.repo
      * [AppStream] repo config 에 enabled=0 추가
    * Yum Repository 구축
      * sudo dnf install -y /tmp/localrepo/createrepo/*.rpm
      * sudo createrepo_c /tmp/localrepo
      * sudo cat << "EOF" | sudo tee -a /etc/yum.repos.d/localrepo.repo
      * [localrepo]
      * name=localrepo
      * baseurl=file:///tmp/localrepo/
      * enabled=1
      * gpgcheck=0
      * EOF
    * Modularity 적용  
      * scp ck-ftp@192.168.1.150/home/ck-ftp/k8s_package/el8/redhat/modules.yaml .
      * mv modules.yaml /tmp/localrepo/repodata
      * modifyrepo_c --mdtype=modules /tmp/localrepo/repodata/modules.yaml
    * 확인
      * sudo dnf clean all && dnf repolist      

## Step 2. local repository에 packages 추가 가이드 (Optional)

1. 원하는 packages를 local repo 경로에 추가
    * 추가하고자 하는 packages를 local repo 경로에 추가
	    * mv {packages} {local repo 경로}		 

    * 예시 ( kubernetes-v1.16 )    
    	    
	    * kubernetes-v1.16 다운로드 (ck-ftp@192.168.1.150:/home/ck-ftp/k8s_package/redhat/k8s/k8s1.16)  		
	    * scp ck-ftp@192.168.1.150:/home/ck-ftp/k8s_package/redhat/k8s/k8s1.16/*.rpm . 		
	    * mv ./*.rpm /tmp/localrepo
    
    * 예시 ( kubernetes-v1.17 )    
    	    
	    * kubernetes-v1.16 다운로드 (ck-ftp@192.168.1.150:/home/ck-ftp/k8s_package/redhat/k8s/k8s1.17)   		
	    * scp -r ck-ftp@192.168.1.150:/home/ck-ftp/k8s_package/redhat/k8s/k8s1.16/*.rpm . 		
	    * mv ./*.rpm /tmp/localrepo 
	    
2. local Repository 구축
    * dnf repository 구축
	    * sudo createrepo_c {repository 경로}	    
    * 예시
	    * sudo createrepo_c /tmp/localrepo
	    * sudo modifyrepo_c --mdtype=modules /tmp/localrepo/repodata/modules.yaml
	    * dnf clean all && dnf repolist




## 삭제 가이드
