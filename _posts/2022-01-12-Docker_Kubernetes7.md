---
layout: post
title: Docker/Kubernetes - (7) Docker Swarm
tags: [Docker, Kubernetes]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2022-01-05-Docker_Kubernetes1/logo.png
---

Enviroment: Ubuntu 18.04 
{: .box-note}
## 1. Docker Swarm
지금까지 저희는 하나의 host에서만 docker engine을 구동하였습니다. 하지만 하나의 Host는 CPU, memory측면에서 자원의 한계를 가지므로 여러대의 서버를 cluster로 만들어 자원을 병렬로 확장하여 사용해야 합니다. 이때 여러대의 서버를 사용할때나 또는 서버를 추가하고 싶을때 container 할당에 대한 스케줄러, 로드밸런스 문제 등 이런 문제를 해결하는 것이 바로 <span style="color:Crimson">docker swarm</span>과 <span style="color:Crimson">swarm mode</span>입니다.

## 2. Docker Swarm mode

Swarm mode는 여러 대의 docker server를 하나의 클러스터로 만들어 container를 생성하는 여러 기능을 제공하고 **특징점**은 다음과 같습니다. (비슷하게 swarm classic이라는 것도 있지만 swarm mode가 대규모 클러스터에서 서비스하기 좋다고 합니다.)

- 마이크로서비스 아키텍처의 container를 다루기 위한 클러스터링 기능에 초점을 맞춤
	- 마이크로서비스 아키텍쳐: 대규모 소프트웨어 서비스를 마이크로 단위의 모듈로 분리하여 loosely-coupled한 구조로 만들고 API를 통해 서로 통신
- 같은 container를 동시에 여러 개 생성 및 유동적으로 컨테이너 수 조절
- container의 연결을 분산하는 로드밸런싱 기능
- 분산 코디네이터(Distributed Coordinator), 매니저(manager), 에이전트(agenet)가 모두 docker engine에 내장됨
	- **분산 코디네이터**: 여러 개의 docker server를 하나의 클러스터 구성하기 위해 각종 정보를 저장하고 동기화
	- **매니저**: 클러스터 내의 서버를 관리하고 제어
	- **에이전트**: 각 서버를 제어


### 2.1 Docker swarm mode의 구조 

아래와 같이 swarm mode는 워커(worker) 노드와 매니저(manager) 노드로 이루어져 있습니다.

- 워커 노드: container가 생성되고 관리되는 docker server
- 매니저 노드: 워커 노드를 관리하기 위한 서버
	-  하지만 container 생성 가능하므로 워커 노드의 역할을 포함함

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/1.png){: .mx-auto.d-block width="70%" :}


매니저 노드는 1개 이상 존재해야 하지만 워커 노드는 없을 수도 있습니다. 매니저 노드가 워커 노드의 역할을 대신할수 있기 때문입니다. 이번 글에서는 매니저 노드를 1개 사용하지만 실제 클러스터링 환경에서는
매니저 노드가 적당히! 많을 수록 매니저의 부하를 줄일 수 있고 특정 매니저가 다운되더라도 스웜 클러스터를 유지가능하기 때문입니다. 추가로 매니저 노드 사이의 네트워크 파티셔닝같은 현상이 일어날경우 매니저 노드 개수를 홀수 개로 구성해야 놔야만 과반수 이상이 유지되는 쿼럼(quorum) 매니저에서 운영을 계속할 수 있습니다.


### 2.2 Docker swarm cluster 구축

저는 swarm cluster예제를 보여드릴려고 VMware를 이용한 ubuntu host3개를 띄웠습니다. 각 host에 대한 ip, host name 및 노드 역할은 다음과 같습니다.

- 매니저 노드: 192.168.26.129 (manager)
- 워커 노드1: 192.168.26.130 (worker1)
- 워커 노드2: 192.168.26.131 (worker2)

먼저 매니저 노드인 192.168.26.129에서 다음 명령어를 입력해 swarm cluster을 시작한다.

```
docker swarm init --advertise-addr 192.168.26.129
```
![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/2.png){: .mx-auto.d-block width="100%" :}


