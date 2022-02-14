---
layout: post
title: Docker/Kubernetes - (1) Docker란?
tags: [Docker, Kubernetes]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2022-01-05-Docker_Kubernetes1/logo.png
---

Enviroment: Ubuntu 18.04 
{: .box-note}
## 1. Docker란?
Docker는 GO 언어로 작성되어 있으며 linux container에 여러 기능을 추가하여 application을 컨테이너로서 쉽게 사용하게 하는 겁니다.

![docker_basic](https://da2so.github.io/assets/post_img/2022-01-05-Docker_Kubernetes1/1.png){: .mx-auto.d-block width="80%" :}

Docker의 필요성에 대한 예시로 AI개발자들은 겪어 봤을 예시를 하나 알려드립니다.

1. **Docker를 사용하지 않은 경우**
- <span style="color:#0000ff">'A'</span>라는 개발자가 자신의 Host OS에 library(pytorch 1.7, cuda 10.1 cuddn 7.1)을 설치하여 code를 작성하고 AI 모델을 학습했습니다. 그리고 github에 해당 code를 올리게 됩니다. 
- <span style="color:Crimson">'B'</span>라는 개발자가 <span style="color:#0000ff">'A'</span>가 github에 배포한 코드를 실행하려면 위에서 언급된 pytorch, cuda, cuddn이 필요하게 되죠. Docker가 없다면 해당 개발자는 <span style="color:#0000ff">'A'</span>가 환경 설정한대로 설치를 해야 code가 실행됩니다.
- 환경이 조금이라도 다르게 설치가 되었다면 <span style="color:Crimson">'B'</span>는 코드를 실행해보지 못합니다.

2. **Docker를 사용한 경우** 
- <span style="color:#0000ff">'A'</span>가 위에서 언급된 library들을 docker container안에서 설치를 하였고 docker image형태로 code와 함께 github에 올립니다.
- <span style="color:Crimson">'B'</span>는 code와 docker image를 다운받습니다. 먼저 docker image를 build하면 docker container가 생성되고 그 안에는 code를 돌리기 위해 필요한 lib이 모두 설치되어있습니다.
- <span style="color:Crimson">'B'</span>는 lib설치 없이 container안에서 <span style="color:#0000ff">'A'</span>의 코드를 실행하게 됩니다.

이 외에도 다음과 같은 이유로 Docker를 사용하게 됩니다.

## 2. Docker의 필요성

1. **application 개발과 배포의 편의성**
	- container안에서 softwared을 설치하고 설정 파일을 수정해도 Host OS에는 영향을 끼치지 않습니다.
	- container안에서 여러 작업을 하고나서 이를 운영 환경에 배포하려고 할때 docker image라는 패키지로 만들어 전달만 하면 됩니다.
	- 해당 container안의 환경 및 파일들을 다른 서버에서 똑같이 복제 가능해서 개발/운영 환경의 통합이 유연
2. **여러 application의 독립성과 확장성**
	- 여러 모듈을 독립된 형태로 구성하기 때문에 언어에 종속되지 않고 변화에 빠르게 대응, 각 모듈의 관리가 쉬워진다.
	- 웹서비스에서 database container와 web server container를 분리하는 것을 말합니다.

## 3. Docker engine 설치

위에서 언급드린 대로 저는 docker Engine과 가장 호환이 잘되는 ubuntu(18.04) 64bit를 사용합니다.

다음 명령어를 통해 docker engine을 설치합시다.

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

apt-get update
apt-get install docker-ce
```

잘 설치 되었다면 다음과 같이 버전 확인이 가능합니다.

![installation_check](https://da2so.github.io/assets/post_img/2022-01-05-Docker_Kubernetes1/2.png){: .mx-auto.d-block width="60%" :}

