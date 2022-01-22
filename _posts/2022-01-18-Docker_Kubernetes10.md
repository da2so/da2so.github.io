---
layout: post
title: Docker/Kubernetes - (10) Kubernetes 이해 및 사용
tags: [Docker, Kubernetes]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2022-01-05-Docker_Kubernetes1/logo.png
---

Enviroment: Ubuntu 18.04 
{: .box-note}
## 1. Kubernetes(k8s) 이해
Kubernetes(k8s)이 가지는 고유한 특성에 대해 알아봅시다.

#### <span style="color:DarkOrchid">1. 모든 리스소는 object형태로 관리됨</span>

이전 글에서 swarm mode의 container 집합을 service(서비스)라고 하였습니다. K8s은 이러한 개념을 폭넓고 세밀한 단위로 사용하기 위해 <span style="color:Crimson">object</span>라는 개념을 사용합니다. 예로 container 집합(Pods), Pods을 관리하는 컨트롤러(Replica Set), 사용자(Service Account), 노드(Node) 모두를 하나의 object로 사용가능합니다. 

사용가능 한 object는 <span style="color:DodgerBlue">kubectl api-resources</span>명령어로 확인가능합니다.

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/1.png){: .mx-auto.d-block width="100%" :}


#### <span style="color:DarkOrchid">2. YAML파일을 통한 k8s 사용</span>

swarm mode의 container service를 생성하는데 사용했던 <span style="color:DodgerBlue">docker service create</span>명령어와 같이 k8s은 <span style="color:DodgerBlue">kubectl</span>을 사용합니다. swarm mode의 stack과 같이 k8s에서도 YAML파일을 사용가능하며 많이 사용합니다. YAML파일을 통해 container뿐만 아니라 거의 모든 리소스 object들에 사용될 수 있다는 장점을 가집니다. 또한 kubectl 명령어가 아닌 여러 개의 YAML파일을 정의해 서비스를 k8s에 적용시킬 수 있습니다.

#### <span style="color:DarkOrchid">3. 여러 개의 컴포넌트로 구성됨</span>

k8s에서 노드의 역할은 크게 **마스터**와 **워커**로 나뉘어져 있습니다.

- 마스터 노드: clutser를 관리
- 워커 노드: apllication container생성

swarm mode에서 단일 docker daemon만을 설치한 것과 다르게 k8s는 docker를 포함한 많은 컴포넌트들을 설치 및 실행하게 됩니다. 예로 마스터 노드에서는 **API 서버**(kube-apiserver), **컨트롤러 매니저**(kube-controller-manager), **스케줄러**(kube-scheduler), **DNS서버**(coreDNS)등이 실행되며, 모든 노드에서는 네트워크 구성을 위해 **프락시**(proxy)와 **네트워크 플러그인**(calico, flannel 등)이 설치 및 실행됩니다.

그리고 k8s clutser 구성을 위해 <span style="color:Crimson">kubelet</span>이라는 agent가 모든 노드에서 실행됩니다. kubelet은 container 생성, 삭제뿐만 아니라 마스터와 워커 노드간의 통신 역할을 함께 담당하는 agent입니다. k8s입장에서 docker daemon도 하나의 컴포넌트로 인식한다는 것도 알아두면 좋겠네요.


## 2. Pod: container를 다루는 기본 단위

### 2.1 Pod 사용

container applicaton의 기본 단위를 Pod라고 부르며 Pod는 1개 이상의 container로 구성된 container집합입니다. 예로 Nginx 웹 서비스를 k8s에서 생성하려고 다음 그림과 같이 pod 1개에 nginx 1개만을 포함하도록 생성가능합니다. 


![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/2.png){: .mx-auto.d-block width="60%" :}

이제 실제로 Nginx container로 구성된 pod을 생성해봅시다. **nginx-pod.yaml**파일에 다음 내용을 담도록 합시다.

