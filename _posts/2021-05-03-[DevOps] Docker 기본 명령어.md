---
layout: post
title:  "[DevOps: Docker] 도커 이미지와 컨테이너 기본 명령어"
date:   2021-05-03 23:03:09 +0530
categories: DevOps Docker Container
use_math: true
---
🐋 Docker Image(pull, inspect, tag, search, rm) & Container(run, start, rm)

_____________________________________


## Docker Image와 Container를 다루는 기본 명령어

### Docker Image 다운로드

```bash
docker image pull centos
# docker image centos -a
# docker image pull gcr.io.tensorflow/tensorflow
```

- 태그명을 생략하면 최신판(latest) 버전을 다운로드한다.
- `-a` 옵션을 사용하면 모든 태그를 다운로드할 수 있다.
- 이미지명 대신 URL을 사용할 수도 있다. `https://` 이후 주소만 사용한다.

<br>

### Docker Image 목록 확인

```bash
docker image ls
```

- `-all`, `-a` : 모든 이미지 표시
- `--digests` : 도커 레지스트리에 업로드된 이미지 고유 식별자인 다이제스트 표시
- `--no-trunc` : 결과를 모두 표시
- `--quiet` , `-q` : Docker image ID만 표시

Ctrl + P + Q : 컨테이너 실행 중에 유지하고 있는 상태로 잠깐 나가는 방법

<br>

### Docker Image 상세 정보 확인

```bash
docker image inspect centos:latest

# OS 정보만 확인
docker image inspect --format="{{ .Os}}" centos:latest
```

- JSON 형식으로 결과가 출력된다.
- `--format` 옵션으로 key를 통해 확인하고 싶은 value를 출력할 수 있다.

![image01](https://user-images.githubusercontent.com/59910975/133795943-93811f44-02a8-4ffc-a9d2-2e28d9b8cd52.png)

![image02](https://user-images.githubusercontent.com/59910975/133795947-e5f9b938-e525-4d85-9d56-c42e7de90b0f.png)

<br>

### Docker Image 태그 달기

```bash
# Docker Hub 사용자명/컨테이너명:태그
docker image tag nginx jhhuh/webserver:1.0
```

- Tag를 붙인 이미지와 원본 이미지의 IMAGE ID는 달라지지 않는다.

<br>

### Docker Image 검색

```bash
docker search nginx
# docker search nginx --limit 5
# docker search nginx --filter=starts=100
```

- `--no-trunc` : 결과 모두 표시
- `--limit` : n 건의 검색 결과 표시
- `--filter=stars=n` : 스타 수 n 이상만 표시

<br>

### Docker Image 삭제

```bash
docker image rm nginx

# image ID : 0716d6ebcc1a
# docker image rm 071
```

- `--force` , `-f` : 이미지를 강제로 삭제
- `--no-prune` : 중간 이미지는 삭제하지 않음
- docker image 이름 대신 image ID를 지정해도 된다. 모든 자리가 아니라 처음 3자리만 지정해도 된다.

```bash
# 사용하지 않은 이미지를 삭제할 때
docker image prune
```

- `--a` , `-a` : 사용하지 않은 이미지 모두 삭제
- `--force`, `-f` : 강제로 삭제

---

<br>

### Container Lifecycle

![image03](https://user-images.githubusercontent.com/59910975/133795952-08c76d94-fe27-4eaa-a7b6-75489ed5cb3b.png)

완벽한 IT 인프라 구축을 위한 Docker

<br>

### Container Run — 컨테이너 생성&실행

`run` 명령어는 컨테이너를 생성한 뒤 실행까지 진행한다.

```bash
# docker run 매뉴얼 확인
man docker-run

# 기본 명령어 구조
docker container run [options] [이미지명]

# centos 컨테이너를 실행한 후 캘린더(/bin/cal) 출력하기
docker pull centos
docker container run -it --name test01 centos /bin/cal
```

- `--attach` , `-a` — container run의 기본 옵션. 기본적으로 attach를 한 상태, 즉 포그라운드로 실행된다.
- `--detach` , `-d` — 컨테이너를 생성한 후 백그라운드에서 실행한다.
- `-it` — 콘솔에 결과를 출력하는 옵션( `--interactive` : 표준 출력 열기 + `--tty` : 단말기 디바이스 사용)
- `--name` — 컨테이너 이름 지정

<br>

### Container Run — 백그라운드 실행

`docker container start` 명령어를 쓰면 자동으로 백그라운드에서 실행된다.

`docker run` 의 기본 옵션은 포그라운드이다. 즉, attach를 한 상태로 실행된다.

```bash
# docker pull nginx
docker run --name webserver2 nginx:latest
```

아래 실행 결과와 같이 webserver2 컨테이너로 접속되는것을 확인할 수 있다.

![image04](https://user-images.githubusercontent.com/59910975/133795957-34ea8545-ab83-437c-9e61-1fa92a321389.png)

- `--detach` , `-d` — 백그라운드에서 실행하는 옵션
- `--user` , `-u` — 사용자명 지정(사용자를 새로 생성하는 옵션은 아니다)
- `--rm` — 컨테이너 실행 후 자동 삭제
- `--restart` — 명령 실행 결과에 따라 컨테이너를 재시작하는 옵션. 컨테이너가 비정상적으로 종료되었을 때 재시작하게 할 수 있다. `--rm` 옵션과 동시에 사용할 수 없다.
    - `no` : 재시작하지 않음
    - `on-failure` : 종료 스테이터스가 0이 아닐 때 (명령 실행이 실패했을 때) 재시작한다.
    - `on-failure:n` : 종료 스테이터스가 0이 아닐 때 n번 재시작한다.
    - `always` : 항상 재시작한다.
    - `unless-stopped` : 최근 컨테이너가 정지 상태가 아니라면 항상 재시작한다.

```bash
# 컨테이너를 백그라운드로 실행하여 컨테이너 실행 상태를 유지한다
# -d : detach 옵션
docker run --name webserver3 -d nginx:latest

# 컨테이너를 백그라운드로 실행하고, 입출력 가능하게 한다.
# -dit 옵션
docker container run --name ubuntu -dit ubuntu:latest

# 컨테이너를 생성한 뒤 바로 삭제
docker run --name hello --rm hello-world:latest
```

![image05](https://user-images.githubusercontent.com/59910975/133795958-9df54f8f-729a-4bad-965a-af5b26e6edec.png)

`--restart` 옵션에서 언급한 종료 스테이터스는 linux 명령 실행 결과를 0, 1, 2로 반환하는 코드이다.

예를 들어 `ls` 명령어를 실행했을 때 Exit status가 **0이면 정상**적으로 디렉토리 내용을 출력한 것이고, **1이면 서브 디렉토리에 접근할수 없는 등의 작은 에러**, **2는 명령어 실행 자체가 실패한 에러** 발생이다.

```bash
# user 권한으로 다음 명령어를 실행한 후
ls /root

# linux 상의 exit status를 호출하면
echo $? --> 2가 나온다.

# 이를 활용하여 다음 명령어에서 sleep 10 명령이 실패할 경우 restart되도록 설정할 수 있다.
docker run --restart on-failure -d centos:latest sleep 10
```

<br>

### Docker container 삭제

```bash
# docker container ls 했을 때 나오는 모든 도커 컨테이너 삭제
docker container stop `docker container ls --quiet`
docker container rm `docker container ls -a --quiet`
```

<br>

Reference:

- [완벽한 IT 인프라 구축을 위한 Docker](http://www.yes24.com/Product/Goods/64728692)
- [Docker Document](https://docs.docker.com/engine/reference/builder/)