---
layout: post
title: Docker/Kubernetes - (12) Kubernetes Ingress
tags: [Docker, Kubernetes]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2022-01-05-Docker_Kubernetes1/logo.png
---

Enviroment: Ubuntu 18.04 
{: .box-note}
## 1. Ingress
Ingress network는 외부에서 서버로 들어오는 트래피을 처리하며 네트워크 7계층 레벨에서 정의되는 k8s object입니다. 주요 기능은 다음과 같습니다.

- **외부 요청의 라우팅**
  - /request, /request/you 등과 같이 특정 경로로 들어오는 요청을 어떠한 service로 전달할지 결정
- **가상 호스트의 요청 처리**
  - 같은 IP에 대해 다른 도메인 이름으로 요청이 도착했을 때, 어떻게 처리할 지 정의
- **SSL/TLS 보안 연결 처리**
  - 여러개의 서비스로 요청을 라우팅할 때, 보안 연결을 위한 인증서를 쉽게 적용

### 1.1 Ingress 사용 이유

구체적으로 기존에 설명하였던 NodePort, LoadBalancer타입의 service도 위와 같은 기능을 구현가능하지만 Ingress를 사용하는 이유는 다음과 같습니다. 


![1](https://da2so.github.io/assets/post_img/2022-01-22-Docker_Kubernetes12/1.png){: .mx-auto.d-block width="100%" :}


왼쪽 그림처럼 기존의 NodePort 또는 LoadBalancer타입의 service는 deployement 3개를 외부에 노출해야한다면 service가 3개필요합니다. 그리고 service마다 세부적인 설정을 할때 추가적인 복잡성이 발생하게 되고 SSL/TLS 보안 연결, 접근 도메인 및 클라이언트 상태에 기반한 라우팅을 구현하려면 각 service와 deployement에 일일이 설정해야합니다. 


하지만 오른쪽과 같이 **Ingress**를 사용하면 3개의 service에 각각 URL이 존재하지 않고 Ingress에 접근하기위한 단 하나의 URL만 존재합니다. 그래서 클라이언트는 Ingress의 URL로 접근하게 되며 해당 요청은 Ingress에 정의된 규칙에 따라 처리된뒤 deployment에 전달됩니다. 이 과정에서 라우팅 정의나 보안 연결 등과 같은 세부 설정은 Ingress에서 수행됩니다. 


### 1.2 Ingress 구조


<span style="color:DodgerBlue">kubectl get ing</span>으로 ingress의 목록을 확인가능하다. 확인 시 ingress목록이 없으니 ingress-example.yaml파일로 생성해보자.

![1](https://da2so.github.io/assets/post_img/2022-01-22-Docker_Kubernetes12/2.png){: .mx-auto.d-block width="55%" :}


```
#ingress-example.yaml  
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-example
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: da2so.com
    http:
      paths:
      - path: /hostname
        pathType: Prefix
        backend:
          service:
            name: hostname-svc
            port:
              number: 8080 
```

- **host**: 해당 도메인 이름으로 접근하는 요청에 대해서 처리 규칙을 적용
  - 여러개의 host를 가질 수 있음
- **path**: 해당 경로에 들어온 요청은 어느 서비스로 전달할지 정의
  - 위에서는 /etc-hostname으로 온 요청을 backend에 정의된 service에 전달
- **pathType**: 경로의 유형
  - Prefix: URL 경로의 접두사를 / 를 기준으로 분리한 값과 일치시킴 (/hostname, /hostanme/ 가능)
  - Exact: URL 경로의 대소문자를 엄격하게 일치시킴 (/hostname 만 가능)
- **service**: path로 들어온 요청이 전달될 내용을 담고있다
  - **name, port.number**: service 이름, port 번호


![1](https://da2so.github.io/assets/post_img/2022-01-22-Docker_Kubernetes12/3.png){: .mx-auto.d-block width="70%" :}

minimal-ingress라는 이름으로 ingress를 생성했지만 이는 단지 요청을 처리하는 규칙을 정의하는 선언적인 object이다. 그래서 외부 요청을 받아들일 수 있는 실제 서버가 아니기 때문에 <span style="color:Crimson">Ingress Controller</span>라는 특수한 서버에 적용해야만 그 규칙을 사용가능하다. 즉, 실제로 외부 요청을 받아들이는 것은 Ingress controller server이며 이 서버가 ingress 규칙을 로드해 사용합니다.


![1](https://da2so.github.io/assets/post_img/2022-01-22-Docker_Kubernetes12/4.png){: .mx-auto.d-block width="85%" :}


그래서 k8s의 ingress는 반드시 ingress controller를 필요로하며 우리는 nginx 웹서버를 사용하므로 **Ngnix 웹서버 Ingress controller**를 사용합니다. Kong이라는 API gateway나 GKE의 클라우드 플랫폼에서 제공되는 ingress controller도 있음을 알아두면 좋다. Nginx 웹서버 Ingress controller는 다음과 같은 명령어로 Nginx ingress controller와 관련된 resource를 다운받습니다. (제가 사용한 k8s 버전이 1.23인데 이는 controller 버전 1.1.1과 연동가능합니다. )

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.1/deploy/static/provider/cloud/deploy.yaml
```

![1](https://da2so.github.io/assets/post_img/2022-01-22-Docker_Kubernetes12/5.png){: .mx-auto.d-block width="85%" :}

위의 그림은 ingress controller의 역할 및 관계를 나타낸 그림**(A)**이다. 이제 <span style="color:DodgerBlue">kubectl get all -n ingress-nginx</span>명령어로 생성한 ingress-controller에 의해 생성된 ingress-nginx namespace의 모든 object를 확인가능합니다.

![1](https://da2so.github.io/assets/post_img/2022-01-22-Docker_Kubernetes12/6.png){: .mx-auto.d-block width="90%" :}


 여기서 default로 ingress-nginx-controller의 service type이 LoadBalancer로 되어있는데 저는 NodePort service로 진행할것이기 때문에 <span style="color:DodgerBlue">kubectl -n ingress-nginx edit service/ingress-nginx-controller</span>명령어를 통해 다음과 같이 해당 내용을 수정해줍니다. 그리고 다시 ingress-nginx-controller의 service type이 NodePort로 변환된것을 확인가능합니다.


![1](https://da2so.github.io/assets/post_img/2022-01-22-Docker_Kubernetes12/7.png){: .mx-auto.d-block width="90%" :}


이제 **A** 그림에서 hostname-service-nodeport 서비스 부분과 deployment에 대한 yaml을 다음과 같이 작성합니다.

```
#ingress-deployment-service.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ingress-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webserver
  template:
    metadata:
      name: my-webserver
      labels:
        app: webserver
    spec:
      containers:
      - name: my-webserver
        image: nginx:1.10
        ports:
        - containerPort: 80
          name: nginx-port
---
apiVersion: v1
kind: Service
metadata:
  name: hostname-svc
spec:
  ports:
    - name: web-port
      port: 8080
      targetPort: 80
  selector:
    app: webserver
  type: NodePort

```

<span style="color:DodgerBlue">kubectl apply -f ingress-deployment-service.yaml</span>명령어로 object들을 실행시키고 나면 다음과 같이 deployment와 **NodePort**타입의 service가 생성된다.

![1](https://da2so.github.io/assets/post_img/2022-01-22-Docker_Kubernetes12/8.png){: .mx-auto.d-block width="80%" :}

마지막으로 현재 예시는 on-premise환경이기 때문에 마스터노드에서 /etc/hosts파일에 IP와 도메인을 설정해 임시로 동작 여부를 테스트하도록한다. 다음과 같이 /etc/hosts에 da2so.example.com과 워커 노드 IP와 연결한다. 이는 ingress controller는 기본적으로 도메인 이름으로 연결되기 때문에 도메인을 IP연동시켜줘야하는 부분이다. 

![1](https://da2so.github.io/assets/post_img/2022-01-22-Docker_Kubernetes12/9.png){: .mx-auto.d-block width="80%" :}

위와 같이 도메인과 IP를 연결해주는 내용을 추가해주면 ingress의 address에 ingress controller service clusterIP로 연결이 되었다. nginx ingress controller는 항상 ingress 리소스의 상태를 지켜보고 있으며 기본적으로 모든 namespace의 ingress리소스를 읽어와 규칙을 적용하게 되는 것입니다. 위의 모든 설정을 그림으로 나타내면 아래와 같고 외부에서 da2so.com:30172/hostname에 접속하는 것은 다음과 같은 프로세스를 거친다.


![1](https://da2so.github.io/assets/post_img/2022-01-22-Docker_Kubernetes12/10.png){: .mx-auto.d-block width="100%" :}


1. 외부에서 da2so.com:30172/hostname로 request
  1. 일반적으로 외부에서 request가 들어오지만 저는 예시를 위해 마스터노드에서 request를 진행
  2. 마스터노드의 /etc/hosts 에 da2so.com을 worker node ip와 연결시켜놓은 상태
  3. 그래서 외부에서 접속되는 경우에는 일반적으로 router에다가 ingress에서 설정한 domain과 ip를 연결시켜줘야함 
2. da2so.com:30172는 ingress controller service로 보내짐 
  1. nodeport 타입의 service를 가지는 ingress controller이기 때문에 port(30172)를 지정하였음
3. ingress controller의 ingress규칙에 따라 da2so.com:30172/hostname은 da2so.com은 worker node ip에 연결되고 해당 request는 hostname-svc:8080라는 service로 전달됨
4. hostname-svc는 nginx:1.10 image기반인 pod와 연결되므로 해당 pod의 80번 Port에 접근하여 웹서비스 request를 받음


### 1.3 Ingress 세부 기능: annotation을 이용한 설정 


위의 **ingress-example.yaml**에서 annotation부분에 대해 설명하지 않았는데 여기서 설명하겠다. 다음은 위에서 작성한 anntotation부분을 가져온 것이다.


```
#ingress-example.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-example
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    kubernetes.io/ingress.class: "nginx"
...
      paths:
      - path: /hostname
...
```

- **kubernetes.io/ingress.class**
  - ingress 규칙을 어떤 ingress controller에 적용할것인지 정함 (Ex: Nginx, Kong, GKE, ...)
- **nginx.ingress.kubernetes.io/rewirte-target**
  - nginx ingress controller에서만 사용가능하며 ingress에 정의된 경로로 들어오는 요청을 rewrite-target에 설정된 경로로 전달합니다.
  - 위에서는 path에 적힌 /hostname으로 접근하면 hostname-svc에는 / 경로로 전달됩니다. 

그리고 다음(yaml)과 같이 정규식으로 설정할 경우 ingress에 요청온 path는 hostname-svc으 다음(밑의 그림)경로로 전달됩니다. 

```
#ingress-example.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-example
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2 # path의 (.*)에서 전달받은 경로로 전달합니다.
    kubernetes.io/ingress.class: "nginx"
...
      paths:
      - path: /hostname(/|$)(.*) # (.*)을 통해 경로를 얻습니다.
...
```


![1](https://da2so.github.io/assets/post_img/2022-01-22-Docker_Kubernetes12/11.png){: .mx-auto.d-block width="60%" :}

참고! Nginx Ingress Controller는 bypassing이라는 기능을 통하여 application pod에 트래픽을 직접 전달합니다. 해당 Pod의 Service를 경유해야 하는 네트워크 홉을 줄이게 됩니다.
{: .box-note}


### 1.4 Nginx ingress controller에 SSL/TLS보안 연결 적용

Ingress의 장점은 ingress controller에서 편리하게 SSL/TLS 보안 연결을 설정할 수 있다는 것입니다. 즉, Ingress controlller지점에서 인증서를 적용해 두면 요청이 전달되는 application에 대해 모두 인증서 처리가능하다. 이번 글에서는 직접 서명한 루트 인증서를 통해 nginx ingress controller에 적용해보자. 

보안 연결에 사용할 인증서와 비밀키를 생성해보자.

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=da2so.com/O=da2so"
```

![1](https://da2so.github.io/assets/post_img/2022-01-22-Docker_Kubernetes12/12.png){: .mx-auto.d-block width="100%" :}

tls.key라는 비밀키와 tls.crt라는 인증서가 생성되었습니다. 그리고 secret object를 다음과 같이 만든다.


```
 kubectl create secret tls tls-secret --key tls.key --cert tls.crt
```
![1](https://da2so.github.io/assets/post_img/2022-01-22-Docker_Kubernetes12/13.png){: .mx-auto.d-block width="80%" :}


tls을 적용한 ingress를 작성하기 전에 위에서 사용한 **ingress-deployment-service.yaml**을 통해 service와 deployment를 실행시키자.

![1](https://da2so.github.io/assets/post_img/2022-01-22-Docker_Kubernetes12/14.png){: .mx-auto.d-block width="80%" :}


이제 tls가 적용될 ingress yaml을 다음과 같이 작성한다. 그리고 해당 ingress을 생성하고 생성한 ingress의 정보와 nginx ingress controller의 https(443port)의 정보를 확인한다.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-example
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    kubernetes.io/ingress.class: "nginx"
spec:
  tls:
  - hosts:
    - da2so.com
    secretName: tls-secret
  rules:
  - host: da2so.com
    http:
      paths:
      - path: /hostname
        pathType: Prefix
        backend:
          service:
            name: hostname-svc
            port:
              number: 8080
```

- spec.tls.hosts: 보안 연결을 적용할 도메인 이름
- spec.tls.secretName: 위에서 생성하였던 tls 타입의 secret 이름

![1](https://da2so.github.io/assets/post_img/2022-01-22-Docker_Kubernetes12/15.png){: .mx-auto.d-block width="100%" :}

위의 그림에서 알 수 있듯이 ingress의 정보에 tls연결 정보가 새로 생긴것을 확인가능하며 tls보안이 있기때문에 https로 접속해야하는데 https로 접속하기 위한 ingress controller의 https포트는 31355인것을 알 수 있다. 즉, **$https$://da2so.com:31355**는 https와 31355와 맵핑되는 443포트(https)를 통해 ingress controller의 https로 접근을 명시하는 것이고 그다음은 위에서 설명한것과 같이 ingress controller에게 da2so.com과 연결되는 ip주소에 접속하도록 요청하는것이다. 다음 명령어를 통해 https연결을 통해 web service에 접속해보자.

```
curl https://da2so.com:31355/hostname -k
# -k 옵션은 신뢰할 수 없는 인증서로 보안연결을 위함이다.
```

![1](https://da2so.github.io/assets/post_img/2022-01-22-Docker_Kubernetes12/16.png){: .mx-auto.d-block width="80%" :}

