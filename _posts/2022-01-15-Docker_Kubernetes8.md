---
layout: post
title: Docker/Kubernetes - (8) Docker Compose
tags: [Docker, Kubernetes]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2022-01-05-Docker_Kubernetes1/logo.png
---

Enviroment: Ubuntu 18.04 
{: .box-note}
## 1. Docker Compose
다른 image로 이루어진 여러개의 container가 하나의 어플리케이션으로 동작시키려면 run명령어를 여러번 사용해야합니다. 하지만 이는 편리성 및 관리의 불편함이 있기 때문에 <span style="color:Crimson">docker compose</span>를 사용하게 됩니다. docker compose는 container를 이용한 서비스 개발과 CI를 위해 여러 개의 container을 하나의 project로 다룰 수있는 작업환경을 제공합니다. run명령어의 옵션을 모두 사용가능하면 swarm cluster처럼 서비스의 container수를 유동적으로 조절가능하며 container의 서비스 디스커버리도 자동으로 이루어집니다. 

### 1.1 Docker Compose 설치 

```
 sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

위의 명령어로 docker-compose 1.24.1버전을 설치하고 다음과 같이 docker-compose를 사용할수 있게 권한을 해제하고 compose 버전을 확인한다.

```
# 권한 해제
sudo chmod +x /usr/local/bin/docker-compose
# 버전 확인
docker-compose -x
```


![docker_compose_check](https://da2so.github.io/assets/post_img/2022-01-15-Docker_Kubernetes8/1.png){: .mx-auto.d-block width="80%" :}

### 1.2 Docker compose 사용

기본적으로 docker compose는 container의 설정이 정의된 **YAML**파일을 읽어 docker engien을 통해 container를 생성합니다. YAML파일 작성과 docker compose 실행 예시를 보여드리기 위해 [이전 글](https://da2so.github.io/2022-01-07-Docker_Kubernetes3/)에서 네트워크 logging을 docker compose로 만들어보도록 하겠습니다. 다음과 같이 **docker-compse.yml**이름으로 파일을 만들어봅니다.

```
#docker-compose.yml
version: '3.0'
services:
  web:
    image: nginx
    ports:
      - "80:80"
    links:
      - fluentd
    logging:
      driver: "fluentd"
      options:
        fluentd-address: localhost:24224
        tag: docker.nginx.webserver
    depends_on:
      - fluentd
  fluentd:
    image: alicek106/fluentd:mongo
    ports:
      - "24224:24224"
    volumes:
      - ./fluent.conf:/fluentd/etc/fluent.conf
    environment:
      - FLUENT_CONF=./fluent.conf
    depends_on:
      - mongo
  mongo:
    image: mongo
    ports:
      - "27017:27017"

