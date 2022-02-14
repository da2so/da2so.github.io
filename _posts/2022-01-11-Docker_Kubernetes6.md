---
layout: post
title: Docker/Kubernetes - (6) Docker daemon
tags: [Docker, Kubernetes]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2022-01-05-Docker_Kubernetes1/logo.png
---

Enviroment: Ubuntu 18.04 
{: .box-note}
## 1. Docker daemon
docker 그 자체에 대해서 알아보는 시간입니다. Docker의 구조는 크게 2가지로 나뉩니다. 

- **<span style="color:Crimson">Docker server</span>**
	- */usr/bin/dockerd* 파일로 실행
	- container 생성 및 실행과 image관리하는 주체
	- 외부에서 API 입력을 받아 docker engine의 기능을 수행
	- docker process가 실행되어 서버로서 API 입력을 받을 준비가 된 상태를 <span style="color:DodgerBlue">docker daemon</span>
- **<span style="color:Crimson">Docker client</span>**
	- */usr/bin/docker* 에서 실행
	- 외부에서 API를 사용할 수 있도록 CLI제공
		- 외부에서 API요청을 해당 clinet가 사용되며 client는 docker daemon에게 API 전달
	- Client는 */var/run/docker.sock*에 위치한 unix socket을 통해 daemon API호출


![docker_client_daemon](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/1.png){: .mx-auto.d-block width="90%" :}

## 2. Docker daemon 실행 및 설정

docker daemon은 다음 명령어로 시작, 정지가 가능합니다. (start는 둘 중 아무 명령어를 사용해도 됩니다.)
```bash
#start
service docker start
dockerd
#stop
service docker stop
```

<span style="color:DodgerBlue">dockerd</span>명령어를 통해서도 daemon을 실행가능합니다. 그리고 daemon에 적용할 수 있는 옵션에 대해 알아봅시다. 

### 2.1 Docker daemon 제어 **-H**

아무런 옵션 없이 daemon을 실행시키면 default로 docker client인 /usr/bin/docker을 위한 unix socket인 /var/run/dokcer.sock을 사용하게 됩니다. 이를 **-H**옵션을 통해 나타내면 다음과 같고 두 명령어는 같은 명령어입니다.

```bash
dockerd
dockerd -H unix:///var/run/docker.sock
```

즉, **-H**옵션을 통해서 dameon의 API를 사용할 방법을 추가할수 있다는 것이고 예를 들어 **-H**뒤에 IP주소와 port번호를 입력하면 원격 API인 Docker remote API로 docker를 제어가능합니다.
(Remote API는 RESTful API형식을 띄고 있으므로 HTTP 요청으로 docker를 제어가능) 다음과 같이 docker daemon을 실행하면 host에 존재하는 모든 네트워크 인터페이스의 IP주소와 2375번 포트를 바인딩해 입력을 받습니다.

```bash
dockerd -H tcp://0.0.0.0:2375
```

![docker_ps](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/2.png){: .mx-auto.d-block width="90%" :}

하지만 이렇게 할경우 Remote API만을 위한 바인딩 주소를 입력했으므로 unix scoket은 비활성화 되고 docker client를 사용못하게 됩니다. 즉, 위와 같이 **docker ps**와 같이 docker로 시작하는 명령어 사용 이 불가합니다. 그러므로 이를 해결하기 위해 Remote API를 위한 바인딩 주소와 unix scoket을 같이 설정해 줍니다.

```bash
dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
```

위와 같이 하면 docker client가 docker daemon에게 명령 수행 요청뿐만 아니라 Remote API또한 사용 가능합니다. 또 하나의 terminal을 켜서 curl을 통해 Http요청을 보내 Remote API가 작동가능한 지 확인해봅니다. Host ip(192.168.26.129), port(2375)를 가지는 docker daemon으로 http 요청을 보내는 것이고 192.168.26.129:2375/version은 <span style="color:DodgerBlue">docker version</span> 명령어와 같습니다. 

![dockerd](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/3.png){: .mx-auto.d-block width="80%" :}


그리고 다음과 같이 docker client에 -H옵션을 설정해 제어할 원격 docker daemon을 설정가능합니다.

![docker_client_option](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/4.png){: .mx-auto.d-block width="60%" :}

### 2.2 Docker daemon 보안 적용 **--tlsverify**

Docker는 기본적으로 보안 연결 설정이 안되어있기 때문에 docker client, remote API 사용할때 위험성이 있다는 것을 의미합니다. 특히나 보안이 없다면 ip, port만 안다면 remote API를 통해 docker를 제어가능해져 버립니다. 

그래서 docker daemon에 TLS 보안을 적용하고 docker client가 인증하지 않으면 docker를 제어할 수 없도록 설정해봅시다. 

