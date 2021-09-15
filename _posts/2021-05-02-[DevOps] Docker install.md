---
layout: post
title:  "[DevOps: Docker] Linux Docker Install"
date:   2021-05-02 18:35:28 +0530
categories: DevOps Docker nginx
use_math: true
---
🐋 CentOs 환경에서 Docker 설치, Docker 정보 확인, nginx 웹 서버 구동하기

_____________________________________


# Docker 설치하고 미니 웹 서버 구동하기

### VirtualBox - CentOs 서버에서 진행합니다.

더 자세한 설명은 [Docker Documents](https://docs.docker.com/engine/install/centos/) 에서 확인할 수 있습니다.

<br>

### 1. 이전 도커 패키지 제거

이전 도커가 이미 설치되어 있다면 다음 명령어를 통해 도커 패키지를 제거해줍니다.

```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
<br>

### 2. 도커 repository 설치

`yum` 명령어로 도커 repository를 설치합니다.

```bash
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

![image01](https://user-images.githubusercontent.com/59910975/133408657-8fe72b25-497d-4875-9769-f32af43fdc3a.png)

nightly 버전이나 임시 레포지토리를 설치하기 위해서는 다음의 명령어를 통해 설치하면 됩니다. (선택사항)

```bash
## Optional
# nightly 버전의 경우
sudo yum-config-manager --enable docker-ce-nightly

# 임시 저장소의 경우
sudo yum-config-manager --enable docker-ce-test

# nightly or 임시 저장소를 비활성화할 때는 --disable 옵션을 사용
sudo yum-config-manager --disable docker-ce-nightly
```

레포지토리 설치 후 다음 명령어를 통해 `docker-ce-stable` 레포가 있는지 확인합니다.

```bash
# yum 저장소 확인 명령어
yum repolist
```

![image02](https://user-images.githubusercontent.com/59910975/133408659-f47f5423-b23e-46c9-a687-9511fae61b76.png)

<br>

### 3. 도커 엔진 설치

레포 설치된 것을 확인 후 도커 엔진을 설치합니다.

```bash
sudo yum install docker-ce docker-ce-cli containerd.io
```

용량이 이정도 되는데 설치해도 되냐는 물음에 y 입력해줍니다.

![image03](https://user-images.githubusercontent.com/59910975/133408660-de94955d-6d90-43c3-b8fc-90bdcb1f75e1.png)

설치 도중 도커의 키가 `060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35` 와 매치되냐고 물어보는데 맞다면  `y`를 입력하고 계속 설치하면 됩니다. (대소문자 구분 안해도 됩니다!)

![image04](https://user-images.githubusercontent.com/59910975/133408661-a0741cc6-579f-4d25-941c-c72e4e09d141.png)

<br>

### 4. 도커 그룹에 현재 계정 추가

도커 설치가 완료된 후, docker group에 현재 user 계정을 추가합니다. 

```bash
# docker 그룹 확인
cat /etc/group | grep docker

# docker 그룹에 계정 추가
# sudo usermod -aG docker <계정 id>
# 현재 계정은 'user'이므로 다음과 같이 docker 그룹에 user를 추가합니다.
sudo usermod -aG docker user
```
<br>

### 5. Docker "Hello World" 실행

Docker 이미지를 받은 후 컨테이너에서 "Hello World"를 출력합니다.

```bash
docker container run centos:latest /bin/echo 'Hello World'
```

![image05](https://user-images.githubusercontent.com/59910975/133408663-5e95c069-c182-4e48-a435-aa5ccaa49338.png)

docker 명령을 실행하면 centos의 이미지가 로컬 환경에 없는 경우 Docker Repository에서 이미지를 다운로드합니다. 

다운로드된 이미지를 바탕으로 컨테이너를 시작하여 `/bin/echo 'Hello World'` 명령이 실행된 것을 확인할 수 있습니다.

다른 옵션을 붙이지 않았으므로 이때 해당 명령 실행이 완료되면 컨테이너는 삭제됩니다.

<br>

### 6. Docker 정보 확인

- **Docker 버전 확인하기**
    - Docker는 클라이언트/서버 아키텍처를 선택하고 있어서 Docker 클라이언트와 Docker 서버가 Remote API를 경유하여 연결되어 있습니다.
    - 따라서, docker 명령은 서버로 보내져 처리됩니다.

```bash
# 도커 버전 명령어를 통해 docker 버전 뿐 아니라 Go 버전도 체크할 수 있습니다.
docker version
```

![image06](https://user-images.githubusercontent.com/59910975/133408665-32df6aa8-7cc4-4622-9889-b779cc4f9949.png)

<br>

- **Docker 실행 환경 정보 확인하기**

```bash
# 현재 실행 중인 container 수와 Storage Driver 종류, OS, Architecture 종류를 확인할 수 있습니다.
docker system info
```

![image07](https://user-images.githubusercontent.com/59910975/133408666-aa0bb7d7-a673-4f83-b4bb-132eff8cd6e7.png)

<br>

- **Docker 디스크 사용 현황**

```bash
docker system df
# 상세한 내역을 확인하려면 -v 옵션을 붙이면 된다.
# docker system df -v
```

![image08](https://user-images.githubusercontent.com/59910975/133408638-79b14ae6-1415-433f-a80e-df76be53d998.png)

<br>

### 7. 웹 서버 구동하기

nginx 이미지를 이용해 웹 서버 컨테이너를 구동해봅시다.

먼저, nginx 도커 이미지를 다운로드합니다.

```bash
docker pull nginx
# 다운로드 완료 후 docker image ls 명령어로 확인
```

![image09](https://user-images.githubusercontent.com/59910975/133408643-d2b46186-7dbe-409e-82f2-144bee9ada46.png)

<br>

다운로드 받은 nginx 이미지를 가동합니다.

```bash
# 'mini-web-server'라는 이름의 컨테이너를 HTTP 80번 포트로 열어줍니다.
docker container run --name mini-web-server -d -p 80:80 nginx
```

![image10](https://user-images.githubusercontent.com/59910975/133408645-d4814f5f-de3b-4e81-bb46-0a4007394568.png)

`http://localhost:80` 에 접속해보면 다음과 같이 HTTP 프로토콜을 이용하여 localhost 주소의 80번 포트로 액세스되는 것을 확인할 수 있습니다.

![image11](https://user-images.githubusercontent.com/59910975/133408646-bb0ab7be-787c-4a8c-a70a-91bd94c2fcb5.png)

<br>

다음 명령어를 사용하여 가동중인 nginx 서버를 모니터링할 수 있습니다.

```bash
# mini-web-server의 서버 상태 확인
docker container ps

# 서버의 상세 정보 확인
docker container stats mini-web-server
```

![image12](https://user-images.githubusercontent.com/59910975/133408648-786fd496-1f06-4f13-bd25-3bea1457dec1.png)

![image13](https://user-images.githubusercontent.com/59910975/133408649-c473bfe0-4ff6-4281-8098-8bcfc92c7f2b.png)

<br>

웹서버 구동을 정지할 때는 다음의 명령어를 사용합니다.

```bash
docker stop mini-web-server
```

![image14](https://user-images.githubusercontent.com/59910975/133408653-e3b8c05e-e004-439c-91fc-9cf4a0d66a88.png)

<br>

Reference:

- [Docker Documents](https://docs.docker.com/engine/install/centos/)
- [완벽한 IT 인프라 구축을 위한 Docker](http://www.yes24.com/Product/Goods/64728692)