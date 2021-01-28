---
layout: post
title:  "알아두면 쓸데있는 리눅스 명령어"
date:   2020-12-23 20:11:03 +0530
categories: Linux Docker Ddos-Virus
---

🤷‍♂️ 개발하면서 몇번씩 구글링하는 리눅스 명령어 모음

_____________________________________

<br/>

👀 꾸준히 업데이트 예정

### Linux 서버에 파일 전송

```bash
# window local ---> Linux Docker Server로도 가능

# 파일 전송
scp -P 포트번호 /data/전송-대상-파일 계정명@123.456.78.99:/전송받을-디렉토리/
# ex) scp -P 1212 /data/file.zip root@123.456.78.99:/root/

# 폴더 전송
scp -r -P 포트번호 /data/전송-대상-폴더 계정명@123.456.78.99:/전송받을-디렉토리/
# ex) scp -r -P 1212 /data/sample_folder root@123.456.78.99:/root/data/
```

### Ubuntu에 설치된 패키지 조회

```python
dpkg -l | grep -i {package name}
# ex. dpkg -l | grep -i docker
```

### Linux로 sftp 접속

```bash
chmod 400 ./{key_name} #key를 private 권한으로 바꾸기
sftp -i key.pem surromind@s3.yper.co.kr
get -r folder_name /local_path
```

### Linux process 조회, 삭제

```bash
# 첫번째 방법
# 현재 실행중인 프로세스 조회
ps -elf | grep python

# 삭제
kill -9 PID

# 두번째 방법
# 실행중인 프로세스 보기
ps -e

# 프로세스 죽이기
kill 3333
```

### Linux 디스크 용량 확인

```bash
df -k : 킬로바이트 단위로 현재 남은 용량을 확인
df -m : 메가바이트 단위로 현재 남은 용량을 확인
df . : 현재 디렉토리가 포함된 파티션의 남은 용량 확인

du -a : 현재 디렉토리 사용량을 파일 단위로 출력
du -s : 총 사용량 확인
du -sh * : 서브 디렉토리 기준으로 사용량 출력
```

### Linux 서버 기본 정보 확인

```python
# ubuntu cpu 정보
$ cat /proc/cpuinfo
or
$ sudo smidecode -t processor | more
# ubuntu cpu node 정보 확인
$ cat /proc/cpuinfo | grep "model name"

# ubuntu RAM 사양 확인
$ sudo dimdecode -t memory | more

# linux 전체 용량 확인 - GB 단위
$ df -P | grep -v ^Filesystem | awk '{sum += $2} END { print sum/1024/1024 " GB" }'
```

### 파일 일부만 조회하고 싶을 때

```bash
# 상위 10개 파일만 보기
ls | head -n 10

# 상위 10개 파일 모든 정보 확인
ls -lrth | head -n 10

# 하위 10개 파일만 보기
ls | tail -n 10
```

### Docker 명령어 모음

```bash
# 먼저 docker login하기
docker login

# docker 이미지 가져오기
docker pull nvidia/cuda:10.0-cudnn7-runtime-ubuntu18.04

# docker image 확인
docker images

# docker tag 변경
docker tag nvidia/cuda:10.0-cudnn7-runtime-ubuntu18.04 trainserver:cuda10.0

# docker 컨테이너 생성, 실행
docker run --gpus '"device=0,1"' --name=myname --shm-size 8G --restart=always -i -t -p 12345:22 -p 12346:6006 -p 12347:8888 -p 12348:8889 -v /data:/data 도커이미지이름 /bin/bash
```

### docker run 상세 설명
`--gpus` : docker에 할당할 GPU 번호. 한개일 땐 번호 하나만 써도 되지만 여러개일 땐 따옴표 두 개를 써줘야한다. `ex. '"device=1,2"'` <br/>
`--name=myname` : myname 자리에 docker 이름 써주기 <br/>
`--shm-size` : RAM 사이즈 할당 <br/>
`--restart=always` : Docker가 꺼지더라도 바로 다시 시작하는 옵션, `run` 명령어 후 `docker start`하지 않아도 바로 도커에 접속할 수 있다. <br/>
`-p` : Docker에서 열어주는 포트(포트 포워딩) `22`번은 외부에서 기본으로 접속하는 포트, `6006`은 tensorboard, `8888`은 jupyter notebook, `8889`는 여분 포트 (주로 장고용 포트로 사용)


### SSH 접속을 위한 Docker 초기 세팅

```bash
# 도커 내부 접속(attach)
docker attach 컨테이너아이디

# 기본 세팅
apt-get update
apt-get install sudo
sudo apt-get install vim
apt-get install net-tools vim openssh-server

# config 파일 수정
vim /etc/ssh/sshd_config
# PermitRootLogin prohibit-password  의 주석을 제거하고
# PermitRootLogin yes로 변경

# ssh로 이 컨테이너에 접속할 때 입력할 비밀번호 설정
passwd root

# ssh 열어주기
service ssh start

# 이제 외부에서 접속 가능
# ssh -p 12345 root@123.45.67.89
```