**--advertise-addr**은 다른 docker server가 매니저 노드에 접근하기 위한 해당 host의 IP주소를 입력합니다. 출력 결과 중 <span style="color:DodgerBlue">docker swarm join</span>명령어는 새로운 워커 노드를 swarm cluster에 추가할때 사용되고 **--token**옵션에 사용된 토큰 값은 새로운 노드를 해당 swarm cluster에 추가하기 위한 private key입니다.


이제 위의 <span style="color:DodgerBlue">docker swarm join</span>을 통해 worker1, 2 host에서 swarm cluster에 join해봅니다. 그리고 manager node에서 <span style="color:DodgerBlue">docker node ls</span>을 통해 cluster에 work들이 잘 포함되어있는 지 확인합니다. 

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/3.png){: .mx-auto.d-block width="85%" :}


ID옆에 별표(\*)는 현재 노드인 매니저를 말합니다. 그리고 새로운 매니저 노드를 추가하려면 매니저 노드를 위한 token은 다음과 같은 명령어로 확인가능합니다.

```
docker swarm join-token manager
```
![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/4.png){: .mx-auto.d-block width="100%" :}

해당 token은 외부에 노출되면 누구든지 해당 swarm cluster에 추가될 수 있기때문에 주기적으로 token을 변경해주는게 좋습니다. token 갱신을 위해서는 **--rotate**옵션을 넣어 아래와 같이 입력하면 새로운 token을 발급받을 수 있습니다.

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/5.png){: .mx-auto.d-block width="100%" :}


워커 노드를 삭제하려고 할때는 먼저 워커 노드에서 <span style="color:DodgerBlue">docker swarm leave</span>를 해주고 매니저 노드에서 down된 워커를  다음과 같은 명령어로 제거해줍니다. 

```
#in worker1 node
docker swarm leave

#in manager node
docker node rm worker1
```
![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/6.png){: .mx-auto.d-block width="80%" :}

그리고 워커 노드를 매니저 노드로 반대로 매니저 노드를 워커노드로 변경하는 명령어는 각각 <span style="color:DodgerBlue">docker node promote</span>, <span style="color:DodgerBlue">docker node demote</span>입니다. 하지만 매니저 노드가 1개일때는 demote명령어를 사용할수 없으며 매니저 리더 노드에 demote를 할경우 다른 매니저 노드 중 새로운 리더를 선출합니다.

```
docker node promote worker2
docker node demote worker2
```

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/7.png){: .mx-auto.d-block width="80%" :}

그리고 매니저 노드를 삭제하려면 다음 명령어로 진행합니다. 매니저 노드가 한개인 경우에 매니저 노드를 삭제할경우 스웜 클러스터는 더 이상 사용하지 못하는 상태가 되므로 삭제시에는 신중히 해야합니다.


```
docker swarm leave --force
```

## 3. Swarm mode 서비스

### 3.1 Swarm mode 서비스 개념

이전 글에서 도커 명령어의 제어 단위는 container엿는데여(ex: docker run) swarm mode에서는 제어하는 단위가 <span style="color:Crimson">서비스(service)</span>로 바뀌게 됩니다. 
서비스는 같은 image에서 생성된 container집합이며, 서비스를 제어하면 해당 서비스 내의 container에 같은 명령어가 수행됩니다. 서비스 내의 container는 1개 이상 존재할 수 있으며 container는 워커, 매니저 노드에 할당됩니다. 이러한 container들을 <span style="color:Crimson">Task</span>라고 부르게 됩니다.

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/8.png){: .mx-auto.d-block width="100%" :}

위와 같이 특정 docker image로 서비스를 생성하고 컨테이너 수를 3개로 설정했다면 swarm scheduler는 container를 적당한 노드에 선정해 할당하게 됩니다. 이와 같이 함께 생성된 container를 replica라고 합니다. replica수는 서비스 생성 시 정해 줄 수 있고 정해진 수만큼의 container가 swarm cluster에 존재해야만 합니다. 그 예로 위의 오른쪽 그림과 같이 하나의 노드가 다운되면 swarm manager는 3개의 replica수를 맞춰야 하므로 새로운 container를 다른 노드에 새롭게 생성하게 됩니다. 

