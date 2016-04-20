2016.04.19 VMware에 Linux - CentOS 설치

# VMware Linux - CentOS 설정
VS Project를 Linux로 Porting 테스트 중.

초기 vmware의 disk 크기를 20GB로 잡는 바람에  디스크 용량 부족으로 model 을 옮길 수 없는 상황이었음.
fdisk 의 여러 옵션을 통해 파티션을 합치는 여러 작업을 시도했지만 vmware 는 불가능한 것으로 결론..
처음부터 다시 vmware에 centOS 부터 시작했다. 그런 김에 과정을 정리해보려고 함.
[fdisk  파티션 통합 방법 ](http://askubuntu.com/questions/66000/how-to-merge-partitions)

## Step1. iso 파일 설치

- root 계정 및 user 계정 설정
- Network 는 Ethernet으로 연결
- 설치 시작

## Step2. development tools 설치
- 필요한 애들은 yum 으로 설치
-- yum install [필요한 애들]
- yum grouplist
- yum groupinstall "Development tools" ( 띄어쓰기 있을때는 "" 안에 )
- yum groupinstall "

## Step3. GNOME DESKTOP
- Terminal 로 작업하기에 어려우니.. 친숙한 GUI를 이용하자
- yum groupinstall "GNOME DESKTOP"
- startx // desktop 버전 실행

## Step4. cmake 설치 & gcc version upgrade
- yum install cmake
- CentOS 7 에서 제공하는 gcc version은 4.8 이다. 
-- 근데 나는 regex를 include 해서 사용하기 때문에 4.9 이상의 gcc가 필요했음.
-- yum 에서는 최대가 4.8 이라 다른 방법을 찾아야 돼
-- 내가 참고한 링크 :  [Fedora-Core23.repo를 만들자](http://serverfault.com/questions/720558/how-to-install-gcc-5-2-on-centos-7-1) // centOS는 없어서 fedora꺼를 대신 쓰는듯?
- cmake 에 대한 내용은 나중에 따로 정리

## Step5. Windows 와 Linux 상의 달랐던 점
- Windows 의 경우 header file 이름의 대소문자를 코드에서 구분 안했는데. Linux에서는 구분해줬어야 됨. build 오류 수정했음
- 파일 경로 역시 \\ 에서 / 로 변경


## Step5. iconv lib 설정
- iconv library를 찾지 못해서 빌드 오류가 남
- [참고링크](http://geeksww.com/tutorials/libraries/libiconv/installation/installing_libiconv_on_ubuntu_linux.php)
-- Link로 파일을 다운받기 위해 wget을 쓴다는 것도 처음 알았음
- 리눅스에 압축 풀고 make 하면 에러가 나는데..
-- [stdio.h:1010 에러 해결](http://linuking.com/GNHome/index.php?document_srl=10960&mid=LinuxEtc&listStyle=viewer&page=7) 여기에 설명이 잘 되어 있다.
- 그래도 에러가 나는데 ldconfig 로  library 경로 설정해줘야됨. [관련링크](http://www.opencode.co.kr/bbs/board.php?bo_table=apache_tips&wr_id=61)

## Step6. 코드 빌드 테스트
- cmake ./
- make
- ./bin/실행파일이름
- 해보면 드디어 결과가 출력 되는 것을 확인할 수 있었다.

## Linux 의 Default Encoding은 UTF-8
- Encoding 문제를 해결하기 위해  iconv 가 필요했던 건데 문제가 많음. 아직 어려워서  나중에 정리해봐야겠다.


## 마지막으로..
- 새로 OS를 깔고 환경설정 할 필요 없이Linux에서 scp 명령어를 통해 할 수 있다고 한다.. 
- 이것도 나중에 정리해보겠음