```

위의 yml파일의 내용설명은 다음과 같습니다.

- version: YAML 파일 포맷 버전을 뜻하며 docker engine과의 의존성이 있으므로 최신버전 사용을 권함
- services:	생성될 container들을 묶어놓은 단위
- web, fluentd, mongo: 생성될 서비스의 이름
	- image: docker image이름
	- ports: 서비스의 container을 개방할 포트를 설정
	- links: 다른 서비스에 서비스명만으로 접근 가능
		- 다른 방법으로[SERVICE:ALIAS]의 형식을 사용하면 서비스에 별칭(ALIAS)으로도 접근가능합니다.
	- depends_on: container간의 의존관계를 뜻하며 항목에 명시된 container가 먼저 생성됨
	- environment: docker run의 -e 옵션과 같은 것으로 환경변수 설정
	- command: container실행시 수행할 명령어
	- build: Dockerfile에서 image를 build해 서비스의 container를 생성
		- 예시로 *build: ./test*는 ./test 디렉터리에 저장된 Dockerfile로 image를 build해 container를 생성합니다.
	- context: build와 같이 사용되는 인자값으로 context을 지정
	- dockerfile: build와 같이 사용되는 인자값으로 도커파일을 지정
	- volume: 호스트와 공유할 디렉터리 지정
	- driver: 사용할 네트워크 드라이버 설정
		- 하위항목으로 driver_ops를 통해 드라이버에 필요한 옵션 설정 가능
		- docker compose는 생성된 container를 위해 기본적으로 bridge타입의 네트워크를 생성, 해당 이름은 [프로젝트 이름]\_default 입니다.
	- tty: docker 특성상 커맨드가 끝나면 container가 종료되므로 해당 옵션을 통해 종료를 방지함
위에서 설명한 옵션이 전부는 아니지만 대부분의 옵션은 설명드렸습니다. 이제 해당 파일을 통해 docker compose를 사용해봅시다.

```
docker-compose up -d
```

![docker_compose_up](https://da2so.github.io/assets/post_img/2022-01-15-Docker_Kubernetes8/2.png){: .mx-auto.d-block width="100%" :}

기본적으로 dokcer-compose up -d는 명령어를 실행한 디렉터리의 docker-compse.yml파일이름을 가진 yml에 대해 docker compose를 실행하게됩니다. 그리고 위와 같이 web, fluentd, mongo서비스를 정의하였고 각 서비스별로 container하나씩 생성되었으며 이름은 **[프로젝트 이름]_[서비스 이름]_[서비스 내의 container번호]**로 정해집니다. 프로젝트 이름은 지정해주지 않으면 위의 그림과 같이 현재 디렉토리 이름으로 지정하게 됩니다. docker-compose로 로깅이 잘되는지 확인도 가능합니다.

![docker_compose_active_check](https://da2so.github.io/assets/post_img/2022-01-15-Docker_Kubernetes8/7.png){: .mx-auto.d-block width="90%" :}


swarm moded에서의 서비스와 마찬가지로, 하나의 서비스에는 여러 개의 container가 존재할 수 있으므로 차례대로 증가하는 container의 번호를 붙여 서비스 내의 container를 구별합니다. 생성된 프로젝트는 <span style="color:DodgerBlue">docker-compose down</span>으로 삭제할 수 있으며 이 명령어는 서비스의 container모두 정지시킨 뒤 삭제합니다. 

![docker_compose_down](https://da2so.github.io/assets/post_img/2022-01-15-Docker_Kubernetes8/3.png){: .mx-auto.d-block width="60%" :}

그리고 ubuntu이미지로 docker-compose.yml을 만들고 **-p**옵션을 사용해서 프로젝트 이름을 설정하여 프로젝트를 생성합니다. 그럼 projectname이 옵션을 준값대로 설정된 것을 확인가능합니다.

```
docker-compose -p myubuntu up -d
```

![docker_compose_up_with_p_option](https://da2so.github.io/assets/post_img/2022-01-15-Docker_Kubernetes8/4.png){: .mx-auto.d-block width="70%" :}

### 1.3 docker swarm mode와 연동

swarm mode와 함께 사용되는 개념으로 stack을 사용하는데요. 기존에는 docker-compose로 제어했다면 swarm mode에서는 <span style="color:DodgerBlue">docker stack</span>명령어로 제어합니다. 이는 swarm mode cluster의 매니저에 의해 생성되는 명령어라고 생각하면 됩니다. 먼저 다음 명령어로 docker swarm mode로 변경해줍니다.

```
docker swarm init --advertise-addr='host의 ip 주소'
```

그리고 다음과 같이 docker-compose.yml을 작성해주고 <span style="color:DodgerBlue">docker stack deploy</span> 명령어로 프로젝트를 생성합니다.


```
docker stack deploy -c docker-compose.yml stack_test
```

![docker_stack](https://da2so.github.io/assets/post_img/2022-01-15-Docker_Kubernetes8/5.png){: .mx-auto.d-block width="90%" :}


-c옵션은 YAML파일을 지정해주는 것이고 마지막은 stack의 이름입니다. 위에서 보이듯이 swarm mode에서도 잘 생성된것을 확인가능합니다. 그리고 종료는 다음과 같습니다.

```
docker stack rm stack_test
```

![docker_stack_stop](https://da2so.github.io/assets/post_img/2022-01-15-Docker_Kubernetes8/6.png){: .mx-auto.d-block width="60%" :}







