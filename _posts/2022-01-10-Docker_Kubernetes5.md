---
layout: post
title: Docker/Kubernetes - (5) Dockerfile 
tags: [Docker, Kubernetes]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2022-01-05-Docker_Kubernetes1/logo.png
---

Enviroment: Ubuntu 18.04 
{: .box-note}

## 1. Dockerfile

이전 글과 같이 container에서 애플리케이션이 동작하는 환경을 만들면 일일이 container안에 들어가서 설치를 위한 수작업을 해서 image로 commit해야합니다.(위의 그림) 하지만 dockerfile를 사용한다면 추가해야하는 패키지, 명령어, 코드 등을 dockerfile에 저장해놓고 build를 통해 위의 작업을 간소화할 수 있습니다. (아래 그림)

![1](https://da2so.github.io/assets/post_img/2022-01-10-Docker_Kubernetes5/1.png){: .mx-auto.d-block width="90%" :}


### 1.1 Dockerfile 생성

다음과 같이 workspace라는 디렉터리와 예제 html, dockerfile를 만들어 봅시다.

![1](https://da2so.github.io/assets/post_img/2022-01-10-Docker_Kubernetes5/2.png){: .mx-auto.d-block width="60%" :}

- **FROM**: 생성할 이미지의 베이스가 될 image
- **MAINTAINER**: author 및 developer 정보 (docker 1.13.0이후 depreciated)
- **LABEL**: 이미지의 metadata로 key=value 형태로 저장
- **RUN**: 컨테이너 내부에서 해당 명령어를 실행
	- apt-get install apaches2 명령어에서 설치할 것인지를 선택하는 Y/N을 yes로 설정하는 -y추가됨
	- ['실행가능한 파일', 'args 1', 'args 2']형식으로도 가능
- **ADD**: 파일을 이미지에 추가, 추가하는 파일은 Dockerfile이 위치한 디텍터리인 context(뒤에서 설명!)에서 가져옵니다.
	- test.html을 container안의 /var/www/html에 저장
- **WORKDIR**: 명령어를 실행할 디렉터리 
- **EXPOSE**: Dockerfile의 빌드로 생성된 이미지에서 expose할 포트 설정
- **CMD**: container가 시작될 때마다 실행할 명령어
	- Dockerfile에서 한번만 사용가능
	- apachectl -DFOREGROUND를 통해 컨테이너 시작시 자동으로 아파치 웹서버가 시작




### 1.2 Dockerfile build

```
docker build -t mybuild:0.0 ./
```

-t 옵션은 image의 이름을 설정하는 것이고 끝의 인자(./)에는 Dockerfile이 저장된 경로입니다.

![1](https://da2so.github.io/assets/post_img/2022-01-10-Docker_Kubernetes5/3.png){: .mx-auto.d-block width="60%" :}

위에서 보이는 step은 새로운 container가 생성됨을 의미하고 이전 step에서 생성된 이미지에 더하여 새로운 컨테이너가 layered되는 것입니다. 그래서 Dockfile의 명령어 줄 수만큼 layer가 존재하며 중간에 container도 같은 수만큼 생성되고 삭제(위의 Removing intermediate ...)됩니다.

![1](https://da2so.github.io/assets/post_img/2022-01-10-Docker_Kubernetes5/4.png){: .mx-auto.d-block width="60%" :}


image를 build하면 docker는 먼저 build context를 읽어들이는데 이는 image를 생성하는 데 필요한 각종 파일, 소스코드, 메타데이터 등을 담고있는 디렉토리를 의미하게되고
Dockerfile이 위치한 디렉터리가 build context가 됩니다. 위의 예제에서는 빌드 경로를 ./로 설정하여 Dockerfile이 존재하는 경로로 설정을 해주었고 해당 디렉터리에 있는 test.html이 build context에 추가된것입니다. 

그래서 주의할 점이 루트 디렉터리에 Dockerfile이 있게 하면 하위 디렉터리 및 파일들을 모두 build context로 포함하게 될 수 있으므로 과부하를 일으킬 수 있으므로 주의해야합니다. 그리고 이를 방지하거나 필요한 파일들만 build context에 포함시킬 수 있도록 <span style="color:DodgerBlue">.dockerignore</span>파일을 통해 명시된 이름의 파일들을 제외 시킬 수 있습니다. 

![1](https://da2so.github.io/assets/post_img/2022-01-10-Docker_Kubernetes5/5.png){: .mx-auto.d-block width="60%" :}






### 1.3 기타 Dockerfile 명령어

- **ENV**: 환경변수를 지정
	- Example) "ENV test_name /test_val": test_name라는 환경변수에 /test_val값 설정
- **VOLUME**: build된 image로 container를 만들었을때 Host와 공유할 컨테이너 내부의 디렉터리 설정
	- Example) "VOLUME /home/test": dockerfile을 통해 만들어진 container는 /home/test를 만들고 해당 경로에서 생성된 파일들은 host의 /var/lib/docker/volumes/{hash} 에 저장된다. 
- **ARG**: build 명령어를 실행할때 추가로 입력을 받는 arugment로 Dockerfile내에서 사용할 변수의 값을 설정합니다.
	- Example) "ARG my_arg"를 Dockerfile에 작성하고 docker build --build-arg my_arg=/test로 변수의 값을 설정가능
- **USER**: container내의 사용될 사용자의 계정의 이름을 설정하면 해당 사용자의 권한으로 실행
- **STOPSIGNAL**: container가 정지될 때 사용될 시스템 콜
	- Example) "STOPSIGNAL SIGKILL": 정지하면 SIGKILL call  

애플리케이션을 build할때 많은 dependency package와 library를 필요로 하게 됩니다. 




#### ADD, COPY

```
COPY test.html /home/
COPY ['test.html', 'home']
```

ADD와 COPY를 로컬디렉터리에서 읽어 들인 context로부터 이미지에 파일을 복사하는 역할을 하지만 ADD는 더하여 외부 URL 및 tar파일도 추가할 수 있다는 점이 다릅니다.
실제로는 외부 URL이나 tar파일을 추가할경우 명확성이 떨어지기 때문에 COPY를 많이 씁니다. 


#### ENTRYPOINT, CMD

CMD는 아까 설명드렸듯이 docker run명령어에서 맨 뒤에 입력했던 커맨드와 같은 역할을 한다고 말씀드렸습니다. 이와 비슷하게 역할을 ENTRYPOINT하는데여. 다른 점은 커맨들르 인자로 받아 사용할 수 있는 스크립트의 역할을 할 수 있습니다. 

![1](https://da2so.github.io/assets/post_img/2022-01-10-Docker_Kubernetes5/6.png){: .mx-auto.d-block width="60%" :}


위에서 Dockerfile에서는 entrypoint.sh을 container안에 add, 실행을 위한 권한해제를 합니다. 그리고 ENTRYPOINT를 통해 컨테이너 시작시 /bin/bash entrypoint.sh을 실행시키게 되는데 entrypoint.sh파일 내용을 볼 수 있듯이 두개의 arugment를 받아 출력(echo)하고 아파치 웹 서버를 실행시킵니다. 그래서 docker run시에 맨뒤에 arugment인 first와 second를 준것입니다.
