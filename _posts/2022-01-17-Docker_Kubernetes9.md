---
layout: post
title: Docker/Kubernetes - (9) Kubernetes란?
tags: [Docker, Kubernetes]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2022-01-05-Docker_Kubernetes1/logo.png
---

Enviroment: Ubuntu 18.04 
{: .box-note}
## 1. Kubernetes(k8s)란?
Kubernetes(k8s)는 docker swarm mode처럼 여러 대의 docker host를 하나의 cluster로 만들어 준다는 것은 같지만 세부적으로 폭넓은 기능을 제공한다는 점이 다릅니다. 구체적으로 특징은 다음과 같습니다. (K8s라는 표기는 "K"와 "s"와 그 사이에 있는 8글자를 나타내는 약식 표기)

- 서버 자원 clustering, 마이크로서비스 구조의 container 배포, 서비스 장애 복구 등 container기반의 서비스 운영에 필요한 폭넓은 오케스트레이션(Orchestration) 기능을 제공
  -  Orchestration: container의 배포, 관리, 확장 ,네트워킹을 자동화하는 기술로, 복잡한 container 환경을 효과적으로 관리하기 위한 도구
- 구글, 레드햇을 비롯한 많은 오픈소스에서 k8s의 소스코드에 기여하므로 성능과 안정성 보장
- Persistent volume, scheduling, 장애 복구, auto scaling, service discovery, ingress등 대부분의 기능과 컴포넌트를 사용자가 직접 customizing가능
- CNCF(Cloud Native Computing Foundation) 및 다른 클라우드 운영 도구들과 쉽게 연동되므로 확장성 
 
하지만 많은 기능을 지원하다보니 다른 오케스트레이션 툴들보다 훨씬 복잡하고 어려울 수 있습니다. 그리고 대규모 서비스에 적합하기 때문에 서비스 규모에 따라 k8s가 오버 엔지니어링 될 수 있음을 명심해야합니다.


## 2. Kubernetes 버전 환경

개발 용도(minikue)가 아닌 실제 서비스 테스트 또는 운영 용도로 k8s를 사용할 수 있도록 설치를 진행하겠습니다. k8s의 사용 환경은 크게 두가지입니다. 첫째는 AWS, GKE등의 클라우드 플랫폼 환경이고 두번째는 자체적으로 보유한 자체 서버(On-premise) 환경입니다. 

