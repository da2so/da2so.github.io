---
layout: post
title: Docker/Kubernetes - (5) Docker daemon
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
	- 외부에서 API 입력을 받아 도커 엔진의 기능을 수행
	- docker process가 실행되어 서버로서 API 입력을 받을 준비가 된 상태를 <span style="color:DodgerBlue">docker daemon</span>
- **<span style="color:Crimson">Docker client</span>**
	- */usr/bin/docker* 에서 실행
	- 외부에서 API를 사용할 수 있도록 CLI제공
		- 외부에서 API요청을 해당 clinet가 사용되며 client는 docker daemon에게 API 전달
	- Client는 */var/run/docker.sock*에 위치한 unix socket을 통해 daemon API호출


![1](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/1.png){: .mx-auto.d-block width="90%" :}

## 2. Docker daemon 실행 및 설정

docker daemon은 다음 명령어로 시작, 정지가 가능합니다. 
```
service docker start
service docker stop
```

<span style="color:DodgerBlue">dockerd</span>명령어를 통해서도 daemon을 실행가능합니다. 그리고 daemon에 적용할 수 있는 옵션에 대해 알아봅시다. 

### 2.1 Docker daemon 제어 **-H**

아무런 옵션 없이 daemon을 실행시키면 default로 docker client인 /usr/bin/docker을 위한 unix socket인 /var/run/dokcer.sock을 사용하게 됩니다. 이를 **-H**옵션을 통해 나타내면 다음과 같고 두 명령어는 같은 명령어입니다.

```
dockerd
dockerd -H unix:///var/run/docker.sock
```

즉, **-H**옵션을 통해서 dameon의 API를 사용할 방법을 추가할수 있다는 것이고 예를 들어 **-H**뒤에 IP주소와 port번호를 입력하면 원격 API인 Docker remote API로 docker를 제어가능합니다.
(Remote API는 RESTful API형식을 띄고 있으므로 HTTP 요청으로 docker를 제어가능) 다음과 같이 docker daemon을 실행하면 host에 존재하는 모든 네트워크 인터페이스의 IP주소와 2375버 포트를 바인딩해 입력을 받습니다.

```
dockerd -H tcp://0.0.0.0:2375
```

![1](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/2.png){: .mx-auto.d-block width="90%" :}

하지만 이렇게 할경우 Remote API만을 위한 바인딩 주소를 입력했으므로 unix scoket은 비활성화 되고 docker client를 사용못하게 됩니다. 즉, 위와 같이 **docker ps**와 같이 docker로 시작하는 명령어 사용 이 불가합니다. 그러므로 이를 해결하기 위해 Remote API를 위한 바인딩 주소와 unix scoket을 같이 설정해 줍니다.

```
dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
```

위와 같이 하면 docker client가 docker daemon에게 명령 수행 요청뿐만 아니라 Remote API또한 사용 가능하다. 또 하나의 terminal을 켜서 curl을 통해 Http요청을 보내 Remote API가 작동가능한 지 확인해봅니다. Host ip(192.168.26.129), port(2375)를 가지는 docker daemon으로 http 요청을 보내는 것이고 192.168.26.129:2375/version은 <span style="color:DodgerBlue">docker version</span> 명령어와 같습니다. 

![1](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/3.png){: .mx-auto.d-block width="80%" :}


그리고 다음과 같이 docker client에 -H옵션을 설정해 제어할 원격 docker daemon을 설정가능합니다.

![1](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/4.png){: .mx-auto.d-block width="60%" :}

### 2.2 Docker daemon 보안 적용 **--tlsverify**

Docker는 기본적으로 보안 연결 설정이 안되어있기 때문에 docker client, remote API 사용할때 위험성이 있다는 것을 의미합니다. 특히나 보안이 없다면 ip, port만 안다면 remote API를 통해 docker를 제어가능해져 버립니다. 

그래서 docker daemon에 TLS 보안을 적용하고 docker client가 인증하지 않으면 docker를 제어할 수 없도록 설정해봅시다. 

![1](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/5.png){: .mx-auto.d-block width="60%" :}


#### 서버 측 파일 생성

1.  인증서에 사용할 key 생성 **<span style="color:Crimson">(ca-key.pem)</span>**

RSA 4096 키 생성 및 개인키를 AES256 으로 암호화하여 ca-key.pem 파일을 만듭니다. 

![1](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/6.png){: .mx-auto.d-block width="90%" :}

2. Public key를 생성 **<span style="color:Crimson">(ca.pem)</span>**

입력하는 모든 항목은 공백으로 둬도 상관없으므로 공백으로 둔다.

![1](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/7.png){: .mx-auto.d-block width="90%" :}


3. 서버에서 사용할 key 생성 **<span style="color:Crimson">(server-key.pem)</span>**


![1](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/8.png){: .mx-auto.d-block width="90%" :}


4. 서버에서 사용될 인증서를 위한 인증 요청서 파일 생성 **<span style="color:Crimson">(server.csr)</span>**

192.168.29.129은 docker host의 IP주소 또는 domain이름이고 이는 외부에서 접근가능해야합니다.

![1](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/9.png){: .mx-auto.d-block width="90%" :}


5. 서버 측의 인증서 파일을 생성합니다. **<span style="color:Crimson">(server-cert.pem)</span>**

접속에 사용될 IP주소를 extfile.cnf로 저장하고 192.168.29.129으로 연결이 사용되도록 인증서 파일을 생성한다.

![1](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/10.png){: .mx-auto.d-block width="90%" :}


#### 클라이언트 측 파일 생성


1. 클라이언트 측의 key 파일 **<span style="color:Crimson">(key.pem)</span>**과 인증 요청파일을 생성 **<span style="color:Crimson">(client.csr)</span>** , extfile.cnt파일에 extendedKeyUsage항목 추가

![1](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/11.png){: .mx-auto.d-block width="90%" :}


2. 클라이언트 측의 인증서 생성 **<span style="color:Crimson">(cert.pem)</span>**

![1](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/12.png){: .mx-auto.d-block width="90%" :}


3. 서버, 클라이언트 측에서 사용할 인증서들에 대한 쓰기 권한 삭제

![1](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/13.png){: .mx-auto.d-block width="90%" :}


#### TLS 보안 적용

TLS 보안 적용을 위해 *--tlsverify* 옵션과 보안을 위해 필요한 옵션들을 더하여 docker daemon을 실행한다.(Top 그림)

```
dockerd --tlsverify --tlscacert=ca.pem --tlscert=server-cert.pem --tlskey=server-key.pem -H=0.0.0.0:2376 -H unix:///var/run/docker.sock
```

그리고 client측에서 TLS 연결 설정을 하지 않고 Remote API로 접근했을 때와(Middle 그림) TLS 연결 설정을 하고 접근했을때의 원격 제어시(Bottom 그림)의 결과 차이를 볼 수 있다

```
#No TLS
docker -H 192.168.26.129:2376 version
#TLS
docker -H 192.168.26.129:2376 --tlscacert=ca.pem --tlscert=cert.pem --tlskey=key.pem --tlsverify version
```

docker의 remote API를 사용하는 포트는 보안이 적용되어 있지 않으면 2375, 되어 있으면 2376를 사용하는 것이 도커 커뮤니티의 관례
{: .box-note}

![1](https://da2so.github.io/assets/post_img/2022-01-11-Docker_Kubernetes6/14.png){: .mx-auto.d-block width="90%" :}