```
# nginx-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: my-nginx-pod
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
      protocol: TCP
```

- apiVersion: YAML 파일에서 정의한 object의 API 버전
- kind: 리소스의 종류
  - 사용가능한 목록은 <span style="color:DodgerBlue">kubectl api-resources</span>명령어의 KIND항목에서 확인 가능
- metadata: 라벨, 주석, 이름과 같은 부가 정보를 입력 
  - name항목을 통해 pod의 고유 이름을 지정함
- spec: 리소스를 생성하기 위한 정보를 입력
  - containers 항목을 입력해 다음과 하위항목을 지정
  - image: docker image
  - name: container 이름
  - ports: nginx container가 사용할 port번호


이제 해당 YAML파일을 통해 <span style="color:DodgerBlue">kubectl apply -f</span>명령어를 통해 pod를 생성한다. 그리고 생성 확인을 위해 k8s에 존재하는 pod를 출력하는 <span style="color:DodgerBlue">kubectl get pods</span>명령어를 사용합니다. 

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/3.png){: .mx-auto.d-block width="60%" :}

위의 YAML에서 사용할 포트(containerPort)는 정의하였지만 아직 외부에서 접근할 수 있도록 노출 된 상태는 아닙니다. 그래서 pod의 Nginx server로 요청을 보내려면 pod container내부 ip로 접근해야합니다. 생성된 리소스의 자세한 정보를 가져올 수 있는 명령어인 <span style="color:DodgerBlue">kubectl describe</span>을 이용해 해당 pod의 ip주소를 확인합니다.

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/4.png){: .mx-auto.d-block width="60%" :}

위에서 주목해야할 부분은 2개인데요. 하나는 IP주소입니다. 저희가 [이전 글](https://da2so.github.io/2022-01-17-Docker_Kubernetes9)에서 설치했을때 설정한 k8s container의 네트워크 대역폭(172.31.0.0/16)대로 IP주소가 할당된 것을 확인가능하며 2번째는 마스터 노드에서 kubectl명령어를 사용했지만 할당된 노드는 워커 노드(worker2)인것을 확인할 수 있습니다. 

본론으로 돌아와서 위의 IP주소는 172.31.189.65인데요 이는 외부에서 접근가능한 IP가 아니기때문에 cluster 내부에서만 접근가능합니다. 외부, 내부 모두에서 pod에 접근하려면 service라고 하는 object를 생성해야하지만 지금은 IP만으로 nginx pod에 접근해보죠. worker3에서 nginx pod의 ip로 http 요청을 전송해보고 잘 다음과 같이 잘되는 지 확인합니다.

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/5.png){: .mx-auto.d-block width="80%" :}

이제 다음명령어로 worker2가 아닌 마스터 노드에서 pod container내부로 직접 들어가봅시다.

```
kubectl exec -it my-nginx-pod bash
```

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/6.png){: .mx-auto.d-block width="80%" :}


bash를 셸을 실행시키고 **-it**옵션은 셸을 유지할수 있게 해줍니다. 또한 <span style="color:DodgerBlue">kubectl logs [pod 이름]</span>을 통해 pod의 로그도 확인가능합니다.

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/7.png){: .mx-auto.d-block width="80%" :}

k8s object는 <span style="color:DodgerBlue">kubectl delete -f</span>명령어로 삭제가능합니다. 



### 2.2 pod vs docker container
 
k8s에서 container가 아닌 pod를 사용하는 이유는 container runtime의 interface 제공 등 여러가지 이유가 있지만 그 중 하나는 **여러 리눅스 네임스페이스(namespace)**을 공유하는 여러 container들을 추상화된 집합으로 사용하기 위함입니다. 예제를 위해 다음과 같이 nginx-ubuntu-pod.yaml파일을 작성해보죠.