### Linux 압축

```bash
# 압축하기
tar -cvf file_name.tar folder_name : "tar" 압축
tar -zcvf file_name.tar.gz folder_name : "tar.gz" 압축
zip file_name.zip file_name : "zip" 압축
zip -r folder_name.zip folder_name : "zip" 폴더 압축

# 압축 풀기
tar -xvf fild_name.tar
tar -zxvf fild_name.tar.gz
unzip file_name.zip

# 조용하게 압축 풀기
unzip -qq filename
```

### Z shell 설정

```bash
# zshell plugin 설정하기
$ vim ~/.zshrc
```

### remove Linus package

```bash
#Remove python2.7
sudo apt purge -y python2.7-minimal

#setup python3 to default python
sudo ln -s /usr/bin/python3 /usr/bin/python
python --version
```

### Linux 바이러스 검사 프로그램 Clamav 설치, 실행

```bash
# clamav 설치
sudo apt-get install clamav

# clamav stop
sudo systemctl stop clamav-freshclam.service

# download database
sudo freshclam

# if failed download
curl -o /var/lib/clamav/daily.cvd https://database.clamav.net/daily.cvd

# clamav start
sudo service clamav-freshclam start

# 바이러스 검사 시작
clamscan -r /

# 바이러스 검사하고 감염된 파일 virus 폴더에 옮기기 (미리 폴더 만들어놓아야함)
clamscan -r / --move=/virus
```

### Traffic monitoring

```bash
# 실시간 모니터링
top

# 인터페이스 별 트래픽 I/O 모니터링
netstat -i

## RX-OK / TX-OK : 정상적인 패킷 수
## RX-ERR / TX-ERR : 체크섬 오류로 거부된 패킷 수
## RX-DRP / TX-DRP : 전체 퍼버에서 누락된 패킷 수
## RX-OVR / TX-OVR : 시스템이 바빠서 누락된 패킷 수

# 수신 포트별 트래픽 모니터링
netstat -ltu
```

### Linux cpu core check

```bash
# cpu 정보 확인
$ cat /proc/cpuinfo

# cpu 코어 전체 개수 확인
$ grep -c processor /proc/cpuinfo
> 48
# 가상 cpu 코어 개수는 48.

# 물리 cpu 수 확인
$ grep "physical id" /proc/cpuinfo | sort -u | wc -l
> 2

# cpu 당 물리 코어 수 확인
$ grep "cpu cores" /proc/cpuinfo | tail -1
> 12

# 물리 cpu 수 : 2
# 물리 cpu 당 물리 코어 수 : 12
# 전체 물리코어 수 : 24 (2 cpu x 12 core)
# 전체 가상코어 수 : 48 (2 cpu x 12 core x 2 thread)
```

### DDos Virus infection on Ubuntu 18.04

```bash
# 어떤 파일이 트래픽을 먹고있는지 확인
top

# 해당 파일 ("obguwqpsgn" 등의 의미없는 문자 나열로 되어 있음) 위치 탐색
locate obguwqpsgn

>output
/etc/init.d/obguwqpsgn
/bin/obguwqpsgn

# 해당 위치로 가서 실행 되었는지 여부를 확인해봅시다.
ll -tr

#가장 마지막에 cron.hourly 혹은 파일 이름이 뜰 것인데 둘 다 들어가서 확인해봅시다.

# 작동 중지 (삭제하면 다시 살아납니다)
kill -STOP 25049
```

- `ll -tr` 로 확인

![img03](https://user-images.githubusercontent.com/59910975/106130464-b7bbc880-61a4-11eb-8fe7-a6d50f331417.png)


- `cat /etc/init.d/파일이름` 내용 확인

![img01](https://user-images.githubusercontent.com/59910975/106130459-b5f20500-61a4-11eb-96ec-828028f28106.png)

- `cat /etc/cron.hourly/gcc.sh` 내용 확인

![img02](https://user-images.githubusercontent.com/59910975/106130463-b7233200-61a4-11eb-8505-95b9b9f2bc03.png)

```bash
$ top
# 실행 후 더이상 PID에 바이러스 파일이 뜨지 않는 것을 확인하고(kill -STOP 명령 후)

$ rm -f /etc/cron.hourly/gcc.sh
$ rm -f /bin/obguwqpsgn
$ rm -f /etc/init.d/obguwqpsgn

$ reboot
# 끝!
# 이렇게 해도 바이러스가 계속 남아있다면 리눅스 초기화뿐..
```