![TLS](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/5.png){: .mx-auto.d-block width="60%" :}


#### 서버 측 파일 생성

-  인증서에 사용할 key 생성 **<span style="color:Crimson">(ca-key.pem)</span>**

RSA 4096 키 생성 및 개인키를 AES256 으로 암호화하여 ca-key.pem 파일을 만듭니다. 

![create_key](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/6.png){: .mx-auto.d-block width="90%" :}

- Public key를 생성 **<span style="color:Crimson">(ca.pem)</span>**

입력하는 모든 항목은 공백으로 둬도 상관없으므로 공백으로 둔다.

![create_public_key](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/7.png){: .mx-auto.d-block width="90%" :}


- 서버에서 사용할 key 생성 **<span style="color:Crimson">(server-key.pem)</span>**


![create_server_key](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/8.png){: .mx-auto.d-block width="90%" :}


- 서버에서 사용될 인증서를 위한 인증 요청서 파일 생성 **<span style="color:Crimson">(server.csr)</span>**

192.168.29.129은 docker host의 IP주소 또는 domain이름이고 이는 외부에서 접근가능해야합니다.

![create_cert_file](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/9.png){: .mx-auto.d-block width="90%" :}


- 서버 측의 인증서 파일을 생성합니다. **<span style="color:Crimson">(server-cert.pem)</span>**

접속에 사용될 IP주소를 extfile.cnf로 저장하고 192.168.29.129으로 연결이 사용되도록 인증서 파일을 생성한다.

![create_cert_server_file](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/10.png){: .mx-auto.d-block width="90%" :}


#### 클라이언트 측 파일 생성


- 클라이언트 측의 key 파일 **<span style="color:Crimson">(key.pem)</span>**과 인증 요청파일을 생성 **<span style="color:Crimson">(client.csr)</span>** , extfile.cnt파일에 extendedKeyUsage항목 추가

![create_client_key](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/11.png){: .mx-auto.d-block width="90%" :}


- 클라이언트 측의 인증서 생성 **<span style="color:Crimson">(cert.pem)</span>**

![create_client_cert](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/12.png){: .mx-auto.d-block width="90%" :}


- 서버, 클라이언트 측에서 사용할 인증서들에 대한 쓰기 권한 삭제

![remove_authorization](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/13.png){: .mx-auto.d-block width="90%" :}


#### TLS 보안 적용

TLS 보안 적용을 위해 *--tlsverify* 옵션과 보안을 위해 필요한 옵션들을 더하여 docker daemon을 실행한다. (밑의 맨위 그림)

```bash
dockerd --tlsverify --tlscacert=ca.pem --tlscert=server-cert.pem --tlskey=server-key.pem -H=0.0.0.0:2376 -H unix:///var/run/docker.sock
```

그리고 client측에서 TLS 연결 설정을 하지 않고 Remote API로 접근했을 때와(Middle 그림) TLS 연결 설정을 하고 접근했을때의 원격 제어시(밑의 맨아래 그림)의 결과 차이를 볼 수 있다.

```bash
# No TLS
docker -H 192.168.26.129:2376 version
# TLS
docker -H 192.168.26.129:2376 --tlscacert=ca.pem --tlscert=cert.pem --tlskey=key.pem --tlsverify version
```

docker의 remote API를 사용하는 포트는 보안이 적용되어 있지 않으면 2375, 되어 있으면 2376를 사용하는 것이 도커 커뮤니티의 관례
{: .box-note}

![TLS_comparsion](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/14.png){: .mx-auto.d-block width="90%" :}


### 2.3 Docker daemon 모니터링

다음과 같은 이유로 도커 데몬 자체에 대한 모니터링이 필요합니다.

- 많은 수의 docker server 관리
- container 어플리케이션 개발 중 오류 디버깅
- 도크를 Paas로써 제공하기 위해 실시간으로 상태 체크


#### Docker daemon 디버그 모드

다음과 Docker daemon을 디버그 옵션으로 실행하면 Remote API뿐만 아니라 로컬 docker client에서 오가는 명령어도 로그로 출력하게 됩니다. 하지만 원치 않는 정보도 출력되고 해당 daemon을 foreground 상태로 실행해야 한다는 단점이 있다.

```bash
dockerd -D
```

#### events, stats, system df 명령어

**<span style="color:Crimson">events</span>**는 docker daemon에서 일어나는 일을 실시간 stream log로 보여줍니다.

```bash
docker events
```

위의 명령어를 실행시키고 **docker pull ubuntu:14.04**을 해보면 다음과 같이 docker events 명령어 밑으로 logging이 됩니다. events는 attach, commit, copy, create와 같은 container관련 명령어, delete, import, pull, push등의 image 관련 명령어, 볼륨, 네트워크, 플러그인 명령어의 수행 결과만을 출력합니다. 