### 3.2 서비스 생성

서비스를 제어하는 도커 명령어는 오직 매니저 노드에서만 가능함을 인지합니다.


#### nginx 웹 서버 서비스

서비스는 <span style="color:DodgerBlue">docker service create</span> 명령어를 통해 생성하며 replica수 2개로 다음과 같이 옵션으로 추가해줍니다. 이번 글에서 만들 서비스는 Nginx 웹서버 image를 이용해 서비스를 외부에 노출하는 것입니다. 서비스가 올바르게 생성 된지 확인하려면 <span style="color:DodgerBlue">docker service ls</span>를 사용하고 더 자세한 정보를 확인하려면 <span style="color:DodgerBlue">docker service ps [서비스이름]</span>을 입력합니다.

```
docker service create --name myweb --replicas 2 -p 80:80 nginx

docker service ls
docker service ps myweb
```

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/9.png){: .mx-auto.d-block width="100%" :}


컨테이너가 정상적으로 생성되엇다면 swarm cluster내의 노드(worker1, worker2) 중 하나를 선택해 80번 포트로 접근해 웹 서비스가 구동되는 지확인해봅니다. worker1 host에서 firefox를 켜시고 ip주소(127.0.0.1:80)로 접속해보면 웹서비스가 잘 되는 것을 알 수 있습니다. 그렇다고 해서 꼭 두 노드에서만 웹서비스에 접근가능한 것은 아닙니다. swarm cluster자체에 80:80포트를 개방했다고 생각하면 되기 때문에 swarm cluster에 속한 모든 노드는 웹서비스에 접근가능하므로 매니저 노드(manager)에서 접속 해보면 똑같이 잘 되는 것을 확인가능합니다.

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/10.png){: .mx-auto.d-block width="80%" :}


서비스내의 Nginx container를 늘리고 줄이기 위해서는 <span style="color:DodgerBlue">docker service scale</span>명령어를 사용합니다. 다음과 같이 워커, 매니저 노드를 합한 수가 3이고 replica수를 4 replica수로 설정했다면 4 replica를 만족시켜야 하기때문에 한 노드(manager)에서 2개의 container가 실행됨을 확인가능합니다. 여기서 각각의 container들이 호스트의 80번 포트에 연결된 것이 아니고 실제로는 각 노의 80번 포트로 들어온 요청을 아래의 4개의 container중 1개로 redirect하게 됩니다. 

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/11.png){: .mx-auto.d-block width="100%" :}

서비스를 삭제 명령어는 다음과 같습니다.

```
docker service rm myweb
```
#### global 서비스 생성하기

지금까지는 --replica 옵션을 통해 container수를 정해주었지만 모든 노드에 container를 하나씩 생성하게 하는 global 서비스를 제공합니다. 

```
docker service create --name global_web --mode global nginx
```

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/12.png){: .mx-auto.d-block width="100%" :}


### 3.3 swarm mode의 서비스 장애 복구

위에서 생성한 container가 정지되거나 특정 노드가 다운되면 swarm manager는 새로운 container를 생성해 자동으로 이를 복구합니다. 실제로 그러한지 확인하기 위해 myweb 서비스 중 container하나를 삭제(<span style="color:DodgerBlue">docker rm -f [container name]</span>)해봅니다. 

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/13.png){: .mx-auto.d-block width="100%" :}

그럼 manager가 스스로 다시 replica수를 3개를 맞추기 위해 새로운 container를 manager노드에 만든것을 확인가능합니다. 이번에는 worker1 Host에서 daemon 프로세스를 종료(<span style="color:DodgerBlue">service docker stop</span>)하고 시스템 복구를 자동으로 하는 지 확인합니다. 

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/14.png){: .mx-auto.d-block width="100%" :}


