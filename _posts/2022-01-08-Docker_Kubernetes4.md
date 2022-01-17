---
layout: post
title: Docker/Kubernetes - (4) docker image 이해 및 배포 
tags: [Docker, Kubernetes]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2022-01-05-Docker_Kubernetes1/logo.png
---

Enviroment: Ubuntu 18.04 
{: .box-note}
## 1. Docker image
Docker는 다음과 같이 docker hub라는 중앙 이미지 저장소에서 이미지를 다운받습니다. Docker계정만 있으면 누구든 이미지를 올리고 내려받을 수 있습니다. 이외에도 docker container로 구현되어 있는 docker private registry로 개인 서버에 이미지를 저장할 수 있는 저장소를 만들 수 있습니다만 여기서는 docker hub만을 이용하겠습니다. 

![1](https://da2so.github.io/assets/post_img/2022-01-08-Docker_Kubernetes4/1.png){: .mx-auto.d-block width="70%" :}



오늘은 다음과 같은 차례를 통해 docker image를 이해해보죠.
- docker image를 Host에서 생성
- 해당 image를 docker hub에 업로드
- 해당 image를 Host에서 pull


### 1.1 Docker image 이해와 생성

다음과 같이 ubuntu:16.04 image로 docker container를 생성하고 test라는 파일하나를 만들어 봅시다.

![1](https://da2so.github.io/assets/post_img/2022-01-08-Docker_Kubernetes4/2.png){: .mx-auto.d-block width="60%" :}


저만의 docker image를 만들기 위해서는 Host에서 <span style="color:DodgerBlue">docker commit</span>명령어를 사용합니다.

```
docker commit -a 'da2so' -m 'writing test' myubuntu  myubuntu:first
``` 

![1](https://da2so.github.io/assets/post_img/2022-01-08-Docker_Kubernetes4/3.png){: .mx-auto.d-block width="70%" :}

<span style="color:DodgerBlue">'docker images'</span>를 통해 실제로 image가 생성된 것을 확인가능하며 옵션 -a는 author의미이고 -m은 commit message이며 myubuntu:first는 생성할 이미지의 tag입니다. 이 상황에서 docker image는 기존의 docker image엿던 ubuntu:16.04와는 아래와 같은 차이를 보입니다.(캡처된 Layers 및 sha는 docker **inspect** ubuntu:16.04(or myubuntu:first)로 확인가능합니다.)


![1](https://da2so.github.io/assets/post_img/2022-01-08-Docker_Kubernetes4/4.png){: .mx-auto.d-block width="80%" :}


## 2. Docker image 배포

위에서 말씀드렸듯이 docker hub에서 이미지를 다운받기 때문에 이번에는 반대로 제가 만든 image를 업로드해봅시다. [docker hub](https://hub.docker.com/)에 들어가셔서 계정을 만드시고 테스트를 위한 repository를 다음과 같이 만들어봅니다. repo만드는 과정은 create repository를 누르셔서 진행하시면 됩니다.

![1](https://da2so.github.io/assets/post_img/2022-01-08-Docker_Kubernetes4/5.png){: .mx-auto.d-block width="90%" :}


그럼 이제 image를 업로드할 장소를 만들어놨으니 Host에서 myubuntu image를 업로드해보죠. <span style="color:DodgerBlue">'docker login'</span>을 typing하시고 docker hub에서 가입한 id, password를 입력해주면 로그인됩니다.

![1](https://da2so.github.io/assets/post_img/2022-01-08-Docker_Kubernetes4/6.png){: .mx-auto.d-block width="80%" :}


업로드하기 전에 만든 repo이름이 da2so/test_repo이기 때문에 기존 myubuntu image를 <span style="color:DodgerBlue">'docker tag'</span>명령어로 da2so/test_repo:0.0으로 바꿔줍니다. 그리고 해당 image를 docker hub에 push합니다. 

```
docker tag myubuntu:first da2so/test_repo:0.0
docker push da2so/test_repo:0.0 
```

![1](https://da2so.github.io/assets/post_img/2022-01-08-Docker_Kubernetes4/7.png){: .mx-auto.d-block width="80%" :}


마지막으로 docker hub에 잘 올라갓는지 확인하고 <span style="color:DodgerBlue">'docker pull'</span>을 통해 docker hub로부터 다운받아 봅시다.

![1](https://da2so.github.io/assets/post_img/2022-01-08-Docker_Kubernetes4/8.png){: .mx-auto.d-block width="80%" :}


