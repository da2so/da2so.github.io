---
layout: post
title: Docker/Kubernetes - (10) Kubernetes 이해 및 사용
tags: [Docker, Kubernetes]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2022-01-05-Docker_Kubernetes1/logo.png
---

Enviroment: Ubuntu 18.04 
{: .box-note}
## 1. Kubernetes(k8s) 이해
Kubernetes(k8s)이 가지는 고유한 특성에 대해 알아봅시다.

#### <span style="color:DarkOrchid">모든 리스소는 object형태로 관리됨</span>

이전 글에서 swarm mode의 container 집합을 service(서비스)라고 하였습니다. K8s은 이러한 개념을 폭넓고 세밀한 단위로 사용하기 위해 <span style="color:Crimson">object</span>라는 개념을 사용합니다. 예로 container 집합(Pods), Pods을 관리하는 컨트롤러(Replica Set), 사용자(Service Account), 노드(Node) 모두를 하나의 object로 사용가능합니다. 

사용가능 한 object는 <span style="color:DodgerBlue">kubectl api-resources</span>명령어로 확인가능합니다.



#### 


![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/1.png){: .mx-auto.d-block width="90%" :}





