---
layout: post
title:  "[DevOps: Kubernetes] AWS EKS Load Balancer Controller"
date:   2021-10-05 18:23:11 +0530
categories: DevOps Kubernetes AWS Kubernetes eksctl load-balancer ALB
use_math: true
---
🌌 EKS Cluster에 AWS Load Balancer 생성하기

_____________________________________



**[AWS Load Balancing](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html)**은 쿠버네티스에서 인그레스를 생성할 때 ALB(Application Load Balancing)와 필요 자원 생성을 트리거하는 컨트롤러이다. 

쿠버네티스 온프레미스 환경에서 인그레스를 생성하면 해당 인그레스를 실행하기 위해 인그레스 컨트롤러를 직접 구현해야하는데, AWS EKS에서는 Load Balancing 컨트롤러를 통해 관리할 수 있다.

![](https://user-images.githubusercontent.com/59910975/142180872-97058371-2189-4ae9-96fb-f357067916b9.png)

_ref : [https://aws.amazon.com/ko/elasticloadbalancing/features/](https://aws.amazon.com/ko/elasticloadbalancing/features/)_


![](https://user-images.githubusercontent.com/59910975/142180880-2f481e59-2840-433f-8e78-c39cfae0db99.png)

_ref: [https://www.middlewareinventory.com/blog/kubernetes-ingress-example/](https://www.middlewareinventory.com/blog/kubernetes-ingress-example/)_


위 쿠버네티스 클러스터 구조를 EKS를 사용하여 구성하면 
**Ingress** 부분은 **AWS Application Load Balancer**로 프로비저닝되고 **Service**는 **AWS Network Load Balancer**로 프로비저닝된다.

## AWS Load Balancer Traffic mode

AWS Load Balancer에서 제공되는 트래픽 모드는 다음의 두 가지이다.

- Instance mode (default) : 노드를 ALB 대상으로 등록한다. ALB로 들어오는 트래픽은 NodePort에 의해 라우팅된 후 Pod로 프록시된다.
- IP mode : 파드를 ALB 대상으로 등록한다. ALB로 들어오는 트래픽이 직접 Pod로 라우팅된다. IP mode를 사용하기 위해서는 ingress.yaml 템플릿에 명시해야한다.

![](https://user-images.githubusercontent.com/59910975/142180904-fb50bc05-88f8-43ad-a1af-74331b7e565c.png)

_ref : [https://aws-eks-web-application.workshop.aws/ko/60-ingress-controller/100-launch-alb.html](https://aws-eks-web-application.workshop.aws/ko/60-ingress-controller/100-launch-alb.html)_

## 매니패스트 관리 폴더 생성

```bash
cd ~/environment

mkdir -p manifests/alb-controller && cd manifests/alb-controller

# 최종 폴더 위치
/home/ec2-user/environment/manifests/alb-controller
```

## 클러스터 권한 설정

Load Balancer 컨트롤러가 워커 노드 위에서 동작하기 때문에 IAM permissions를 통해, AWS ALB/NLB 리소스에 접근할 수 있도록 만들어야한다. 

자세한 내용은 [AWS IAM OIDC](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/enable-iam-roles-for-service-accounts.html) 공식문서를 참고!

먼저, 클러스터에 대한 **IAM OIDC(OpenID Connect) identity Provider**를 생성한다.

```bash
eksctl utils associate-iam-oidc-provider \
    --region ${AWS_REGION} \
    --cluster eks-demo \
    --approve
```

![Untitled](https://user-images.githubusercontent.com/59910975/142180918-5a2203c7-014e-4927-a8c9-9ac9e85cbb5f.png)

생성한 IAM OIDC 자격 증명 공급자는 IAM 콘솔 Identity providers 메뉴 혹은 아래의 명령어를 통해 확인할 수 있다.

```bash
# OIDC provider URL 확인
aws eks describe-cluster --name eksworkshop-eksctl --query "cluster.identity.oidc.issuer" --output text

# 결과값은 다음과 같은 내용을 가진다.
# https://oidc.eks.ap-southeast-1.amazonaws.com/id/319C47050CFFAF6BCFAC907ABCDEDF

# 위 결과에서 id/ 뒤에 있는 값을 복사한 뒤 다음 명령어를 입력한다.
aws iam list-open-id-connect-providers | grep 319C47050CFFAF6BCFAC907ABCDEDF

# 결과 값이 출력되면 IAM OIDC identity provider가 클러스터에 생성이 된 것. 
# 아무 값도 나타나지 않으면 생성 작업을 다시 수행해야한다.
```

AWS Load Balancer Controller에 부여할 IAM Policy를 생성한다.

```bash
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
```

AWS Load Balancer Controller를 위한 ServiceAccount를 생성한다.

```bash
eksctl create iamserviceaccount \
    --cluster eks-demo \
    --namespace kube-system \
    --name aws-load-balancer-controller \
    --attach-policy-arn arn:aws:iam::$ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --approve
```

![Untitled](https://user-images.githubusercontent.com/59910975/142180921-53f9fc9a-883e-4163-ba97-0289dec2e749.png)

## AWS Load Balancer 컨트롤러를 Cluster에 추가

```bash
# 인증서 구성을 webhook에 삽입할 수 있도록 cert-manager를 설치한다.
# Cert manager는 쿠버네티스 클러스터 안에서 TLS 인증서를 자동으로 프로비저닝 및 관리하는 오픈 소스이다. 
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.4.1/cert-manager.yaml

# Load balancer controller yaml 파일 다운로드
wget https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.2.1/docs/install/v2_2_1_full.yaml
```

위에서 내려받은 `v2_2_0_full.yaml` 파일을 열고 다음의 두 가지를 수정한다.

1. `your-cluster-name`을 현재 사용하고 있는 클러스터 이름으로 수정한다.
    
    ```yaml
    --- 중략
    spec:
        containers:
        - args:
          - --cluster-name=your-cluster-name
    ```
    
2. ServiceAccount 부분을 삭제한다. 삭제하지 않으면 컨트롤러가 배포될 때 IAM 역할이 덮어씌워지게 된다. 삭제함으로써 클러스터를 삭제했을 때 위에서 생성한 서비스 계정이 유지되는 역할도 한다. 
    
    ```yaml
    --- 다음 부분을 전부 삭제한다.
    apiVersion: v1
    kind: ServiceAccount
    metadata:
    labels:
        app.kubernetes.io/component: controller
        app.kubernetes.io/name: aws-load-balancer-controller
    name: aws-load-balancer-controller
    namespace: kube-system
    ```
    

파일을 쿠버네티스에 적용한다.

```bash
kubectl apply -f v2_2_0_full.yaml
```

![Untitled](https://user-images.githubusercontent.com/59910975/142180928-2b3ec2c9-2b88-4931-9fe1-179eb16735b2.png)

Load Balancer 컨트롤러 동작 확인!

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

![Untitled](https://user-images.githubusercontent.com/59910975/142180933-e24fa0e8-0608-4af1-9f61-3eefdd5986cd.png)

아래 명령어를 통해 service account가 생성된 것도 확인할 수 있다.

```bash
kubectl get sa aws-load-balancer-controller -n kube-system -o yaml
```

## 관련 로그 확인하기

kube-system 네임스페이스의 로그 출력하기

```bash
kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o "aws-load-balancer[a-zA-Z0-9-]+")
```

자세한 속성값 확인하기

```bash
ALBPOD=$(kubectl get pod -n kube-system | egrep -o "aws-load-balancer[a-zA-Z0-9-]+")

kubectl describe pod -n kube-system ${ALBPOD}
```

### _Reference:_

- [Application load balancing on Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html)
- [AWS 로드 밸런서 컨트롤러](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/aws-load-balancer-controller.html)
- [클러스터에 대한 IAM OIDC 공급자 생성](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/enable-iam-roles-for-service-accounts.html)
- [AWS 로드 밸러스 구축 워크샵](https://aws-eks-web-application.workshop.aws/ko/60-ingress-controller.html)
- https://github.com/jetstack/cert-manager