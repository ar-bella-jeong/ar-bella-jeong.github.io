---
title: "[Kubernetes][AWS] Amazon EKS cluster access"
date: 2021-09-04 16:19:28 -0400
categories: kubernetes
tags:
  - kubernetes
  - aws
  - security
comments: true
---
Amazon EKS의 인증은 kubeconfig file만 사용하여 access 권한을 얻을 수 있는 다른 많은 cluster와 약간 다르다.

 하기 내용에서 Amazon EKS의 인증이 어떻게 작동하는 지 밝힐 것이다.
 
## Authentication and authorization in Amazon EKS
Amazon EKS는 IAM을 사용하여 Kubernetes 클러스터에 인증을 제공한다. **kubectl** 을 사용하여 **내부** 에서 Amazon EKS와 상호 작용할 때를 살펴보자.
![Amazon EKS 클러스터에서 클라이언트의 인증 및 권한 부여](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2020/06/07/RBAC-IAM.png)

먼저 cluster에 access할 수 있도록 kubeconfig file 생성이 필요하다. 하기는 그 구성 file의 예시이다.
![Kubeconfig 구성 파일](https://vnt-software.com/wp-content/webpc-passthru.php?src=https://vnt-software.com/wp-content/uploads/2020/08/Kubeconfig-Config-File.png&nocache=1)
cluster section에 certificate 정보와 cluster api server endpoint url이 있다. 
kubernetes 설치 시 일반적으로 단일 root CA가 존재하는데, 이 인증서는 kubelet, api-server, controller 등과 같은 component간에 통신에 사용되는 하위 SSL certificate를 생성할 때 쓰인다. certificate-authority-data 항목엔 root CA의 인증서 내용이 들어간다. 이는 cluster server 접근 시 내려오는 server 인증서를 verify 하기 위함이다.

user section에서 흥미로운 것을 볼 수 있는데, user name대신 실제로 aws cli명령을 실행하는 exec 명령이 있음을 알 수 있다.
```bash
aws –region us-east-1 eks get-token –cluster-name test-cluster
```
![Amazon EKS Kubernetes Cluster](https://vnt-software.com/wp-content/webpc-passthru.php?src=https://vnt-software.com/wp-content/uploads/2020/08/Amazon-EKS-Kubernetes-Cluster.png&nocache=1)
보다 싶이 cluster에 access하는 데 사용할 수 있는 token을 생성하는 command이다. 
이것은 실제로 AWS STS API 인 GetCallerIdentity의 호출 결과 이다.

AWS CLI 버전 1.18.49 이상에서 사용할 수 있는 **aws eks get-token** 명령 및 AWS IAM Authenticator를 사용하여 인증 token을 가져온다. 이 token은 **Authorization** header에 전달 되는데 따라서, Amazon EKS는 [token 인증 webhook](https://ar-bella-jeong.github.io/kubernetes/kubernetes-webhook-autentication/)을 사용하는 것것이다. 내부적으로 Kubernetes RBAC에 의존하는데, 하여 IAM(role/user)과 RBAC(user/group)간의 통합을 위해 [**aws-auth** ConfigMap](https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html) 을 사용한다.

아무래도 file 기반이다 보니 매번 등록하는 것이 번거로울 수 있다. 이에 자동 등록을 위한 [architecture](https://aws.amazon.com/ko/blogs/containers/kubernetes-rbac-and-iam-integration-in-amazon-eks-using-a-java-based-kubernetes-operator/)도 aws blog에서 제공 중이다.

> 출처
> - https://vnt-software.com/accessing-an-amazon-eks-kubernetes-cluster/
> - https://m.blog.naver.com/alice_k106/221492934710
> - https://aws.amazon.com/ko/blogs/containers/kubernetes-rbac-and-iam-integration-in-amazon-eks-using-a-java-based-kubernetes-operator/