{: .box-note}
참고로 global 서비스 모드에서는 container를 삭제하면 위와 같이 자동으로 복구하지만 노드를 중지시킬경우는 자동으로 복구되지 않습니다. 하지만 <span style="color:DodgerBlue">service docker start</span>를 다시 하여 daemon을 재시작시킬경우 자동 복구 됩니다. 


### 3.4 서비스 롤링 업데이트

롤링 업데이트란 여러 개의 server, container로 구성된 cluster의 설정이나 데이터 등을 변경하기 위해 하나씩 재시작하는 것을 의미합니다. 그리하여 하나를 업데이트를 해도 다른 server, container가 실행중이므로 지속적인 서비스가 가능합니다.

이번 예제에서는 nginx:1.10 image로 service를 만들고 nginx:1.11로 롤링 업데이트를 하는 방법으로 설명합니다. 서비스를 생성할 때 롤링 업데이트 주기(--update-delay)와 업데이트를 동시에 진행할 container 수(--update-parallelism)을 설정가능합니다. replica수는 4개, 주기는 10초, 동시에 업데이트 할 container를 2개로 지정하려면 다음과 같습니다.


```
docker service create --replicas 4 --name rolling_web --update-delay 10s --update-parallelism 2 nginx:1.10
```

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/15.png){: .mx-auto.d-block width="80%" :}

롤링 업데이트는 다음 명령어로 진행합니다. 아래와 같이 업데이트는 container 2개씩 진행됨을 확인가능하며 실제로 보면 주기가 10초임을 알 수 있습니다.

```
docker service update --image nginx:1.11 rolling_web
```

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/16.png){: .mx-auto.d-block width="80%" :}

그리고 롤링 업데이트 후에 서비스를 업데이트 전으로 되돌리고 싶다면 Rollback을 사용합니다.

```
docker service rollback rolling_web
```

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/17.png){: .mx-auto.d-block width="80%" :}



### 3.5 서비스 container에 설정 정보 전달: secret, config

swarm cluster에서 환경에 맞춘 설정 파일이나 값들이 컨테이너에 전달 될 수 있도록 하는 기능을 secret과 config가 하게 됩니다. 

- **secret**: 보안에 민감한 데이터 전송을 위함
	- 비밀번호, SSH 키, 인증서 키
- **config**: 암호화할 필요 없는 설정값
	- nginx, 레지스트리 설정 파일

#### secret

Mysql image로 서비스를 만들때 mysql 사용자 비밀번호를 container내부에 마운트하는 예제로 이해해보죠. 먼져 **passwd.txt**파일 안에 da2so라는 비밀번호를 적어놓고 다음 명령어로 secret을 생성합니다.

```
cat passwd.txt | docker secret create mysql_passwd -
```

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/18.png){: .mx-auto.d-block width="80%" :}

생성된 secret을 조회해도 비밀번호는 조회할수 없는데요. 이는 secret값은 매니저 노드 간에 암호화된 상태로 저장됩니다. secret파일은 container에 배포된 뒤에도 파일 시스템이 아닌 메모리에 저장되기 때문에 service container가 삭제되면 secret도 삭제되므로 휘발성을 띕니다. secret파일로 MySQL service를 생성해 보죠.

```
docker service create \
--name mysql \
--replicas 1 \
--secret source=mysql_passwd,target=mysql_root_password \
--secret source=mysql_passwd,target=mysql_password \
-e MYSQL_ROOT_PASSWORD_FILE="/run/secrets/mysql_root_password" \
-e MYSQL_PASSWORD_FILE="/run/secrets/mysql_password" \
-e MYSQL_DATABASE="wordpress" \
mysql:5.7
```

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/19.png){: .mx-auto.d-block width="80%" :}

--secret 옵션을 통해 container로 공유된 값은 기본적으로 container내부의 */run/secrets/* 에 마운트 되기때문에 환경변수에 대한 path가 */run/secrets/*하위 폴더를 가르키도록 옵션을 주는 것입니다. 그리고 다음과 같이 */run/secrets/mysql_password*에 secret값(da2so)이 잘 저장된 것을 확인가능합니다.


![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/20.png){: .mx-auto.d-block width="100%" :}

#### config

