---
layout: post
title:  "[DevOps: Kubernetes] AWS EKS Install"
date:   2021-10-02 16:46:18 +0530
categories: DevOps Kubernetes AWS eksctl
use_math: true
---
🌌 AWS Cloud9 환경에서 EKS Cluster 구성하기

_____________________________________


[AWS EKS Workshop](https://awskrug.github.io/eks-workshop/prerequisites/)의 내용 중 실행되지 않는 부분을 수정하여 재구성하였습니다.

* 실습 환경은 [AWS 9Cloud](https://ap-southeast-1.console.aws.amazon.com/cloud9/home?region=ap-southeast-1)이며 싱가포르 리전에서 사용했습니다. 

* ❗ 관리자 권한을 가진 IAM 계정으로 진행해야합니다.❗ 

* 자세한 IAM 권한 설정은 [이곳](https://aws-eks-web-application.workshop.aws/ko/30-setting/100-aws-cloud9.html)을 참고해주세요.

## AWS CLI update

```bash
sudo pip install --upgrade awscli
aws --version
```

## Install kubectl

2021년 9월 기준 가장 최신 버전의 kubectl을 설치합니다.

```bash
sudo curl -o /usr/local/bin/kubectl  \
   https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/kubectl

sudo chmod +x /usr/local/bin/kubectl

# kubectl 설치 확인
kubectl version --client=true --short=true
```

![Untitled01](https://user-images.githubusercontent.com/59910975/142176673-52daa544-8473-4f66-a566-aef0568d8b65.png)

## Install aws-iam-authenticator

aws의 iam-authenticator를 설치합니다. 

```bash
curl -o aws-iam-authenticator https://amazon-eks.s3-us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/aws-iam-authenticator
chmod +x ./aws-iam-authenticator
mkdir -p $HOME/bin && cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$PATH:$HOME/bin
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc

# 설치 확인
aws-iam-authenticator help
```

## 그 외 필요한 라이브러리 설치하기

jq : json 형식의 커맨드를 다루는 유틸리티. 

bash-completion : kubectl 명령어의 자동완성 기능 사용

```bash
# jq 설치
sudo yum install -y jq

# bash-completion 설치
sudo yum install -y bash-completion
```

## EKS binary download

eksctl은 EKS 클러스터를 쉽게 생성 및 관리하는 CLI 툴입니다. Go 언어로 쓰여 있으며 CloudFormation 형태로 배포됩니다.

eksctl 바이너리 파일을 다운로드합니다.

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv -v /tmp/eksctl /usr/local/bin

# 제대로 설치 되었는지 확인
eksctl version
```

## AWS CLI 리전 설정

현재 실습하고 있는 리전을 사용하도록 설정한다.

```bash
export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')

echo "export AWS_REGION=${AWS_REGION}" | tee -a ~/.bash_profile
   
aws configure set default.region ${AWS_REGION}
```

![Untitled02](https://user-images.githubusercontent.com/59910975/142176665-0b2580ed-6adf-40be-a737-02f443b8dac4.png)

## ID를 환경변수로 설정

현재 eks 클러스터를 구성하는 계정의 ID를 환경변수로 설정한다.

```bash
export ACCOUNT_ID=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.accountId')

echo "export ACCOUNT_ID=${ACCOUNT_ID}" | tee -a ~/.bash_profile
```

## 디스크 증설

도커 이미지를 빌드하는 동안 용량 부족 문제가 발생할 수 있으므로 디스크 사이즈를 미리 증설한다.

```bash
# 디스크 사이즈를 증설하는 shell script이다.
wget https://gist.githubusercontent.com/joozero/b48ee68e2174a4f1ead93aaf2b582090/raw/2dda79390a10328df66e5f6162846017c682bef5/resize.sh
```

증설 전과 증설 후

![Untitled03](https://user-images.githubusercontent.com/59910975/142176668-687d7a93-1367-4431-be6e-7a3083ff5b7e.png)

![Untitled04](https://user-images.githubusercontent.com/59910975/142176671-ec074d8b-adce-43d3-8ce0-7e531cfa35ec.png)

## EKS Cluster config 파일 구성

eksctl을 통해 아무 설정값도 주지 않고 eksctl create cluster를 실행하면 default parameter로 클러스터가 생성된다.

여기서는 yml 템플릿 파일을 작성하여 배포한다. 템플릿을 작성하여 배포하게되면 CLI로 직접 입력하는 것보다 명시한 오브젝트들의 바라는 상태를 쉽게 파악하고 관리할 수 있다.

```bash
# 워크 디렉토리로 이동
cd ~/environment

# 다음의 파일 생성
cat << EOF > eks-demo-cluster.yaml
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: eks-demo # 생성할 EKS 클러스터명
  region: ${AWS_REGION} # 클러스터를 생성할 리젼
  version: "1.21"

vpc:
  cidr: "192.168.0.0/16" # 클러스터에서 사용할 VPC의 CIDR

managedNodeGroups:
  - name: node-group # 클러스터의 노드 그룹명
    instanceType: m5.large # 클러스터 워커 노드의 인스턴스 타입
    desiredCapacity: 3 # 클러스터 워커 노드의 갯수
    volumeSize: 10  # 클러스터 워커 노드의 EBS 용량 (단위: GiB)
    ssh:
      enableSsm: true
    iam:
      withAddonPolicies:
        imageBuilder: true # AWS ECR에 대한 권한 추가
        # albIngress: true  # albIngress에 대한 권한 추가
        cloudWatch: true # cloudWatch에 대한 권한 추가
        autoScaler: true # auto scaling에 대한 권한 추가

cloudWatch:
  clusterLogging:
    enableTypes: ["*"]
EOF
```

eks config 파일에 부여할 수 있는 다양한 속성값은 [eks 공식문서](https://eksctl.io/usage/creating-and-managing-clusters/)에서 더 확인할 수 있다.

## EKS Cluster 배포

```bash
eksctl create cluster -f eks-demo-cluster.yaml

# 클러스터 배포가 잘 되었는지 확인
kubectl get nodes
```

## Console Credential 설정 (option)

EKS 클러스터는 클러스터 접근 제어를 위해 IAM entity(사용자 또는 역할)를 사용한다. 해당 rule은 **aws-auth**라는 ConfigMap에서 실행된다. 

기본적으로 클러스터를 생성하는데 사용된 IAM entity에는 컨트롤 플레인에서 클러스터 RBAC 구성의 **system:masters** 권한이 자동적으로 부여된다.

```bash
# amazon resource number 정의하기
rolearn=$(aws cloud9 describe-environment-memberships --environment-id=$C9_PID | jq -r '.memberships[].userArn')
echo ${rolearn}

# 위 명령을 호출했을 때 assumed-role이 있으면 아래 명령을 추가 진행
assumedrolename=$(echo ${rolearn} | awk -F/ '{print $(NF-1)}')
rolearn=$(aws iam get-role --role-name ${assumedrolename} --query Role.Arn --output text)

# identiy 맵핑 생성하기
eksctl create iamidentitymapping --cluster eks-demo --arn ${rolearn} --group system:masters --username admin

# credential 관련 설정하기
kubectl describe configmap -n kube-system aws-auth
```


## EKS Cluster 삭제

클러스터를 사용하고 다시 삭제할 때

```bash
# EKS에서 실행중인 모든 서비스 목록 확인
kubectl get svc --all-namespaces

# ExternalIP값과 연결된 모든 서비스 삭제
kubectl delete svc <service-name>

# 클러스터 삭제
eksctl delete cluster --name <cluster-name>
```