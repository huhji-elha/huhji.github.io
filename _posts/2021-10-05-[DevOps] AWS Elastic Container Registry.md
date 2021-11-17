---
layout: post
title:  "[DevOps: Kubernetes] AWS Elastic Container Registry"
date:   2021-10-05 19:22:35 +0530
categories: DevOps Kubernetes AWS Kubernetes ECR 
use_math: true
---
🌌 EKS Cluster에서 사용할 컨테이너 레지스트리 연결

_____________________________________


AWS의 Elastic Container Registry 연결하고 도커 이미지 push하기

## Repository 생성

```bash
aws ecr create-repository \
--repository-name plate-detection \
--image-scanning-configuration scanOnPush=true \
--region ${AWS_REGION}
```

![Untitled](https://user-images.githubusercontent.com/59910975/142182482-250e1df5-7e5f-48c8-a34a-df577d0cf354.png)

## Repository에 로그인

다음의 명령은 AWS ECR colsole에서 위에서 생성한 레포지토리 클릭 후 우측 상단의 '푸시 명령 보기'를 선택하여 진행해야한다.

리전 정보와 AWS CLI를 사용하여 docker 로그인하기

```bash
aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
```

flask 소스코드를 다운로드받고 해당 레포 안에서 도커를 빌드한다.

```bash
git clone https://github.com/joozero/amazon-eks-flask.git

cd ~/environment/amazon-eks-flask
   
docker build -t demo-flask-backend .
```

![Untitled](https://user-images.githubusercontent.com/59910975/142182471-2fe53a51-0ff9-4af3-81d8-7bc420a9d133.png)

이미지 빌드가 완료되면 이미지 태그를 이용해 ECR로 이미지를 푸시할 수 있다.

```bash
# docker tag를 demo-flask-backend:latest --> aws-ecr-주소/demo-flask-backend:latest로 변경한다.
docker tag demo-flask-backend:latest $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/demo-flask-backend:latest
```

도커 푸시 명령어를 통해 해당 이미지를 ECR로 푸시한다.

```bash
docker push $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/demo-flask-backend:latest
```

![Untitled](https://user-images.githubusercontent.com/59910975/142182480-f704efba-2744-47e5-ad7c-ea10f4354afb.png)

다음과 같이 이미지가 푸시된 것을 확인할 수 있다.

![Untitled](https://user-images.githubusercontent.com/59910975/142182481-f2caa630-a2d7-4cfb-b9b7-1cf2b953f422.png)