---
layout: post
title: Docker/Kubernetes - (2) Docker image/cotainer
tags: [Docker, Kubernetes]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2022-01-05-Docker_Kubernetes1/logo.png
---

Enviroment: Ubuntu 18.04 
{: .box-note}
## 1. Docker image/container
Docker Engine에서 사용하는 기본 단위가 container와 image이고 image는 container를 생성할때 필요한 요소입니다.

### 1.1 Docker image 

Docker image 특성
- 여러 개의 계층으로 된 바이너리 파일
- 컨테이너를 생성하고 실행할 때 읽기 전용으로 사용
- 도커 명령어로 image 다운 가능

Docker image 구성은 다음과 같습니다.

![docker_image](https://da2so.github.io/assets/post_img/2022-01-06-Docker_Kubernetes2/1.png){: .mx-auto.d-block width="50%" :}

- 저장소
	- 이미지가 저장된 장소를 의미하고 명시되지 않을경우 docker hub라는 공식 이미지 저장소에서 해당 이미지를 가져옵니다.
- 이미지 이름
	- 가져올 이미지를 뜻하고 그에 대한 이름은 어떤 역할을 하는지를 나타냅니다.
- 태그
	- 이미지의 버전 관리 혹은 리비전 관리에 사용됩니다. latest는 등록된 버전 중 가장 최신버전을 의미합니다.

### 1.2 Docker container

위에서 설명한 image로 container를 생성하면 해당 이미지의 파일시스템과 격리된 시스템 자원 및 네트워크를 사용할 수 있는 독립적 공간이 생성됩니다.

다음과 같이 한 image로 여러개의 docker container를 생성가능하며 생성된 각 container는 독립된 공간을 가지기 때문에 각 container안에서 새로 설치할 파일은
다른 container에 영향을 주지 않습니다.

![docker_container](https://da2so.github.io/assets/post_img/2022-01-06-Docker_Kubernetes2/2.png){: .mx-auto.d-block width="60%" :}

## 2. Docker image 다운로드 및 container 생성

docker engine의 pull명령어로 docker hub(docker image들이 저장되어있는 공식적인 장소)에서 ubuntu:16.04라는 docker image를 다운받아 봅시다.

```bash
docker pull ubuntu:16.04 
```

image가 잘 다운되었는지 확인하려면 "docker images"로 확인가능합니다.

![pull_check](https://da2so.github.io/assets/post_img/2022-01-06-Docker_Kubernetes2/3.png){: .mx-auto.d-block width="60%" :}


docker engine의 run명령어로 ubuntu:16.04 image로 container생성을 해봅시다.

```bash
docker run -i -t  ubuntu:16.04 
```

다음과 같이 container안으로 들어오게 되고 해당 container안에서 ubuntu 버전 확인시 16.04임을 확인가능합니다. 참고로 -i -t는 컨네이터 내부로 들어가게 하여 상호작용 가능한 셸환경을 설정하는 것입니다. 


![container_create](https://da2so.github.io/assets/post_img/2022-01-06-Docker_Kubernetes2/4.png){: .mx-auto.d-block width="50%" :}


docker run에 유용한 옵션은 다음과 같습니다.

<details>
<summary>--name [명칭]</summary>
<div markdown="1">

```bash
docker run -i -t  --name myubuntu ubuntu:16.04 
```
name옵션은 container의 명칭을 정해주는 것입니다. 
</div>
</details>


<details>
<summary>-p [포트]</summary>
<div markdown="1">

```bash
docker run -i -t -p 88:80 ubuntu:16.04 
```
-p 뒤의 88(호스트의 포트):80(컨테이너 포트)로 호스트와 컨테이너의 포트를 바인딩시킵니다. 
컨테이너안의 apache 웹서비스가 설치되어 있고 해당 웹서비스가 80포트를 사용하게될 경우 다음과 같은 접근이 가능합니다.
1. 외부에서 호스트의 IP의 88번 포트로 접근
2. 88번 포트는 컨테이너의 80번 포트로 포워딩
3. 컨테이너 안의 웹서버 접근 가능

</div>
</details>

<details>
<summary>-e [환경설정]</summary>
<div markdown="1">

```bash
docker run -i -t -e MYSQL_ROOT_PASSWORD=password mysql 
```
-e옵션은 컨테이너 내부의 환경변수를 설정합니다. DB(mysql)과 같은 특정 image에서는 e옵션을 통해 환경설정을 안해줄경우 실행이 되지않는 경우가 있으니 주의하세요.

</div>
</details>


<details>
<summary>-v (볼륨)</summary>
<div markdown="1">

```bash
docker run -i -t -v /home/wordpress_db:/var/lib/mysql mysql:5.7
```
-v옵션 뒤의 home/wordpress_db(호스트의 볼륨):/var/lib/mysql(컨테이너 볼륨)은 호스트의 디렉터리와 컨테이너의 디렉터리를 공유한다는 뜻입니다.
</div>
</details>

<details>
<summary>--workdir (실행 디렉터리)</summary>
<div markdown="1">

```bash
docker run -i -t --workdir="/var/www" nginx:latest
```
--workdir 옵션 뒤의 /var/www 디렉토리에서 프로세스가 실행된다라는 뜻입니다.
</div>
</details>

## 3. Docker container관련 유용한 명령어


docker engine의 유용한 명령어는 다음과 같습니다.

1. **정지된 container 포함하여 모든 container 출력**
```bash
docker ps -a
```

2. **실행 중인 컨테이너 정지/삭제**
```bash
docker stop/rm ${CONTAINER ID 또는 NAMES}
```

3. **실행 중인 모든 컨테이너 정삭제**
```bash
docker container prune
```

3. **정지된 컨테이너 시작/접속**
```bash
docker start/attach ${CONTAINER ID 또는 NAMES}
```