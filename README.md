# learn_docker
가장 빨리 배우는 도커 실습



1) 오픈자바와 SSH서버를 통합한 Dockerfile을 만드시오 (font 13)

하기는 도커 파일입니다.

Dockerfile

---- start Dockerfile ----

FROM ubuntu:latest
MAINTAINER Jaesik phee <maguire1815@gmail>
RUN apt-get update
RUN apt-get install -y nano

#하기는 openjdk
RUN apt-get install -y openjdk-8-jdk
RUN apt-get clean			

ENV TERM=xterm

ENV JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
ENV CLASSPATH=$JAVA_HOME/lib/*:.
ENV PATH=$PATH:$JAVA_HOME/bin

apt-cache search openjdk
RUN apt-get clean
#deb가 설치되어서 삭제하고 싶은 경우 deb를 한다.

#하기는 sshd
RUN apt-get install -y openssh-server

RUN mkdir /var/run/sshd
RUN echo 'root:kitri' | chpasswd
#id 및 패스워드
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config	
RUN sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config
#root는 원래 ssh가 안되는데 풀어줌

# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
#ssh 버그가 있어서 버그픽스용 문자열이 있어야 함

ENV NOTVISIBLE "in users profile"
RUN echo "expose VISIBLE=now" >> /etc/profile

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
#직접 sshd 서버를 실행을 해줘야함

---- end Dockerfile ----


2) 다음요구조건을 만족하도록 코드를 추가하시오
- wget을 사용해서 https://archive.apache.org/dist/hadoop/common/hadoop-1.2.1/hadoop-1.2.1.tag.gz를 다운
- 압축을 푸시오 (tar xvfz 사용) - HADOOP_HOME설정 - PATH추가 ($HADOOP_HOME/bin, $HADOOP_HOME/sbin) 추가

---- start Dockerfile ----
FROM ubuntu:latest
MAINTAINER jaesik phee <maguire1815@gmail.com>
RUN apt-get update
RUN apt-get install -y nano
RUN apt-get install wget
RUN wget https://archive.apache.org/dist/hadoop/common/hadoop-1.2.1/hadoop1.2.1.tar.gz
RUN tar zxvpf hadoop1.2.1.tar.gz

ENV HADOOP_HOME=/usr/local/hadoop-1.2.1
ENV PATH=$PATH:${HADOOP_HOME}/bin:{$HADOOP_HOME}/sbin
---- end Dockerfile ----


sudo apt-get update
sudo apt-get install openssh-server

putty
NAT주소 공유?
브릿지 (Bridged) : 독립 주소 받기

도커의 특징
시스템의 분리에는 Linux Containers(LXC)
파일 시스템은 Advanced multi layered unification filesystem (Aufs)를 사용해서
de facto standard

하이퍼바이저 기반의 우분투와 토커기반의 우분투의 차이
환경변수 설정 및 반영, 서비스 수행 방식
go라는 언어로 작성
DotCloud

LXC
cgroups (control groups)
CPU, Memory, Disk, network - provisioning

Namespaces(Namespace Isolation)
프로세스트리, 사용자계정, 파일시스템, IPC, ...
호스트와 별개의 공간 설정

chroot(change root)명령어에서 발전
chroot jai
chroot상의 폴더에서 외부 디렉토리 접근 안됨

Libcontainer
별도의 실행드라이버를 만들어 다양한 리눅스를 지원 가능

도커 = LXC (cgroups+namespaces) or libcontainer, AUFS, 이미지

LXD - LXC에 보안 개념까지 추가
KMC 커널 머신 컨테이너


도커 설치
http://docker.com
도커 툴박스, 도커머시
Boot2Docker vs DockerMachine

도커 설치 및 사용 방법
docker search ubuntu
docker pull ubuntu
docker images
docker run --name=ubuntu ubuntu
docker attach ubuntu
docker exec -it ububtu bash
exit or 컨트롤P-Q (컨테이너 정지하지 않고 나옴)

docker ps -a
docker stop ububtu
docker restart ububtu
docker rm ubuntu or docker rm -f ubuntu
docker rmi ubuntu, docker images

docker save -o ubuntu_img.tar ubuntu
gzip unbuntu_img.tar / bzip2 ubuntu_img.tar
gzip -d ubuntu_img.tar.gz / bzip2 -d ubuntu_img.tar.bz2
docker rmi ubuntu
docker load -i ubuntu_img.tar
docker tag 이미지ID ubuntu
일반적인 리눅스와 다른 부분
일반 : ifconfig 
도커 : docker inspect ubuntu | grep "IPAddress"

도커 run 옵션
p(publish) : 포트 노출
d(detach)/--detach : 서버형 실행
e(env)/--env : 환경변수
i(iteractive): 표준 입력 열어두기
등..




도커 다운로드 -> 도커닷컴 또는 도커툴박스

도커툴박스 설치 -> 하이퍼바이저 버추얼박스


이미지 검색 + 다운로드 + 설치
docker search ubuntu
=> docker 서버 repository에서 확인 가능

docker pull ubuntu
=> 이미지 다운 받음
=> default :latest 태그가 지정되어있음

docker images
=> 이미지 확인

docker run -it --name=ubuntu_server ubuntu
=> ubuntu_server의 이름으로 ubuntu를 이미지를 실행
=> cd /etc && ls && cat issue 
==>issue는 현재 우분투 버전을 알 수 있음

ctrl+p+q
=> 돌고 있는 상태로 나옴
quit
=> docker 종료
docker ps
=> 도커가 잘 작동하는지 확인

docker run -it --name=ubuntu_server2 ubuntu
=> 도커 이미지 하나 더 띄움

docker exec -it ubuntu_server bash
=> attach와 비슷하나 attach는 port 변경등에 문제가 있음
=> 본어게인

docker exec -it ubuntu_server2 bash
=> 위와 같음

docker ps
docker ps -a

docker stop ubuntu_server2
docker ps -a
docker rm ubuntu_server2
docker ps -a

컨데이너 삭제 및 상태 확인 등

docker rmi ubuntu
=> 시스템에 있는 다운로드 받은 이미지를 완전하게 삭제
docker ps -a

docker rm -f ubuntu_server
=> stop 없이 바로 삭제

docker images
docker save -o ubuntu_img.tar ubuntu
=> ubuntu_img.tar로 파일로 저장을 하는 방법 (130MB)

docker rmi ubuntu
docker images
docker load -i ubuntu_img.tar
=> 파일에서 이미지를 로딩하라는 명령어
docker images

docker tag ubuntu sample
=> ubuntu이미지를 sample이라는 이미지로 태그 하나 더 복제
=> 동일한 이미지 아이디이기 때문에 실제 디스크는 복사된것이 아니다.

ifconfig
=> 일반적인 리눅스 ip확인

ipconfig
=> 윈도우 ip확인, 버추얼박스로 설치한 경우 해당 ip 확인

docker ps -a

docker run -it --name=ubuntu_server ubuntu
#ifconfig
=> 우분투 내에서 해당 ip를 확인 불가능하다.

docker inspect ubuntu_server
=> Json형태로 값이 나옴
=> NetWork : {brigde : { IPAddress 확인
=> NAT 가 적용되어있음

docker inspect ubuntu_server | grep "IPAddress"
=>위의 값을 간단하게 보는 방법

docker run -it --name=ubuntu_server2 ubuntu
docker inspect ubuntu_server2 | grep "IPAddress"
=> 두번째 서버

docker ps -a

docker images
docker ps -a
docker stop ubuntu_server
docker ps -a
docker restart ubuntu_server
docker ps -a
=> up으로 다시 바뀌어서 동작하고 있는 것을 확인
docker stop ubuntu_server
docker rm ubuntu_server
docker ps -a
docker run -it --name=ubuntu_server ubuntu
ctrl + P + Q
docker ps -a
docker rm -f ubuntu_server
=> unix와 같이 kill이라는 명령어를 사용해도 된다.
docker images
docker rmi ubuntu
docker images




docker

커밋을 통한 이미지 생성
apt-get update & install
도커 기반의 우분투는 필수

apt-get install nano
도커 기반 이미지에서는 깔려 있지 않음

image -> original image
image' = image + update + nano

도커 이미지 생성 방법 
docker diff ubuntu (컨테이너 변경사항 확인)
docker commit -m "test" -a "sjha" ubuntu ubuntu_nano
-m : 메시지를 남김
-a : 사용자 명시 (누가 만들었는지)
docker images

docker ps -a

docker exec -it ubuntu_server bash
// bash 명령어 수행

#nano
#vi
위 둘다 안 깔려있음

sudo 필요 없음
docker의 ubuntu는 기본적으로 root 계정
ubuntu는 데비안 계열이므로 apt 패키지가 있다.

apt-get install nano
-> error 발생 
apt-get update 우선 해야함
docker의 ubuntu는 repository 및 여러 변경사항을 반영하기 때문에 docker ubuntu를 설치해야 함
apt-get install nano
-> 설치됨

ctrl + p + q
docker images
docker run -it --name=sample ubuntu
-> nano가 안 깔려 있음

docker commit -m add_nano -a sjha ubuntu_server ubuntu_nano
A : 기반 이미지명, B : 새로 만들어지는 image 이름

docker run -it --name=ubuntu_nano ubuntu_nano
ubuntu_nano와 ubuntu_server의 차이점 궁금

docker diff ubuntu_server 
오리지날 이미지에서 어떤 차이가 있는지 확인

도커 파일을 기반으로 한 이미지 생성
Dockerfile-> 스크립트 파일로 보면 됨

build?

docker build --tag=ubuntu_nano
docker images
docker history ubuntu_nano (이미지 변경사항 확인)

========================
FROM ubuntu:latest
MAINTAINER jaesik phee <maguire1815@gmail.com>

RUN apt-get update
RUN apt-get install nano
ENV TERM=xterm
========================

docker file 생성하기
도커파일은 기본적으로 생성할 때, ./ 디렉토리 안에 있는 모든 파일을 참조?함
mkdir tmp
cd tmp

========================
FROM ubuntu:latest // base image 
//FROM ubuntu:14.04 // tag
//FROM ubuntu:16.04
MAINTAINER jaesik phee <maguire1815@gmail.com> // 만든 사람의 정보

RUN apt-get update
RUN apt-get install -y nano
// 원래는 설치할 때 yes, no를 물어본다.
// -y 옵션을 두어서 yes no를 물어보지 않고 설치할 수 있게 한다.

Dockerfile 상세

FROM
도커 이미지 생성할 때 기본 이미지를 지정
만약 해당 이미지가 없으면 서버 저장소(repository)에서 다운로드 받는다.
?? 만약 서버에 없는 이미지를 지정하면?

MAINTAINER
이미지를 생성한 사람에 대한 기본 정보 표시

RUN
FROM에서 지정한 기본 이미지 위에 명령 수행해 새로운 이미지 생성
- 우분투 최신 패치 수행
RUN apt-get update
- 우분투 wget 패키지 설치
RUN apt-get install wget

컨테이너가 생성되는 시점에 명령을 실행시키고 싶을 때
CMD
컨테이너가 수행될 때 지정한 명령어/명령/스크립트 파일 실행
Dockerfile에서 한 번만 가능
- CMD ["echo $PATH"]

ENTRYPOINT
CMD와 거의 같으나 컨테이너 생성이나 시작(start)될 때 실행
- ENTRYPOINT ["/sample.sh"]

CMD, ENTRYPOINT는 RUN과 다르게 이미지 자체에는 영향이 없고
이미지가 만들어진 이후에 영향을 주게된다.

환경변수 설정
일반적인 우분투의 경우
홈디렉토리의 ~/.bashrc나 ~/.profile에
-export sample=/sample과 같이 지정한 후
-source ~/.bashrc나 source ~/.profile로 환경변수에 반영한다

도커의 경우
환경변수를 지정하려면
-docker run --env sample=/sample --name=ubuntu ubuntu

Dockerfile
- ENV sample=/sample

환경변수 확인
echo $sample

포트 노출
컨테이너의 포트와 호스트의 포트를 연결

컨테이너의 80번 포트와 호스트의 80번 포트를 연결
외부에서 80번 포트로 접근하면 컨테이너의 80번 포트로 연결(포트포워딩)
docker run -p 80:80 -name=ubuntu ubuntu
-p A:B -> A는 host port, B는 container port

Dockerfile
expose 80
-> container port
컨티이너의 80번 포트를 외부에 노출한다.
-p옵션과 같이 사용 (run에서 지정)

파일을 이미지에 추가
호스트의 파일을 이미지 생성시 추가(복사)

Dockerfile
ADD ~/sample.txt /sample.txt
-호스트의 ~/sample.txt파일을 컨테이너의 /에 추가(복사)
압축파일을 지정할 경우 압축을 풀어추가 (알아서 풀어서 추가한다.)
URL을 지정하는 경우에는 압축해제 없이 추가됨

명령 수행할 사용자/폴더 지정하는
RUN/CMD/ENTRYPOINT 수행하기 전 사용자계정 지정
USER sample
-sample 사용자로 변경

RUN/CMD/ENTRYPOINT 수행하기 전 폴더 지정하는
WORKDIR ~/sample
~/sample 폴더로 변경해 아래 명령을 수행하라

볼륨연결
컨테이너의 폴더와 호스트의 물리폴더 간의 연결

물리 폴더~[홈디렉토리]/Downloads를 컨테이너의 /download폴더로 연결
docker run --name=ubuntu ubuntu -v ~/Downloads:/download
-> -v 옵션을 사용
-> -v A:B -> A는 물리디렉토리 B는 컨테이너 디렉토리

Dockerfile
VOLUME /sample
VOLUME ["/data", "/sample"]
해당 디렉토리는 컨테이너 폴더가 아닌 호스트의 물리폴더로 저장하고
-v 옵션과 같이 사용

도커 컨테이너간 연결

컨테이너간 상호연결 설정

mysql 다운로드
docker pull mysql

mysql 컨테이너 실행 (서버모드)
docker run -d -e MYSQL_ROOT_PASSWORD=kitri --name=mysql mysql

우분투 컨테이너 실행(연결)
docker run --name ubuntu -d --link mysql:mysql ubuntu
-> --link A:B C
-> 

mysql -uroot -p는 우분투에서 실행이 안된다.
client가 안 깔려 있어서이다.
apt-get install mysql-client

mysql -uroot -p -h mysql (-h)중요

host <=>[a <-> b] <=> C

환경변수
ENV TERM=xterm

[[[실행]]]
반드시 Dockerfile이라는 이름을 가지고 있어야하며 확장자도 있으면 안된다.

docker build --tag=ubuntu_nano .
--가 두개 -> 인자가 2개 (리눅스 관행)
--tag=A B -> 태그명은 A, 실행위치는 B는
-> build중 에러가 발생하면 에러 발생 위치에서 종료된다.

Successfully -> 성공
docker images
-> Dockerfile로 생성됨을 확인

docker run -it --name=ubuntu_nano1 ubuntu_nano
-> 우분투 컨테이너 하나 생성됨을 확인

#nano
->nano가 실행됨을 확인 
#echo $TERM
->xterm이 나오는 것을 확인



오픈자바설치(OpenJava)

일반적인 우분투 환경변수 설정
~/.profile이나 ~/.bashrc에 설정하고 source명령어로 반영

도커에서는
Dockerfile에서 RUN/ENV 명령어로 설정


FROM ubuntu:latest
MAINTAINER Jaesik phee <maguire1815@gmail>
RUN apt-get update
RUN apt-get install -y nano
RUN apt-get install -y openjdk-8-jdk
RUN apt-get clean			

ENV TERM=xterm

ENV JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
ENV CLASSPATH=$JAVA_HOME/lib/*:.
ENV PATH=$PATH:$JAVA_HOME/bin

apt-cache search openjdk
RUN apt-get clean	// deb가 설치되어서 삭제하고 싶은 경우 deb를 한다.

docker build --tag=openjava .

docker images

docker run -it openjava
#clear
#env
-> 환경변수 확인 // path가 정상적으로 설치되었는지 확인
#cd /usr/lib/jvm
-> openjdk가 설치되어있는지 확인
#nano Sample.java

class Sample{
	public static void main(String[] args) {
		System.out.println("Hello World");
	}
}

#cat Sample.java
#javac Sample.java
#ls
#ls -al
#java Sample
-> Hello World 실행됨

오라클 자바설치 (Orcle Java) : 공식적으로 우분투를 지원하지 않음

FROM ubuntu:latest
MAINTAINER Jaesik Phee <maguire1815@gmail>

RUN apt-get update -yes
RUN apt-get install nano

RUN apt-get install -y software-properties-common/hadoop-1
RUN add-opt-repository ppa:webupd8team/java
RUN apt-get update -y
RUN echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | \
	/usr/bin/debconf-set-selections
RUN apt-get install -y oracle-java8-installer
RUN apt-get clean

ENV TERM=xterm

ENV JAVA_HOME=/usr/lib/jvm/java-8-oracle
ENV CLASSPATH=$JAVA_HOME/lib/*:.
ENV PATH=$PATH:$JAVA_HOME/bin

docker build --tag=oracle-java

docker run -it oracle_java
#java -version
-> HotSpot -> 오라클 자바가 설치됨을 확인 가능


SSH(Secure Shell) 설정

일반적인 우분투 SSH설정
 openssh-server 설치
  - sudo apt-get install openssh-server
  
서버 실행종료
 sudo service ssh start/restart/stop
 /etc/init.d/ssh start/restart/stop
-> 도커에서는 서비스 자체가 지원이 안됨

도커에서의 SSH 설정
 도커에서는 리눅스의 service가 제대로 실행되지 않음
 별도의 방식으로 제공해야 함
 
root 계정으로 원격접속하려면
 /etc/ssh/sshd_config 파일의
  - PermitRootLogin 설정을 prohibit-password/without-password에서 yes로 수정
  
 chpasswd를 통해서 root의 비밀번호 지정
 
FROM ubuntu:latest
MAINTAINER Jaesik Phee <maguire1815@gmail>

RUN apt-get update
RUN apt-get install nano
ENV TERM=xterm

RUN apt-get install -y openjdk-8-jdk

RUN apt-get install -y openssh-server
RUN mkdir /var/run/sshd

RUN echo 'root:kitri' | chpasswd

RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]


// sed -> 대체하라는 뜻

설명
 SSH 비밀번호 지정(root/kitri)
 EXPOSE 명령어를 사용해서 외부로 포트(22) 노출
 CMD 명령을 사용해서 sshd 프로그램을 서버로 노출
  -리눅스의 서비스(service 수행) 대체


FROM ubuntu:latest

RUN apt-get update
RUN apt-get install -y openssh-server

RUN mkdir /var/run/sshd
RUN echo 'root:kitri' | chpasswd
#id 및 패스워드
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config	
RUN sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config
#root는 원래 ssh가 안되는데 풀어줌

# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
#ssh 버그가 있어서 버그픽스용 문자열이 있어야 함

ENV NOTVISIBLE "in users profile"
RUN echo "expose VISIBLE=now" >> /etc/profile

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
#직접 sshd 서버를 실행을 해줘야함

docker build --tag=ssh

docker run -d -p 22:22 ssh
-> ssh는 서버 모드이기 때문에 -d로 시작하고 -p로 포트 지정
docker ps -a
odcker rm -f 다른 이미지

docker run -d -p 22:22 ssh 
-> 실행 안되면 조금 있다가 다시 실행

ssh가 뜨면 보통 윈도우에서는 putty를 사용

ssh localhost root:kitri


MySQL

다운로드
docker pull mysql

실행
docker run -e MYSQL_ROOT_PASSWORD=kitri -d --name db mysql
// -e 환경변수를 root환경변수를 지정
// -d 옵션으로 서버 모드로 띄우기

컨테이너 접속
docker exec -it db hash

MySQL 접속
mysql -uroot -p


MySQL 컨테이너 접속

다운로드
docker pull mysql

실행
docker run -e MYSQL_ROOT_PASSWORD=kitri -d -name=db mysql
docker run --name=ubuntu --link db:db ubuntu

컨테이너 접속
docker exec -it ubuntu bash

MYSQL 접속
apt-get install mysql-client
mysql -h db -uroot -p
// -h 호스트 명령어를 지정해줘야 함


docker pull mysql

docker run -d -p 3306:3306 --env MYSQL_ROOT_PASSWORD=kitri --name=mysql1 mysql:5

docker exec -it mysql1 bash
#mysql -uroot -p 
#kitri

로그인 됨

>show databases;
>use mysql;
>show tables;

>desc slow_log;
// 스키마, 테이블의 구조를 보여줌
>select * from slow_log;
>quit
#
Ctrl + P + Q
// 빠져 나옴

docker run -it ubuntu
#apt update
#apt-get install -y mysql-client

mysql의 ipaddress를 확인해야함
docker ps -a
docker inpect mysql1 
// 172.17.0.3 IPAddress


docker ps -a
docker exec -it relaxed_gates bash
#ls
#mysql -h 172.17.0.3 -uroot -p
password:kitri
-> 원격 MySQL을 접속함

ctrl + p + q

// 한번에하는 부분
docker run -it --link mysql1:mysql1 --name=ubuntu1 ubunt
// ip주소를 확인하지 않고, 이름을 가지고 확인
// container끼리 확인함
#apt update
#apt install mysql-client
#mysql -h mysql1 -uroot -p
// mysql1과 같은 alias로 사용 가능