![docker_pull](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/15.png){: .mx-auto.d-block width="80%" :}

**<span style="color:Crimson">stats</span>**은 실행 중인 모든 container의 자원 사용량을 stream으로 출력한다.

```
docker stats
```

![docker_stats](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/16.png){: .mx-auto.d-block width="80%" :}

**<span style="color:Crimson">system df</span>**은 docker에서 사용하고 있는 이미지, 컨테이너, 로컬 볼륨의 총 개수 및 사용중인 개수, 크기 등을 출력합니다.

```bash
docker system df
```

![docker_system_df](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/17.png){: .mx-auto.d-block width="70%" :}


#### CAdvisor

구글이 만든 컨테이너 모니터링 도구로 실기간 자원 사용량 및 도커 모니터링 정보 등을 시각화해서 보여줍니다. 다음과 같이 docker hub에서 docker image로 사용 가능하며 container agent형태로 도커 모니터링에 필요한 자료를 수집합니다. 


```bash
docker run \
--volume=/:/rootfs:ro \
--volume=/var/run:/var/run/:ro \
--volume=/sys:/sys:ro \
--volume=/var/lib/docker/:/var/lib/docker:ro \
--volume=/dev/disk/:/dev/disk:ro \
--publish=8080:8080 \
--detach=true \
--name=cadvisor \
google/cadvisor:latest
```

![cadvisor](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/18.png){: .mx-auto.d-block width="80%" :}


Host에서 *127.0.0.1:8080*에 접속하면 CAdvisor 대시보드에 접근할 수 있고 해당 웹에서 말한 다양한 정보를 볼 수 있다. [Subcontainers]항목의 [/docker]를 클릭하면 docker daemon의 정보, 컨테이너의 목록을 보여주는 페이지로 이동한다. CAdvisor의 container는 위의 옵션에서 많은 *--volume*옵션을 통해 docker의 정보를 담고 있는 파일들을 마운트했기때문에 CAdvisor에서 다양한 모니터링이 가능함을 알아둡시다. 예를 들어 /var/lib/docker에는 도커 컨테이너, 이미지 등이 파일로 존재합니다. 


하지만 CAdvisor는 단일 docker host만을 모니터링 가능하므로 여러개의 호스트로 docker를 사용하고 있다면 **Kubernetes**나 **swarm mode** 등과 같은 오케스트레이션 툴을 설치한 뒤에 프로메테우스(Prometheus), InfluxDB등을 이용해 여러 host의 데이터를 수집해야한다. (이후 글에서 더 자세히!!)


### 2.4 Remote API라이브러리를 이용한 docker 사용

docker daemon을 제어하기 위해 -H옵션을 통해 진행했었는데 container application이 수행햐 할 작업이 많거나 초기화등 복잡한 과정이 포함되어 있을경우 docker를 제어하는 라이브러리를 사용한다. 즉, Remote API를 wrapping해서 사용하기 쉽게 만들어 놓은 라이브러리를 쓰면 도커 사용이 편해진다. 


#### 파이썬 라이브러리

파이썬 라이브러리를 사용하기 위한 환경은 python 3.6입니다. docker library를 설치하려면 pip3가 필요하기 때문에 pip3가 설치되어 있지 않으시면 다음과 같이 설치하고 docker library를 설치합시다. 

```bash
# if you don't installed pip3 before
apt install python3-pip -y
# install docker library 
pip3 install docker
```

라이브러리가 정상적으로 설치되어있는 지 확인하기위해 unix socket에 연결해 docker engine의 정보를 출력해봅니다.

![python3](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/19.png){: .mx-auto.d-block width="90%" :}


좀더 심화 과정으로 다음과 같은 시나리오를 진행해보죠.

1. Remote API를 사용할 수 있게 0.0.0.0:2376을 바인딩 주소로 해놓고 TLS보안이 적용되도록 docker daemon실행
2. 파이썬 라이브러리를 통해 해당 docker daemon에 연결 후 ubuntu:16.04 image로 container실행

위에서 TLS적용을 위해 진행한 key들이 있는 directory에서 tls_docker_run.py이라는 python 파일을 만들어 실행해보았습니다. 

```bash
#ls_docker_run.py

import docker
import requests.packages.urllib3 as urllib3
urllib3.disable_warnings() # disable to print warning

tls_config = docker.tls.TLSConfig(client_cert=('./cert.pem', './key.pem')) 

client= docker.DockerClient(base_url='tcp://192.168.26.129:2376', tls=tls_config) 
container = client.containers.run('ubuntu:16.04', name='python_ubuntu', detach=True)

print(f'Created container is : {container.name}, {container.id}')
```
![python_with_docker](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/20.png){: .mx-auto.d-block width="90%" :}

