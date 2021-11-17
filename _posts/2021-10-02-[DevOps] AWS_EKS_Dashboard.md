---
layout: post
title:  "[DevOps: Kubernetes] AWS EKS Dashboard"
date:   2021-10-02 18:23:11 +0530
categories: DevOps Kubernetes AWS Kubernetes-dashboard
use_math: true
---
🌌 AWS Cloud9 환경에서 EKS Cluster 구성하기

_____________________________________



[쿠버네티스 공식 문서](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) 와 [AWS EKS 공식문서](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/dashboard-tutorial.html) 참고하여 작성하였습니다.

## Deploying Kubernetes Dashboard

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml
```

![Untitled](https://user-images.githubusercontent.com/59910975/142179557-817e8059-4fdf-4bc4-a7e6-787d7c54ad38.png)

## eks-admin 서비스 계정 및 클러스터 역할 바인딩

eks-admin-service-account.yaml 파일을 생성하여 다음의 내용을 붙여넣는다.

해당 매니패스트는 eks-admin이라는 서비스 계정을 정의하고 클러스터 역할을 바인딩한다.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: eks-admin
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: eks-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: eks-admin
  namespace: kube-system
```

저장한 파일을 클러스터에 바인딩한다.

```yaml
kubectl apply -f eks-admin-service-account.yaml
```

다음과 같은 출력 결과가 나오면 성공이다.

```yaml
serviceaccount "eks-admin" created
clusterrolebinding.rbac.authorization.k8s.io "eks-admin" created
```

## Set kube Proxy

설치한 쿠버네티스 대시보드는 개인 클러스터에서 실행되기 때문에 kube-proxy를 이용해 대시보드 서비스로 요청을 수락해야한다.

```bash
kubectl proxy --port=8080 --address='0.0.0.0' --disable-filter=true &
```

위 명령어를 실행하면 proxy를 이용해 8080 포트로 모든 인스턴스에서 접속이 가능하다. 

`--disable-filter=true`로 하면 XSRF 공격을 방어하는 보안 기능인 요청 필터링을 비활성화하는 것인데, 로컬이 아닌 호스트 요청 필터링을 하지 않는다. 프로덕션 환경에서는 보안의 위험이 있지만 개발 환경에서는 유용한 옵션이다.

## Dashboard Available

Cloud9의 **Tools > Preview > Preview Running Application** 를 클릭한다.

URL 입력창에 있는 주소 뒤에 다음의 URL을 붙여넣는다.

```bash
api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#!/login

# 붙여넣으면 다음과 같은 주소를 가진다. 
# https://5b3aabcdedfg38b.vfs.cloud9.ap-southeast-1.amazonaws.com/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#!/login
```

터미널에 다음 명령어를 입력하여 token 값을 가져온다.

```bash
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep eks-admin | awk '{print $1}')
```

토큰 버튼이 선택된 상태에서 토큰 입력* 창에 복사한 토큰을 붙여넣는다.

![Untitled](https://user-images.githubusercontent.com/59910975/142179547-cad58186-f640-4630-a7cd-c3f0491648fa.png)

![Untitled](https://user-images.githubusercontent.com/59910975/142179550-d7020e50-c575-4d2c-940d-b04bba0080f9.png)

토큰 인증이 되면 다음과 같은 대시보드를 확인할 수 있다.

![Untitled](https://user-images.githubusercontent.com/59910975/142179554-a3eab25a-49d1-472d-9abd-7e4c9bf89f70.png)

## kube proxy stop

생성한 kubectl proxy를 중단하는 방법은 다음과 같다.

```bash
pkill -9 -f "kubectl proxy"
```