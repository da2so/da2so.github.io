---
layout: post
title: Docker/Kubernetes - (3) docker container 네트워크/로깅/제한
tags: [Docker, Kubernetes]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2022-01-05-Docker_Kubernetes1/logo.png
---

Enviroment: Ubuntu 18.04 
{: .box-note}

## 1. Docker container 네트워크

Docker는 컨테이너에 내부 IP를 순차적으로 할당하며, IP는 컨테이터 재시작마다 변경될 수 있습니다. 컨테이너 내부의 IP는 외부와 연결될 수 있도록 Host에 veth(virtual eth)라는 네트워크 인터페이스를 자동으로 만들게 됩니다. 그렇다면 각 컨테이너 마다 veth를 생성하게 되고 docker0이라는 브리지를 통해 veth인터페이스와 바인딩되어 호스트의 eth0과 연결시켜줍니다.

![1](https://da2so.github.io/assets/post_img/2022-01-07-Docker_Kubernetes3/1.png){: .mx-auto.d-block width="70%" :}

실제로 Host에서 container생성후 다음과 같이 veth와 docker0를 확인가능합니다.


![1](https://da2so.github.io/assets/post_img/2022-01-07-Docker_Kubernetes3/2.png){: .mx-auto.d-block width="70%" :}

### 1.1 Docker network 기능

'docker networ ls'로 Docker에서 기본적으로 쓸수 있는 네트워크를 확인 가능합니다.

![1](https://da2so.github.io/assets/post_img/2022-01-07-Docker_Kubernetes3/3.png){: .mx-auto.d-block width="50%" :}

컨테이너 생성 시 자동으로 bridge를 선택하게 되는 것이고 위에서 docker0이 해당 브리지를 의미합니다.

#### 브리지 네트워크

해당 네트워크는 docker0이 아닌 사용자 정의 브리지를 새로 생성도 가능합니다.

```
docker network create --driver bridge mybridge
```

다음과 같이 브리지 타입의 네트워크가 생성된 것을 확인가능하며 기존의 bridge와 대역폭이 다르다는 것도 알 수 있습니다.

![1](https://da2so.github.io/assets/post_img/2022-01-07-Docker_Kubernetes3/4.png){: .mx-auto.d-block width="50%" :}


새로 만든 네트워크를 사용하여 container생성이 가능합니다.

```
docker run -i -t --net mybridge ubuntu:16.04
```

#### 호스트 네트워크

호스트 네트워크로 설정시 호스트의 네트워크 환경을 그대로 쓸 수 있습니다.

```
docker run -i -t -net host ubuntu:16.04
```

#### 논 네트워크

말 그대로 None으로 네트워크를 쓰지 않겠다는 것이고 이는 외부와 단절됩니다.

```
docker run -i -t -net none ubuntu:16.04
```

## 2. Docker container 로깅

컨테이너의 내부에서 일어나는 일을 아는 것이 중요한데 이를 위해 로깅(loggin)을 하게 됩니다.
저는 fluentd라는 로그를 수집하고 저장(*json형태*)할 수 있는 기능을 제공하는 오픈소스를 이용해서 docker container의 로그를 저장해 보겠습니다.

다음과 같은 로그 수집 시나리오를 실행해보죠.

![1](https://da2so.github.io/assets/post_img/2022-01-07-Docker_Kubernetes3/5.png){: .mx-auto.d-block width="70%" :}

예시를 위한 것이니 위의 3개의 서버의 호스트는 모두 같게 해봅시다. 즉, docker server, fluentd, docker server모두 한 host에서 진행한다는 말입니다.
먼저 mongo 서버의 호스트에서 로그를 저장을 위한 container를 생성해보죠.

```
docker run --name mongoDB -d -p 27017:27017 mongo
```

그리고 fluentd 서버의 호스트에서 다음과 같은 fluent.conf를 작성합니다.

![1](https://da2so.github.io/assets/post_img/2022-01-07-Docker_Kubernetes3/6.png){: .mx-auto.d-block width="70%" :}


해당 내용은 로그 데이터를 mongodb에 전송하고 mongodb의 access라는 collection에 로그를 저장하겟다는 것과 mongodb의 host, port를 지정한것입니다.
또한 'match docker' 부분은 로그의 태그가 docker로 시작하면 이를 mongodb에 저장한다는 뜻입니다. 다음 명령어를 통해 fluent.conf를 fluentd container의 설정파일로 공유시키고
실행시켜봅니다. (주의할 점은 host는 각자 서버에서 ifconfig로부터 나오는 ip를 적어주셔야합니다!)

```
docker run -d --name fluentd -p 24224:24224 -v $(pwd)/fluent.conf:/fluentd/etc/fluent.conf -e FLUENT_CONF=fluent.conf alicek106/fluentd:mongo
```

마지막으로 docker서버에서 로그를 수집할 컨테이너를 생성합니다. --log-driver를 fluentd로, fluentd-address를 fluentd 서버 주소로 설정합니다. 그리고 tag는 위의 fluent.conf에서 docker로 시작하는 tag로부터 로그를 받기때문에 아래와 같은 태그도 mongodb로 저장됩니다. 로그기록위해 nginx로 이미지를 생성하였습니다.

```
docker run -p 80:80 -d --log-driver=fluentd --log-opt fluentd-address=192.168.26.128:24224 --log-opt tag=docker.nginx.webserver nginx
```

이와 같이 되었다면 이제 ubuntu에 127.0.0.1로 접속하면 다음과 같이 nginx 웹서비스가 보이는데 이것은 host의 80포트가 위의 container80포트와 바인딩 된것이기 때문에 가능합니다.

![1](https://da2so.github.io/assets/post_img/2022-01-07-Docker_Kubernetes3/7.png){: .mx-auto.d-block width="90%" :}


그리고 해당 웹서비스를 하는 컨테이너의 로그를 fluentd가 수집하여 mongodb의 access collection에 저장하게 됩니다. 확인을 위해 'docker exec -it mogoDB mongo'로 mongodb 서버의 mongo container에 접속하여 다음과 같은 명령어로 저장된 로깅을 확인가능합니다.


![1](https://da2so.github.io/assets/post_img/2022-01-07-Docker_Kubernetes3/8.png){: .mx-auto.d-block width="70%" :}


**정리**
- docker server host와 nginx container가 80번 포트로 서로 연결됨
- 해당 host에서 nginx containerd안의 nginx 웹서비스 접속(127.0.0.1)
- nginx에 대한 log를 fluentd로 logging 수집
- fluentd에 수집된 log들은 mongo db server로 전달되고 db의 access라는 collection에 log저장

## 3. Docker container 제한

1. container memory제한
```
docker run -d --memory='1g' nginx
# memory를 1 GB로 제한
```

2. container cpu 개수 제한
```
docker run -d --cpuset-cpu=2 nginx
# container가 3번째 cpu(0부터 시작이라 3임)만 사용하도록 함.
```

3. container cpu 사용량 제한
```
docker run -d --cpus=0.5 nginx
# container가 cpu사용량을 50%만 점유할수 있도록 함.
```