위에서 만든 **passwd.txt** 재사용해서 config를 사용해보죠. config파일 이름은 config_test로 합시다. 

```
docker config create config_test passwd.txt
```

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/21.png){: .mx-auto.d-block width="80%" :}


위에서 <span style="color:DodgerBlue">docker config inspect</span>명령어를 통해 data를 볼 수있고 이는 base64로 인코딩 되어있으므로 값을 보려면 **base64 -d**로 디코딩하면 값을 확인가능합니다. 그리고 secret과 동일하게 --secret부분을 --config부분으로 바꿔 사용하면 됩니다.(source, target을 동일하게 쓰면됩니다.) 그리고 service container가 새로운 설정 값을 사용해야한다면 docker service update명령어의 **--config-rm, --config-add, --secret-rm, --secret-add**옵션을 사용하여 서비스가 사용하는 secret이나 config를 추가 삭제가능합니다.


### 3.6 docker swarm network

swarm mode는 같은 container를 분산해서 할당하기 때문에 각 docker daemon의 네트워크가 하나로 묶인 즉, 네트워크 풀이 필요합니다. 또한 서비스를 외부로 노출했을 때 어느 노드로 접근하더라도 해당 서비스의 container에 접근 가능하도록 라우팅 기능도 필요합니다. 이를 위해 다음과 같이 swarm mode에서는 <span style="color:Crimson">docker_gwbridge</span>와 <span style="color:Crimson">ingress</span> 네트워크를 제공합니다.


![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/22.png){: .mx-auto.d-block width="70%" :}

#### ingress 네트워크 

로드밸런싱과 라우팅 메시(routing mesh)에 사용되는 네트워크로 swarm cluster생성시 자동으로 등록되는 네트워크입니다. 매니저 노드뿐 아니라 swarm cluster에 등록된 네트워크라면 ingress 네트워크가 생성되고 구조는 다음과 같습니다. 해당 구조는 구체적으로 swarm cluster로 nginx 웹 서비스를 했을때의 예시이며 위에서 말씀드렸듯이 어느 노드에 해당 container가 있느냐에 상관없이 swarm cluster안의 어떤 노드에서도 웹서비스에 접속가능한 이유입니다.

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/23.png){: .mx-auto.d-block width="90%" :}


이 네트워크는 어떤 swarm 노드에 접근하더라도 서비스 내의 container에 접근할 수 있게 설정하는 라우팅 메시를 구성하고 서비스 내의 container에 대한 접근을 라운드 로빈 방식으로 분산하는 로드 밸런싱을 담당합니다. 

{: .box-note}
**라운드로빈 방식**이란?? 서버에 들어온 요청을 순서대로 돌아가며 배정하는 방식입니다. 클라이언트의 요청을 순서대로 분배하기 때문에 여러 대의 서버가 동일한 스펙을 갖고 있고, 서버와의 연결(세션)이 오래 지속되지 않는 경우에 활용하기 적합


#### 오버레이 네트워크 

ingress 네트워크는 오버레이 네트워크 드라이버를 사용합니다. 오버레이 네트워크는 여러 개의 docker daemon을 하나의 네트워크 풀로 만드는 **네트워크 가상화** 기술의 하나이며 여러 docker daemon에 존재하는 container가 서로 통신할 수 있도록 해줍니다. 그래서 manager노드의 container는 별도의 포트 포워딩을 설정하지 않아도 worker1의 컨테이너의 ping을 전송가능합니다.


#### docker_gwbridge 네트워크

swarm에서 오버레이 네트워크를 사용할 때 사용되는 네트워크입니다. 오버레이 네트워크를 사용하지 않는 container는 기본적으로 존재하는 bridge를 사용해 외부와 연결합니다. 그러나 ingress를 포함한 모든 오버레이 네트워크는 다른 bridge 네트워크인 docker_gwbridge와 함께 사용됩니다. docker_gwbridge는 외부로 나가는 통신 및 오버레이 네트워크의 트래픽 종단점(VTEP)역할을 담당하고 container내부의 네트워크 interfacewnd eth1과 연결됩니다. 