![1](https://da2so.github.io/assets/post_img/2022-01-17-Docker_Kubernetes9/1.png){: .mx-auto.d-block width="90%" :}



- On-premise 환경
  - k8s를 포함하여 모든 인프라를 직접관리 해야함
  - 세부적인 설정까지 가능하지만 모든 관리를 해야하므로 운영 및 유지보수 힘듬
- 클라우드 플랫폼
  - 서버 인스턴스만을 사용해 Kubernetes를 설치할 지, k8s자체를 서비스로서 제공하는 managed 서비스를 사용할지 결정이 필요
  - 전자인경우 서버, 네트워크 등 인프라에 대한 관리는 AWS, GCP와 같은 클라우드 제공에게 맡기되, 쿠버네티스의 설치 및 관리를 직접 수행
- k8s자체를 클라우드 서비스로서 사용
  - AWS의 EKS(Elastic Kubernetes Service), GCP의 GKE(Google Kubernetes Engine) 등의 managed 서비스를 이용한다며 k8s의 설치 및 관리까지도 클라우드 제공자가 담당
  - 너무 다해주니 공부하는 입장이면 쓰지말자...


환경에 맞는 설치 방법은 다음과 같습니다.

![1](https://da2so.github.io/assets/post_img/2022-01-17-Docker_Kubernetes9/2.png){: .mx-auto.d-block width="90%" :}



## 3. Kubernetes 설치

저는 실제 서비스와 비슷하도록 하기 위해 여러 개의 서버를 이용해 k8s cluster를 설치하는 방법 중 하나인 <span style="color:Crimson">kubeadm</span>으로 해보겠습니다.
<span style="color:Crimson">kubeadm</span>은 minkube 및 kubespray와 같은 설치 도구도 내부적으로 kubeadm을 사용하고 있고 on-premise환경, 클라우드 인프라 환경에 상관없이 리눅스 서버라면 사용가능합니다. k8s 사용을 위해 다음과 같은 환경을 구성하였습니다.

- 마스터 노드 1개, 워커 노드 3개 (가상머신을 통해)
  - 모든 서버(노드)의 시간이 ntp를 통해 동기화 되어있음을 확인
  - 모든 서버가 MAC주소가 다른 지 확인
  - 모든 서버가 2GB 메모리, 2 CPU이상의 자원을 가짐
  - **swapoff -a** 명령어를 통해 모든 서버에서 메모리 swap을 비활성합니다.
- 모든 서버는 우분투 18.04로 통일

사용할 서버의 IP과 호스트 이름 (역할)은 다음과 같습니다.

```
192.168.26.148 manager (마스터 노드)
192.168.26.145 worker1 (워커 노드)
192.168.26.144 worker2 (워커 노드)
192.168.26.149 worker3 (워커 노드)
```

#### Kubernetes 저장소 추가 

k8s를 설치할 모든 노드에서 다음 명령어들을 통해 저장소를 추가합니다.

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
```

#### kubeadm 설치

모든 노드에 docker의 최신버전 설치는 다음 명령어를 통해 진행합니다. 또한 k8s에 필요한 패키지를 설치합니다.

```
# docker install
sudo wget -qO- http://get.docker.com/ | sh
# package install 
apt-get update
apt-get install -y kubelet kubeadm kubectl kubernetes-cni
```

#### Kubernetes 클러스터 초기화 

마스터 노드로 사용할 host에서 다음 명령어를 통해 clutser를 초기화 합니다. 

```
kubeadm init --apiserver-advertise-address 192.168.26.148 --pod-network-cidr=172.31.0.0/16
```

- **--apiserver-advertise-address**: 다른 노드가 마스터에게 접근할 수 있는 IP주소
- **--pod-network-cidr**: k8s에서 사용할 container의 네트워크 대역 (서버의 대역과 중복되지 않도록 함)

![1](https://da2so.github.io/assets/post_img/2022-01-17-Docker_Kubernetes9/3.png){: .mx-auto.d-block width="90%" :}


위의 사진 중간의 highlight쳐져 있는 3줄의 명령어를 복사해 마스터 노드에 실행합니다. 그리고 다음사진은 맨 마지막의 highlight된 명령어는 워커 노드들에서 실행한 모습입니다.


![1](https://da2so.github.io/assets/post_img/2022-01-17-Docker_Kubernetes9/4.png){: .mx-auto.d-block width="100%" :}


위의 **init**와 **join**명령어로부터 *The kubelet is not running*라는 에러가 생기면 [이글](https://syhwang.tistory.com/50)을 참조하여 따라해서 해결하고 [이글](https://likefree.tistory.com/13)을 통해 k8s 초기화해주고 다시 init해주면 된다.
{: .box-note}


#### container 네트워크 애드온 설치 

네트워크 에드온은 CNI(Container Network Interface)라고도 합니다. k8s Cluster 내부는 마스터 노드에 의해 여러 container가 생성 삭제 복구를 반복하고 있는데 그에 따라 각 container의 고정적이지 않고 재할당이 빈번함 이러한 특징을 해결하기 위해 k8s Cluster는 가상 네트워크가 구성되어 있는데 기본적으로는 Worker Node의 kube-proxy 가 네트워크를 관리하지만 보다 효율적인 네트워크 환경을 구성하기 위해 사용되는 것이 <span style="color:Crimson">네트워크 에드온</span>입니다. 


k8s의 container 간 통신을 위해 여러 오버레이 네트워크를 사용할 수 있지만 여기서는 calico를 사용합니다. 마스터 노드에서 calico.yaml을 다운받아 줍니다.

```
# version 3.21 calico.yaml 다운로드 (2022.1월 기준)
wget https://docs.projectcalico.org/v3.21/manifests/calico.yaml
```

그리고 vi(vim)을 이용해 **CALICO_IPV4POOL_CIDR**을 찾아 아래와 같이 주석처리를 풀어주고 내용을 추가합니다.

![1](https://da2so.github.io/assets/post_img/2022-01-17-Docker_Kubernetes9/6.png){: .mx-auto.d-block width="100%" :}


위의 192.168.126.2는 마스터 노드의 게이트 웨이로 다음 명령어로 확인가능합니다. 

```
route -n | grep 'UG[ \t]' | awk '{print $2}'
```

이제 calicp.yaml을 통해 플러그인을 설치합니다.

```
kubectl apply -f calico.yaml
```

{: .box-note}
**Calico란?** container 및 기본 host 기반 워크로드를 위한 오픈 소스 네트워킹 및 네트워크 보안 솔루션이며 캡슐화 또는 오버레이없이 구축되어 고성능의 대규모 데이터 센터 네트워킹을 제공 할 수 있다. 또한 Calico는 분산 방화벽을 통해 k8s 포드에 대해 세분화 된 의도 기반 네트워크 보안 정책을 제공함


설치가 정상적으로 완료되었는지 확인하기 위해 다음 명령어로 k8s 핵심 컴포넌트들이 모두 running상태인지 확인합니다. 또한 <span style="color:DodgerBlue">kubectl get nodes</span>를 통해 등록된 모든 노드도 확인가능합니다. 

![1](https://da2so.github.io/assets/post_img/2022-01-17-Docker_Kubernetes9/5.png){: .mx-auto.d-block width="100%" :}

calico가 설치 유무에 따라 노드간의 통신이 가능해지냐 마느냐를 결정합니다. 다음은 calico 플러그인 설치에 따라 마스터 노드에서 워커노드에 ping이 잘 되는지 아닌지에 대한 비교입니다. 

![1](https://da2so.github.io/assets/post_img/2022-01-17-Docker_Kubernetes9/7.png){: .mx-auto.d-block width="100%" :}


마지막으로 kubeadm으로 설치된 k8s는 각 노드에서 다음 명령어로 삭제 가능하다. k8s설치 도중 오류 발생했거나 테스트용 k8s 클러스터를 삭제할때 사용합니다.

```
kubeadm reset
```


