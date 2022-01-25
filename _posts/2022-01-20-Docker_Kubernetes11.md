---
layout: post
title: Docker/Kubernetes - (11) Kubernetes 리소스의 관리와 설정
tags: [Docker, Kubernetes]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2022-01-05-Docker_Kubernetes1/logo.png
---

Enviroment: Ubuntu 18.04 
{: .box-note}
## 1. Namespace: 리소스를 논리적으로 구분하는 장벽
k8s에서 용도에 따라 container와 그와 관련된 리소스를 구분 지어 관리할 수 있는, 하나의 논리적인 그룹을 제공하기 위해 Namespace라는 object를 사용합니다. 예를 들어 모니터링을 위한 리소스들은 monitoring이라는 이름의 namespace로 생성될수 있고 테스트를 위한 리소스들은 test라는 namespace를 생성가능합니다. 


### 1.1 Namespace 이해

Namespace는 namespace(ns)라는 이름으로 k8s에서 사용가능하며 다음과 같이 namespace목록을 확인가능하다.

![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/1.png){: .mx-auto.d-block width="50%" :}


기본적으로 3개의 namespace가 존재하는데 각각의 namespace는 논리적인 리소스 공간이기 때문에 pod, replicaset, service와 같은 리소스가 따로 존재합니다. 예로 default라는 이름의 namepspace에 생성된 pod를 확인하려면 다음과 같이 **--namespace**또는 **-n**옵션을 사용한다.

```
kubectl get po -n default
```
![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/2.png){: .mx-auto.d-block width="80%" :}