### 3.7 swarm mode volume

daemon명령어 중 run명령어에서 -v옵션을 통해 host와 디렉터리를 공유하는데 이와 비슷하게 swarm cluster에서는 **--mount**옵션을 통해 volume타입, bind타입으로 호스토와 디렉토리를 공유하게됩니다.

#### volume type 볼륨 

**--mount**옵션에서 type=volume이며 source는 사용할 볼륨이며(해당 볼륨이 존재하지 않을경우 임의의 16진수로 이름을 구성) target은 컨테이너 내부에 마운트될 디렉터리 위치입니다.

```
docker service create --name volume_nginx --mount type=volume,source=volume_test,target=/root -p 80:80 nginx
```

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/24.png){: .mx-auto.d-block width="100%" :}


#### bind type 볼륨

bind타입은 host와 디렉터리를 공유할때 사용되므로 공유될 호스트의 디렉터리가 존재해야하며 이를 source 옵션에 반드시 명시해야합니다. type=bind을 사용합니다.

```
docker service create --name bind_nginx --mount type=bind,source=/home/kangsinhan/bind_test,target=/root -p 80:80 nginx
```

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/25.png){: .mx-auto.d-block width="100%" :}


#### swarm mode에서 volume 한계점

swarm cluster에서 서비스를 할당받을 수 있는 모든 노드가 volume data를 가지고 있어야하는 문제점이 있습니다. 특히나 PaaS같은 시스템에서는 어느 노드에 container을 할당해도 volume을 사용할수 있는 방버비 모든 노드에 같은 데이터 volume을 구성하는 것이므로 좋은 방법은 아닙니다. 

그래서 해결하기위해 persisent stroage을 쓰는 것이 방법인데 이는 외부에 존재해 네트워크로 Mount할수 있습니다. 이렇게 되면 노드에 volume을 굳이 생성하지 않아도 되고 container가 어느 노드에 할당되든 container에 필요한 파일을 읽고 쓸수 있습니다.


### 3.8 노드 AVAILABILITY 변경

<span style="color:DodgerBlue">docker node ls</span>을 통해 노드의 AVAILABILITY항목을 볼 수 있는데여. 이를 변경시킴으로써 특정 노드에 문제가 발생해 유지보수 작업을 수행할때 해당 노드에 container에 해당 작업이 할당안되게 할 수 있습니다.

#### Active

새로운 노드가 swarm clutser에 추가되면 기본적으로 설정되는 상태이며 노드가 서비스의 container를 할당 받을 수 있습니다. Active상태가 아닌 노드를 Active 상태로 바꾸고 싶으면 다음과 같다

```
docker node update --availability active worker1
```

#### Drain

해당 상태로 설정되면 scheduler는 container를 해당 노드에 할당하지 않습니다. 노드에 문제가 있을경우 Drain상태로 만듭니다.

```
docker node update --availability drain worker1
```
![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/26.png){: .mx-auto.d-block width="90%" :}


실행 중인 노드를 Drain상태로 변환시 서비스의 container는 모두 중지되고 active 상태인 노드로 재할당 됩니다.


#### Pause

해당 상태는 서비스의 container를 더는 할당받지 않는다는 점에서 drain과 동일하지만 실행 중인 container가 중지되지 않는다는 점에서 다릅니다.

```
docker node update --availability pause worker1
```

### 3.9 노드 label 추가

노드에 Label을 추가하는 것은 노드를 분류하는 것입니다. label은 key-value 형태를 가지고 있으며 key값으로 노드를 구별합니다. label추가는 <span style="color:DodgerBlue">docker node update</span>명령어의 **--label-add**옵션을 통해 key, value를 넣어줍니다. 

```
docker node update --label-add worker=1 worker1
```

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/27.png){: .mx-auto.d-block width="80%" :}


위에서 key는 worker, value는 1로 label을 설정해주었고 해당 노드에만 service의 container을 만들어낼 수 있습니다.

![1](https://da2so.github.io/assets/post_img/2022-01-12-Docker_Kubernetes7/28.png){: .mx-auto.d-block width="100%" :}