```
#nginx-ubuntu-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx-ubuntu-pod
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
      protocol: TCP

  - name: ubuntu-container
    image: ubuntu:16.04
    command: ["tail"]
    args: ["-f", "/dev/null"] # 포드가 종료되지 않도록 유지합니다
```

전과 같이 <span style="color:DodgerBlue">kubectl apply -f</span>명령어로 해당 YAML을 k8s에 적용시켜 2개의 container가 실행중인 nginx 포드를 실행시킵니다. 

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/8.png){: .mx-auto.d-block width="100%" :}


위와 같이 container2개를 생성했으므로 READY항목의 값이 2인것을 확인가능합니다. 그리고<span style="color:DodgerBlue">kubectl exec</span>로 ubuntu container에 접속합니다. **-c** 옵션은 pod의 어떤 container에 대해 명령어를 수행할 지 명시합니다.


![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/9.png){: .mx-auto.d-block width="100%" :}

ubuntu container안에서 다음 명령어를 입력합니다.

```
# curl install
apt-get update
apt-get install curl -y

# http request to localhost
curl localhost
```
![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/10.png){: .mx-auto.d-block width="60%" :}


위와 같이 localhost로 http 요청을 했는데도 응답이 도착합니다. ubuntu container가 nginx 서버를 실행하고 있지 않는데도 말이죠. 이는 pod내의 container들이 namespace등과 같은 linux namespace을 공유하기 때문입니다. container 네트워크 타입은 네트워크 namespace를 container간에 공유해 사용할 수 있도록 설정하기 때문에 여러개의 container가 동일한 네트워크 환경을 가지게 됩니다. 

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/11.png){: .mx-auto.d-block width="60%" :}

### 2.3 완전한 application로서의 pod

실제 k8s환경에서 1개의 container로 구성된 pod를 사용하는 경우가 많습니다. 이는 하나의 pod는 하나의 완전한 application이라는 점에 그렇습니다. 그러나 nginx container가 실행되기 위해 다른 부가적인 기능이 필요할 경우에는 pod의 주 container는 nginx가 되고 기능 확장을 위한 추가 container를 함께 pod에 포함할 수 있습니다. 부가적인 container를 <span style="color:Crimson">sidecar container</span>라고 부릅니다. pod에 포함된 container들은 모두 같은 워커 노드에서 함께 실행되고 이러한 구조 및 원리에 따라 pod에 정의된 여러개의 container는 **하나의 완전한 application**으로 동작하게 됩니다.


## 3. Replica set: 일정 개수의 pod를 유지하는 controller

### 3.1 Replica set 사용 이유

다음과 같이 여러개의 동일한 container를 생성한 뒤 외부 요청이 각 container에 적절히 분배될 수 있도록 하는 마이크로서비스 구조에서 k8s는 replica set을 사용합니다.

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/12.png){: .mx-auto.d-block width="60%" :}


즉, replica set이라는 object를 통해서 다음과 같은 역할을 수행하도록합니다.

- 정해진 수의 동일한 Pod가 항상 실행되도록 관리
- 노드 장애 등의 이유로 pod를 사용할수 없다면 다른 노드에서 pod를 다시 생성함


### 3.2 Replica set 사용

nginx pod를 생성하는데 replica set을 사용해보겠습니다. 다음과 같은 내용으로 replicaset-nginx.yaml을 만들어봅시다.