default는 자동으로 사용하도록 설정되는 namespace로 kubectl 명렁어로 k8s 리소스를 사용할때 default namespace를 사용합니다. 즉, **--namespace**옵션을 명시하지 않으면 기본적으로 default namespace를 사용한다는 것이다. [이전 글](https://da2so.github.io/2022-01-18-Docker_Kubernetes10/)에서 사용했던 명령어는 모두 default namespace를 사용한것과 같다.


![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/3.png){: .mx-auto.d-block width="80%" :}

위와 같이 kube-syste namespace는 k8s 클러스터 구성에 필수적인 컴포넌트들과 설정값이 존재한는 namespace입니다. 위의 namespace는 pod에 관한 것들이지만 당연하게도 service, replicaset에도 별도의 namespace를 가지고 있습니다. 예로 다음은 kube-system namespace에는 k8s의 pod, service을 이름으로 찾을 수 있게 하는 DNS 서버의 serivce가 기본적으로 생성되어있습니다. 

![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/4.png){: .mx-auto.d-block width="75%" :}

namespace는 k8s의 리소스를 논리적으로 묶을 수 있는 가상 클러스터처럼 사용할수 있고 여러명이 사용한다면 사용자마다 namespace를 별도로 취할수있습니다. 하지만 중요한 점은 논리적으로 구분된것이므로 물리적으로 격리된것이 아니기때문에 서로 다른 namespace에서 생성된 pod라도 같은 node에 존재가능합니다. 기존의 배웠던 label과의 차이점 및 장점은 다음과 같습니다

- ResourceQuota object를 이용해 특정 namespace에서 생성되는 pod의 자원 사용량을 제한
- 애드미션 controller 기능을 이용해 특정 namespace에서 생성되는 pod에서는 항상 사이드카 container붙이도록 할수있음
- 사용목적에 따라 pod, service등의 리소스를 격리하여 구분가능


### 1.2 Namespace 사용

namespace는 다음과 같이 production-ns.yaml을 통해서나 <span style="color:DodgerBlue">kubectl create namespace</span>로 생성가능합니다.

```
#production-ns.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

```
# from yaml
kubectl apply -f production-ns.yaml
# from create
kubectl create namespace production
```
![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/5.png){: .mx-auto.d-block width="60%" :}

그리고 해당 namespace에 리소스를 생성하는 방법은 다음과 같이 metadata.namespace 항목에 만들어놓은 namespace를 입력하면 됩니다.

```
#deployment-nginx-svc-ns.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-nginx-svc-ns
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-nginx
  template:
    metadata:
      name: my-nginx-pod
      labels:
        app: my-nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.10
        ports:
        - containerPort: 80
--- 
apiVersion: v1
kind: Service
metadata:
  name: svc-clusterip-ns
  namespace: production
spec:
  ports:
    - name: web-port
      port: 8080
      targetPort: 80
  selector:
    app: my-nginx
  type: ClusterIP
```

---을 통해 여러개의 리소스를 정의를 한 것이고 각 리소스에 namespace를 production으로 설정하고 다음과 같이 apply해보면 service와 deployment가 해당 production namespace에 생성된것을 확인가능합니다.

![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/6.png){: .mx-auto.d-block width="80%" :}

### 1.3 Namespace의 service에 접근 

다른 namespace에 존재하는 service에는 service이름만으로 접근 불가하게 됩니다. 예를 들어 다음과 같이 테스트용으로 만든 임시용 pod는 default namespace를 사용하므로 production namespace의 service에 접근하지 못합니다. 

```
# run ubuntu pod for testing and connect to it
kubectl run -i --tty --rm debug --image=ubuntu:16.04 --restart=Never -- bash

#install curl
sudo apt-get update
sudo apt-get install curl

# http request to service of production namespace
curl svc-clusterip-ns:8080
```

![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/7.png){: .mx-auto.d-block width="60%" :}

하지만 **$<$service 이름$>$$<$namespace 이름$>$.svc**와 같이 service이름 뒤에 namespace를 붙이면 다른 namespace에 접근가능합니다.

![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/8.png){: .mx-auto.d-block width="60%" :}

namespace는 <span style="color:DodgerBlue">kubectl delete -f [yaml 파일]</span> 또는 <span style="color:DodgerBlue">kubectl delete namespace</span>명령어로 삭제가능하며 namespace삭제시 해당 namespace에 존재하는 리소스도 함께 삭제됩니다.


### 1.4 namespace에 종속되는 k8s object와 독립적인 object

A라는 namespace에 존재한는 pod는 A namespace에서만 보이고 B namespace에는 보이지 않습니다. 이를 **object가 namespace에 속한다(namespaced)**라고 표현합니다. namespace에 속하는 object의 종류는 <span style="color:DodgerBlue">kubectl api-resources --namespaced=true</span>로 확인가능합니다.

![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/9.png){: .mx-auto.d-block width="90%" :}


위에서 설명한 pod, service, deployment가 namespace에 속하는 것을 확인가능하다. 그럼 반대로 속하지 않는 object로는 다음과 같이 node, namespace그 자체도 포함이다. 그래서 namespace에 포함하려해도 포함되지 않는다.


![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/10.png){: .mx-auto.d-block width="90%" :}


## 2. Configmap, Secret: 설정값을 Pod에 전달

대부분의 application은 설정값을 가지고 있습니다. 예를 들어 application loggin level을 정의하는 LOG_LEVEL=INFO와 같이 단순한 key-value형태의 설정을 사용할 수 있습니다. 이를 위해 k8s는 pod를 정의하는 YAML파일에 환경변수를 직접 적어놓은 하드 코딩방식을 사용할 수 있습니다. 아래는 기존에 사용하던 deployment-nginx.yaml에 env환경변수를 추가해 env-deployment-nginx.yaml로 만든 예시입니다. 

```
#env-deployment-nginx.yaml
...
  spec:
    containers:
    - name: nginx
      env:
      - name: LOG_LEVEL
        value: INFO
      image: nginx:1.10
...
```

위는 pod의 LOG_LEVEL이라는 이름의 환경변수를 INFO라고 설정한 것입니다. 이렇게 환경 변수를 직접 pod template에 명시해두 되지만 운영 및 개발 환경에서 각각 다른 deplotment를 생성해야한다면 환경변수가 서로 다르게 설정된 두 가지 버전의 YAML이 필요하게 됩니다. k8s에서는 YAML파일과 설정값을 분리 할 수 있는 <span style="color:Crimson">Configmap</span>과 <span style="color:Crimson">Secret</span>의 object를 제공합니다. Configmap은 설정값을, Secret은 노출되어서는 안되는 비밀 값을 저장합니다.

그래서 configmap을 이용한다면 1개의 pod YAML파일만을 사용하되 환경에 따라 다른 configmap을 생성해 사용할 수 있다. 즉, 환경 변수나 설정값까지 k8s object에서 관리하므로 이러한 설정값 또한 YAML파일로 pod와 함께 배포가능하다는 장점이 있다.

### 2.1 Configmap 사용

일반적인 설정값을 담아 저장할 수 있는 k8s object이고 namespace에 속하기 때문에 namespace별로 configmap이 존재한다. YAML을 사용해서 configmap을 생성할 수 있지만 <span style="color:DodgerBlue">kubectl create cm [configmap 이름] $[$각종 설정값들$]$ </span>을 통해 쉽게 생성가능하다. (cm은 configmap과 동일)

![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/11.png){: .mx-auto.d-block width="90%" :}


다음과 같이 **--from-literal**옵션을 여러번 사용함으로써 여러 개의 key-value을 configmap에서 사용하도록 할수 있습니다. 그리고 configmap에 저장된 설정값은 <span style="color:DodgerBlue">kubectl describe cm</span>과 <span style="color:DodgerBlue">kubectl get cm -o yaml</span>으로 확인 가능하다.


![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/12.png){: .mx-auto.d-block width="100%" :}

![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/13.png){: .mx-auto.d-block width="55%" :}

configmap의 설정값을 pod에 적용해보기 전에 어떤 방법으로 configmap이 사용되는 지를 알아보죠.

![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/14.png){: .mx-auto.d-block width="100%" :}


- configmap의 값을 container 환경 변수로 사용
  - 위와 같은 그림이 해당 예시입니다. container내부의 환경변수 key-value값으로 설정하는 경우입니다.
- configmap의 값을 pod 내부의 파일로 마운트해 사용
  - configmap의 값을 pod container내부의 특정 파일로 마운트합니다.
  - 예를 들어 LOG_LEVEL=INFO라는 값을 가지는 configmap을 /etc/config/log_level이라는 파일로 마운트하면 log_level파일에 INFO라는 값이 저장됩니다.

#### config map의 데이터를 container 환경 변수로 가져오기

이제 실제로 configmap의 설정값을 pod에 적용해보기 위해 다음과 같은 내용으로 env-configmap.yaml을 만들어보죠.

```
#env-configmap.yaml
apiVersion: v1
kind: Pod
metadata:
  name: container-env-configmap
spec:
  containers:
    - name: my-container
      image: busybox
      args: ['tail', '-f', '/dev/null']
      envFrom:
      - configMapRef:
          name: log-level-configmap
      - configMapRef:
          name: start-k8s
```

configmap과 연동되는 부분이 **envFrom**과 **configMapRef** 항목입니다. 이전에 생성한 log-level-configmap과 start-k8s라는 configmap의 값을 가져와 환경변수로 설정하는 YAML입니다. **envFrom**은 하나의 configmap에 여러 개의 key-value 쌍이 존재하더라도 모두 환경변수로 가져올수 있게 하고 **configMapRef**하위항목의 name와 매칭되는 configmap을 명시하게 됩니다.

![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/15.png){: .mx-auto.d-block width="70%" :}


그리고 다음과 같이 **envFrom**과 다르게 **valueFrom**, **configMapKeyRef**으로는 key값도 넣어서 해당 configmap에 해당하는 key값을 선택하여 그에 대한 value값을 환경변수로 설정하게됩니다. 그래서 아래를 보면 ENV_KEYNAME_1이라는 환경변수 key값을 만들고 그에 대한 value값을 log-level-configmap의 LOG_LEVEL의 value값으로 설정합니다.

```
#env-valuefrom-configmap.yaml
apiVersion: v1
kind: Pod
metadata:
  name: container-valuefrom-env-configmap
spec:
  containers:
    - name: my-container
      image: busybox
      args: ['tail', '-f', '/dev/null']
      env:
      - name: ENV_KEYNAME_1 #container에 새롭게 등록될 환경 변수 key값
        valueFrom:
          configMapKeyRef:
            name: log-level-configmap
            key: LOG_LEVEL
      - name: ENV_KEYNAME_2
        valueFrom:
          configMapKeyRef:
            name: start-k8s
            key: k8s
```

![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/16.png){: .mx-auto.d-block width="70%" :}


#### configmap의 내용을 파일로 pod 내부에 마운트

application이 nginx.conf와 같은 특정 파일로부터 설정값을 읽어올 수 있습니다. 예를 들어, 다음과 같은 YAML파일은 start-k8s configmap에 존재하는 모든 key-value 쌍을 /etc/config 디렉터리에 위치시킵니다.

```
#env-volume-configmap.yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-env-configmap
spec:
  containers:
    - name: my-container
      image: busybox
      args: [ "tail", "-f", "/dev/null" ]
      volumeMounts:
      - name: configmap-volume          # volumes에서 정의한 컨피그맵 볼륨 이름 
        mountPath: /etc/config             # 컨피그맵의 데이터가 위치할 경로

  volumes:
    - name: configmap-volume            # 컨피그맵 볼륨 이름
      configMap:
        name: start-k8s
```

**volumeMounts**와 **volumes**라는 항목은 다음과 같은 역할을 수행합니다. 

- spec.volumes: YAML파일에서 사용할 볼륨의 목록을 정의합니다.
  - 위에서는 start-k8s라는 configmap을 통해 configmap-volume을 정의하였습니다.
- spec.containers.volumeMounts: volume 항목에서 정의된 볼륨을 container내부의 어떤 디렉터리에 마운트할것인지 명시
  - 위에서는 /etc/config 디렉터리에 configmap의 값이 담긴 파일이 마운트됨

![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/17.png){: .mx-auto.d-block width="70%" :}

위처럼 /etc/config라는 폴더에 start-k8s의 key값들로 파일을 만들고 해당 파일내용에는 value값이 들어간다. 그리고 다음과 같이 volumes항목을 다음과 같이 바꾸면 원하는 key-value쌍만 가져올수 있습니다. 


```
#env-volume-configmap.yaml (원하는 key-value선택)
...
  volumes:
    - name: configmap-volume            # 컨피그맵 볼륨 이름
      configMap:
        name: start-k8s
        items:
        - key: k8s
          path: k8s_fullname
```

- items: configmap에서 가져올 key-value의 목록을 뜻함
- path: 최종적으로 디렉터리에 위치할 파일의 이름을 입력

![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/18.png){: .mx-auto.d-block width="70%" :}

#### 파일로부터 configmap 생성

실제 환경에서는 설정 파일 그자체를 configmap으로 사용하는 경우가 많습니다. 이를 위해 기존의 configmap생성시 **--from-literal**옵션 대신 **--from-file**이라는 옵션을 통해 파일로 부터 configmap을 생성가능합니다. 

```
kubectl create configmap <configmap 이름> --from-file <파일 이름>
```

![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/19.png){: .mx-auto.d-block width="70%" :}


위와같이 index.html에서 별도의 key를 정의하지 않았으므로 파일 이름(index.html)이 key이고 해당 value는 파일내용이됩니다. 그리고 **--from-env-file**옵션으로 여러개의 key-value형태로 구성된 설정파일을 한번에 가져올 수 있습니다. 

```
#file-configmap.env
a=1
b=2
c=3
```

```
kubectl create configmap file-env-configmap --from-env-file file-configmap.env
```

![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/20.png){: .mx-auto.d-block width="70%" :}

정적 파일을 pod에 제공하려면 --from-file을 사용하고 여러개의 환경 변수를 pod로 가져올 경우는 --from-env-file을 쓰시면 됩니다.


## 3. Secret

Secret은 ssh key, 비밀번호와 같은 보안이 필요한 정보를 저장하기 위해 사용되는 object입니다. 사용방법은 configmap과 유사하다.

### 3.1 image registry 접근을 위한 docker

다음과 docker hub에 private한 docker image를 올렸습니다. (직접 예제를 진행해보려면 docker hub에서 image push하셔야 하셔야 하는데 [이글](https://da2so.github.io/2022-01-08-Docker_Kubernetes4/)을 참고하세여.) 해당 private docker image를 pull받으려면 해당 docker image를 소유하는 계정의 id, password가 필요하겠죠.

![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/21.png){: .mx-auto.d-block width="50%" :}

그러면 지금부터 secret object로 id, password에 대한 정보를 생성하고 이를 deployment를 만드는 yaml에 해당 secret정보를 넣어 pod를 만드는 예제를 진행해보죠. secret키를 만드는 명령어는 다음과 같습니다.

```
kubectl create secret docker-registry registry-auth --docker-username=das2o --docker-password=<비밀번호>
```
- docker-registry: secret 타입으로 이외의 다른것(Opaque)등이 존재
- registry-auth: secret 이름

![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/22.png){: .mx-auto.d-block width="90%" :}


위와 같이 <span style="color:DodgerBlue">kubectl get secret [secret 이름] -o yaml</span>로 .dockerconfigjson이라는 데이터를 확인가능하고 해당 데이터는 base64로 인코딩되어있으므로 다음과 같이 디코딩가능하다. 디코딩해보면 secret을 만들때의 정보들을 볼 수 있다.

![1](https://da2so.github.io/assets/post_img/2022-01-20-Docker_Kubernetes11/21.png){: .mx-auto.d-block width="100%" :}

그리고 다음은 deployment를 생성하는 yaml이다. 해당 object에서는 private image를 pull하기위해서 secret object를 지정해줘야만 한다. 다음과 같이 정의해주면 secret object의 key-value값을 통하여 계정 인증을 하고난뒤 private image pull이 가능해지는 것이다. 

```
#deployment-secret.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-secret
spec:
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      name: mypod
      labels:
        app: myapp
    spec:
      containers:
      - name: test-container
        image: da2so/test_repo:0.0
        args: ['tail', '-f', '/dev/null']
      imagePullSecrets:
      - name: registry-auth
```

## 4. 리소스 정리

위의 예제들은 생성한 리소스가 많을텐데 다 지우고 싶으면 다음과 같은 명령어를 사용한다.

```
kubectl delete deployment --all
kubectl delete pod --all
kubectl delete configmap --all
kubectl delete secret --all 
```