```
#replicaset-nignx.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-nginx-pods-label
  template:
    metadata:
      name: my-nginx-pod
      labels: 
        app: my-nginx-pods-label
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

- spec.replicas: 동일한 pod을 몇개 유지 시킬 것인지 설정
- spec.template 아래 내용들: pod생성할 때 사용할 template정의
  - pod 생성에 사용했던 내용을 동일하게 replicaset에서도 정의하여 pod의 구성 내용을 담음

리소스의 고유한 이름은 모든 object에 설정가능하므로 replicaset의 name을 replicaset-nginx로 설정하였습니다. 위 내용의 파일로 replicaset을 만들어보죠.

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/13.png){: .mx-auto.d-block width="80%" :}

<span style="color:DodgerBlue">kubectl get po</span>와 <span style="color:DodgerBlue">kubectl get rs</span>으로 정상적으로 3개의 pod가 생성되었는지 확인합니다. po는 pods alias, rs는 replicasets의 alias로 사용됩니다.

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/14.png){: .mx-auto.d-block width="70%" :}

여기서 pod의 개수를 4개로 늘리고 싶다면 YAML파일에서 replicas의 숫자를 4로만 변경하고 다시 <span style="color:DodgerBlue">kubectl apply -f</span>명령어를 사용합니다.

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/15.png){: .mx-auto.d-block width="70%" :}


### replicaset 동작원리

pod와 replicaset은 느슨한 연결(loosely coupled)을 유지하며 이러한 느슨한 연결은 pod와 replicaset의 정의 중 <span style="color:Crimson">Label Selector)</span>를 이용해 이뤄집니다. 위에서 만든 YAML을 봐봅시다.

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/16.png){: .mx-auto.d-block width="70%" :}


위에서 replicaset영역과 pod영역에 정의된 highligt된 label은 서로 다른 object가 서로를 찾기 위해 사용됩니다. replicaset은 **spec.selector.matchLabel**에 정의된 label을 통해 생성해야하는 pod를 찾습니다. 즉, **app:my-nginx-pods-label** label을 가지는 pod의 개수가 replicas 항목에 정의된 숫자인 3개와 일치하지 않으면 pod을 정의하는 pod template항목의 내용으로 pod를 생성합니다.

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/17.png){: .mx-auto.d-block width="70%" :}

그래서 app:my-nginx-pods-label이라는 label을 가지는 pod를 미리 생성해두고 replicaset을 생성하면 어떻게 될까요? 먼저 해당 label을 가지는 pod을 수동으로 생성해보죠.

```
#nginx-label-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx-pod
  labels:
    app: my-nginx-pods-label
spec:
  containers:
  - name: my-nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/18.png){: .mx-auto.d-block width="70%" :}

이 상태에서 replicaset을 생성해보죠.

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/19.png){: .mx-auto.d-block width="70%" :}

replicaset의 selector.matchLabel에 정의된 app:my-nginx-pods-label을 가지는 label을 이미 1개(my-nginx-pod) 존재하기 때문에 template에 정의된 pod설정을 통해 3개의 pod만 생성된다. 그리고 수동으로 생성된 pod를 삭제하면 다음과 같이 replicaset이 알아서 새로운 pod를 생성해줍니다.

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/20.png){: .mx-auto.d-block width="70%" :}


만약 replicase이 생성해 놓은 pod의 label을 삭제하면 예상하셨듯이 label을 통해 replicaset 숫자를 결정하므로 app:my-nginx-pods-label이름의 label을 가지는 새로운 pod가 생성됩니다. 예시를 위해 <span style="color:DodgerBlue">kubectl edit</span>명령어을 사용하여 pod 중 하나의 label을 삭제해봅니다. label삭제는 아래 그림과 같이 label에 대한 정보를 담는 내용을 삭제하면 됩니다.

```
# replicaset-nginx-vmnrz는 pod의 이름 중 하나임
kubectl edit pods replicaset-nginx-vmnrz
```

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/21.png){: .mx-auto.d-block width="100%" :}

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/22.png){: .mx-auto.d-block width="80%" :}


edit한 부분을 저장하면 다시 pod의 목록을 보면 새로운 하나의 pod가 생성되었고 label의 정보를 삭제한 pod는 label 정보가 사라졌음을 알 수 있습니다. 그리고 label이 없는 pod는 <span style="color:DodgerBlue">kubectl delete rs</span>명령어로부터 삭제되지 않으므로 직접 삭제해주어야 합니다.

![1](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/23.png){: .mx-auto.d-block width="70%" :}




