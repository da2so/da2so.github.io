---
layout: post
title: Docker/Kubernetes - (10) Kubernetes 이해 및 사용
tags: [Docker, Kubernetes]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2022-01-05-Docker_Kubernetes1/logo2.png
---

Enviroment: Ubuntu 18.04 
{: .box-note}
## 1. Kubernetes(k8s) 이해
Kubernetes(k8s)이 가지는 고유한 특성에 대해 알아봅시다.

#### <span style="color:DarkOrchid">1. 모든 리스소는 object형태로 관리됨</span>

이전 글에서 swarm mode의 container 집합을 service(서비스)라고 하였습니다. K8s은 이러한 개념을 폭넓고 세밀한 단위로 사용하기 위해 <span style="color:Crimson">object</span>라는 개념을 사용합니다. 예로 container 집합(Pods), Pods을 관리하는 컨트롤러(Replica Set), 사용자(Service Account), 노드(Node) 모두를 하나의 object로 사용가능합니다. 

사용가능 한 object는 <span style="color:DodgerBlue">kubectl api-resources</span>명령어로 확인가능합니다.

![api_resources](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/1.png){: .mx-auto.d-block width="100%" :}


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


![pod](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/2.png){: .mx-auto.d-block width="60%" :}

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

![get_pods](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/3.png){: .mx-auto.d-block width="60%" :}

위의 YAML에서 사용할 포트(containerPort)는 정의하였지만 아직 외부에서 접근할 수 있도록 노출 된 상태는 아닙니다. 그래서 pod의 Nginx server로 요청을 보내려면 pod container내부 ip로 접근해야합니다. 생성된 리소스의 자세한 정보를 가져올 수 있는 명령어인 <span style="color:DodgerBlue">kubectl describe</span>을 이용해 해당 pod의 ip주소를 확인합니다.

![kubectl_describe](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/4.png){: .mx-auto.d-block width="60%" :}

위에서 주목해야할 부분은 2개인데요. 하나는 IP주소입니다. 저희가 [이전 글](https://da2so.github.io/2022-01-17-Docker_Kubernetes9)에서 설치했을때 설정한 k8s container의 네트워크 대역폭(172.31.0.0/16)대로 IP주소가 할당된 것을 확인가능하며 2번째는 마스터 노드에서 kubectl명령어를 사용했지만 할당된 노드는 워커 노드(worker2)인것을 확인할 수 있습니다. 

본론으로 돌아와서 위의 IP주소는 172.31.189.65인데요 이는 외부에서 접근가능한 IP가 아니기때문에 cluster 내부에서만 접근가능합니다. 외부, 내부 모두에서 pod에 접근하려면 service라고 하는 object를 생성해야하지만 지금은 IP만으로 nginx pod에 접근해보죠. worker3에서 nginx pod의 ip로 http 요청을 전송해보고 잘 다음과 같이 잘되는 지 확인합니다.

![pod_check](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/5.png){: .mx-auto.d-block width="80%" :}

이제 다음명령어로 worker2가 아닌 마스터 노드에서 pod container내부로 직접 들어가봅시다.

```
kubectl exec -it my-nginx-pod bash
```

![kubectl_exec](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/6.png){: .mx-auto.d-block width="80%" :}


bash를 셸을 실행시키고 **-it**옵션은 셸을 유지할수 있게 해줍니다. 또한 <span style="color:DodgerBlue">kubectl logs [pod 이름]</span>을 통해 pod의 로그도 확인가능합니다.

![kubectl_log](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/7.png){: .mx-auto.d-block width="80%" :}

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

![kubectl_apply](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/8.png){: .mx-auto.d-block width="100%" :}


위와 같이 container2개를 생성했으므로 READY항목의 값이 2인것을 확인가능합니다. 그리고<span style="color:DodgerBlue">kubectl exec</span>로 ubuntu container에 접속합니다. **-c** 옵션은 pod의 어떤 container에 대해 명령어를 수행할 지 명시합니다.


![kubectl_exec_c_option](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/9.png){: .mx-auto.d-block width="100%" :}

ubuntu container안에서 다음 명령어를 입력합니다.

```
# curl install
apt-get update
apt-get install curl -y

# http request to localhost
curl localhost
```
![curl_localhost](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/10.png){: .mx-auto.d-block width="60%" :}


위와 같이 localhost로 http 요청을 했는데도 응답이 도착합니다. ubuntu container가 nginx 서버를 실행하고 있지 않는데도 말이죠. 이는 pod내의 container들이 namespace등과 같은 linux namespace을 공유하기 때문입니다. container 네트워크 타입은 네트워크 namespace를 container간에 공유해 사용할 수 있도록 설정하기 때문에 여러개의 container가 동일한 네트워크 환경을 가지게 됩니다. 

![network_interface](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/11.png){: .mx-auto.d-block width="50%" :}

### 2.3 완전한 application로서의 pod

실제 k8s환경에서 1개의 container로 구성된 pod를 사용하는 경우가 많습니다. 이는 하나의 pod는 하나의 완전한 application이라는 점에 그렇습니다. 그러나 nginx container가 실행되기 위해 다른 부가적인 기능이 필요할 경우에는 pod의 주 container는 nginx가 되고 기능 확장을 위한 추가 container를 함께 pod에 포함할 수 있습니다. 부가적인 container를 <span style="color:Crimson">sidecar container</span>라고 부릅니다. pod에 포함된 container들은 모두 같은 워커 노드에서 함께 실행되고 이러한 구조 및 원리에 따라 pod에 정의된 여러개의 container는 **하나의 완전한 application**으로 동작하게 됩니다.


## 3. Replica set: 일정 개수의 pod를 유지하는 controller

### 3.1 Replica set 사용 이유

다음과 같이 여러개의 동일한 container를 생성한 뒤 외부 요청이 각 container에 적절히 분배될 수 있도록 하는 마이크로서비스 구조에서 k8s는 replica set을 사용합니다.

![replicaset](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/12.png){: .mx-auto.d-block width="60%" :}


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

![kubectl_replicaset](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/13.png){: .mx-auto.d-block width="80%" :}

<span style="color:DodgerBlue">kubectl get po</span>와 <span style="color:DodgerBlue">kubectl get rs</span>으로 정상적으로 3개의 pod가 생성되었는지 확인합니다. po는 pods alias, rs는 replicasets의 alias로 사용됩니다.

![get_pods_in_replicaset](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/14.png){: .mx-auto.d-block width="70%" :}

여기서 pod의 개수를 4개로 늘리고 싶다면 YAML파일에서 replicas의 숫자를 4로만 변경하고 다시 <span style="color:DodgerBlue">kubectl apply -f</span>명령어를 사용합니다.

![scale_pods](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/15.png){: .mx-auto.d-block width="70%" :}


### 3.3 replicaset 동작원리

pod와 replicaset은 느슨한 연결(loosely coupled)을 유지하며 이러한 느슨한 연결은 pod와 replicaset의 정의 중 <span style="color:Crimson">Label Selector</span>를 이용해 이뤄집니다. 위에서 만든 YAML을 봐봅시다.

![replicaset_yaml](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/16.png){: .mx-auto.d-block width="70%" :}


위에서 replicaset영역과 pod영역에 정의된 highligt된 label은 서로 다른 object가 서로를 찾기 위해 사용됩니다. replicaset은 **spec.selector.matchLabel**에 정의된 label을 통해 생성해야하는 pod를 찾습니다. 즉, **app:my-nginx-pods-label** label을 가지는 pod의 개수가 replicas 항목에 정의된 숫자인 3개와 일치하지 않으면 pod을 정의하는 pod template항목의 내용으로 pod를 생성합니다.

![replicaset_network_interface](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/17.png){: .mx-auto.d-block width="70%" :}

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

![nginx_label_pod](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/18.png){: .mx-auto.d-block width="70%" :}

이 상태에서 replicaset을 생성해보죠.

![replicaset_nginx](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/19.png){: .mx-auto.d-block width="70%" :}

replicaset의 selector.matchLabel에 정의된 app:my-nginx-pods-label을 가지는 label을 이미 1개(my-nginx-pod) 존재하기 때문에 template에 정의된 pod설정을 통해 3개의 pod만 생성된다. 그리고 수동으로 생성된 pod를 삭제하면 다음과 같이 replicaset이 알아서 새로운 pod를 생성해줍니다.

![get_pods_nginx](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/20.png){: .mx-auto.d-block width="70%" :}


만약 replicase이 생성해 놓은 pod의 label을 삭제하면 예상하셨듯이 label을 통해 replicaset 숫자를 결정하므로 app:my-nginx-pods-label이름의 label을 가지는 새로운 pod가 생성됩니다. 예시를 위해 <span style="color:DodgerBlue">kubectl edit</span>명령어을 사용하여 pod 중 하나의 label을 삭제해봅니다. label삭제는 아래 그림과 같이 label에 대한 정보를 담는 내용을 삭제하면 됩니다.

```
# replicaset-nginx-vmnrz는 pod의 이름 중 하나임
kubectl edit pods replicaset-nginx-vmnrz
```

![delete_label](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/21.png){: .mx-auto.d-block width="100%" :}

![check_deleted_label](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/22.png){: .mx-auto.d-block width="80%" :}


edit한 부분을 저장하면 다시 pod의 목록을 보면 새로운 하나의 pod가 생성되었고 label의 정보를 삭제한 pod는 label 정보가 사라졌음을 알 수 있습니다. 그리고 label이 없는 pod는 <span style="color:DodgerBlue">kubectl delete rs</span>명령어로부터 삭제되지 않으므로 직접 삭제해주어야 합니다.

![kubectl_delete_rs](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/23.png){: .mx-auto.d-block width="70%" :}


그리고 중요한 특징 중 하나로 replicaset은 다음과 같은 YAML파일에서 표현식(matchExpressions)으로 정의가능합니다.

```
# nginx-expression-pod.yaml
...
spec:
  replicas: 3
  selector:
    matchExpressions:
      - key: app
        values:
          - my-nginx-pods-label
          - your-nginx-pods-label
        operator: In
```

위의 예시는 key가 appd인 label을 가지고 있는 pod들 중에서 values항목에 정의된 값들이 존재(In)하는 pod들 대상으로하겠다는 말으로 **my-nginx-pods-label**뿐만 아니라 **your-nginx-pods-label**이라는 label을 가진 pod또한 replicaset 관리하에 놓이게 됩니다.


## 4. Deployment: replicaset, pod의 배포를 관리


### 4.1 Deployment 사용
실제 k8s 운영환경에서 replicaset을 YAML파일에서 사용하는 경우는 없습니다. 대부분은 replicaset과 pod의 정보를 정의하는 **Deployment**라는 object를 YAML파일에 정의해 사용합니다.
Deployment는 replicaset의 상위 object이기 때문에 deployment생성시 자동으로 대응되는 replicaset도 생성됩니다. 다음과 같은 **deployment-nginx.yaml**로 deployment를 생성해봅니다. 

![deployment](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/24.png){: .mx-auto.d-block width="70%" :}


```   
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx-deployment
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
```

보시면 아시겠지만 replicaset과 비교했을 때 kind부분만 *Deployement*로 바뀌었지 다른변화는 거의 없습니다. 일단 deployment을 생성해보죠. 

![deployment_nginx](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/25.png){: .mx-auto.d-block width="70%" :}


<span style="color:DodgerBlue">kubectl get deploy</span>로 deployment의 실행을 확인하고 replicaset셋 또한 생성됨을 확인하자.


![create_deployment](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/26.png){: .mx-auto.d-block width="70%" :}

deployement로 생성하였지만 replicaset과 크게 다르지 않는 데 차이점이라고는 NAME항목에서 중간에 해시값(**6b4b7f7cdc**)이 포함되어있는데 이는 pod를 정의하는 template으로부터 생성된 것인데 자세히는 뒤에서 설명할것이므로 기억해두자. deployment삭제는 다음 명령어로 진행한다.

```
kubectl delete deploy my-nginx-deployment
```

### 4.2 Deployment 사용 이유

Deployment를 사용하는 큰 이유 중 하나는 application의 업데이트와 배포 및 관리를 해준다. 예로 application을 업데이트할 때 replicaset의 변경사항을 저장하는 revision을 남겨 rollback를 가능하게 해주고 무중단 서비스를 위해 Pod의 롤링 업데이트 전략을 지정가능하다.

Deployment을 이용해 application의 버전을 업데이트해 배포하는 예시를 알아보자. 위의 deployment-nginx.yaml을 이용해 다시 deployment을 실행하자. **--record**옵션을 통해 deployment의 변경사항을 저장하도록 한다.

```
kubectl apply -f deployment-nginx.yaml --record
```

이제 만약 당신이 nginx:1.10 을 nginx:1.11로 업데이트하고 싶다고 할때 deployment에서 생성된 pod의 image을 <span style="color:DodgerBlue">kubectl set image</span>명령어로 업데이트 가능하다.

```
kubectl set image deployment my-nginx-deployment nginx=nginx:1.11 --record
```
![record](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/27.png){: .mx-auto.d-block width="85%" :}


위에서 알 수 있듯이 기존의 사용되었던 nginx:1.10이미지를 가지는 replicaset의 해시값은 (6b4b7f7cdc)이며 기존의 replicaset의 값이 0으로 설정된 것을 보아 정지된 것을 알 수 있고 새로 nginx:1.11로 실행되는 replicaset과 그에 대한 해시값(55bbf495bd)을 확인가능하다. 그리고 이는 이전 버전의 replicaset을 삭제하지 않고 남겨두고 있는것을 말하고 이전의 정보를 리비전으로서 보존하는 것입니다. <span style="color:DodgerBlue">kubectl rollout history deploy</span>명령어로 리비전 정보를 확인하자.

![rollout](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/28.png){: .mx-auto.d-block width="75%" :}

CHANCE-CAUSE에 나오는 명령어들은 **--record**에 의해 저장된 것이며 이제 nginx:1.10으로 다시 롤백을 해보자. **--to-revision**옵션의 값으로 되돌리고자하는 revision번호의 값을 설정하면 된다.

```
kubectl rollout undo deploy my-nginx-deployment --to-revision 1
```

![rollout_undo](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/29.png){: .mx-auto.d-block width="75%" :}

롤백이 잘된것을 확인가능하다. 생성된 모든 리소스를 삭제하려면 <span style="color:DodgerBlue">kubectl delete deploy,po,rs --all</span>를 사용한다.. 정리하자면 deployment를 통해 replicaset의 리비전 관리뿐만 아니라 다양한 pod의 롤링 업데이트 정책을 사용할수 있으므로 deployment를 통해 application을 서비스하자.

## 5. Service: pod를 연결하고 외부에 노출

deployment을 실행시키기 위해 사용한 YAML에서 pod를 외부로 노출하지 않았으므로 외부에서 접근이 불가하다. Ngnix의 containerPort항목은 80번 port로 웹서버로 제공하기 때문에 설정한 값일뿐이기 때문에 해당 Nginx pod가 외부로 노출되는 것은 아닙니다. 그래서 외부에서 pod에 접근하기 위해 <span style="color:Crimson">서비스(service)</span> object를 생성해야 합니다. service의 핵심 역할은 다음과 같습니다. 

- 여러 개의 pod에 쉽게 접근할수 있도록 고유한 domain 이름을 부여
- 여러 개의 pod에 접근할 때 요청을 분산하는 로드 밸런서 기능을 수행
- 클라우드 플랫폼의 로드 밸런서, 클러스터 노드의 Port등을 통해 포드를 외부로 노출


### 5.1 service 종류

k8s 서비스는 pod에 어떻게 접근할 것이냐에 따라 종류가 여러개로 세분화 되어있는데 다음과 같은 특징을 고려하여 서비스 종류를 고르셔야합니다.

- **Clutser IP**
  - k8s 내부에서만 pod들에 접근할 때 사용함, 외부로 pod가 노출되지 않음
- **NodePoint**
  - pod에 접근할 수 있는 port를 클러스터의 모든 노드에 동일하게 개방함 외부에서 접근 가능
  - 접근 가능한 port는 랜덤으로 정해지지만 특정 Port로 접근하도록 정할 수 있음
- **LoadBalancer**
  - 클라우드 플랫폼(AWS, GCP)에서 제공하는 로드 밸런서를 동적으로 프로비저닝하여 pod에 연결, 외부에서 접근 가능
  - 실제 운영 환경에 많이 사용

#### Clutser IP

먼저 지금부터 deployment를 설명할때 사용했던 deployment-nginx.yaml과 함께 예제를 진행할 것입니다. 다음과 같이 svc-clusterip.yaml을 생성해봅시다.

```
# svc-clusterip.yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-clusterip
spec:
  ports:
    - name: web-port
      port: 8080
      targetPort: 80
  selector:
    app: my-nginx
  type: ClusterIP
```

- **spec.selector:** 해당 label을 가지는 pod에 접근하게 함
  - my-nginx는 deployment설명 때 사용한 deployment-nginx.yaml의 nginx container의 label이름
- **spec.ports.port:** k8s 내부에서만 사용할 수 있는 고유한 IP(ClusterIP)를 할당 받음
  - 서비스의 IP에 접근할 때 사용하는 port설정 값
- **spec.ports.targetPort:** selector항모게서 정의된 label에 의해 접근 대상이 된 pod들이 사용하는 내부 port번호
  - deployment-nginx.yaml의 containerPort항목의 값을 입력해야 함.
- **spec.type:** 서비스의 타입을 입력

deployment-nginx.yaml을 통해 deployment를 실행시키고 svc-clusterip.yaml로 service를 생성해보자. 그리고 <span style="color:DodgerBlue">kubectl get svc</span>명령어로 생성된 service의 목록을 확인한다.


![get_service](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/30.png){: .mx-auto.d-block width="70%" :}


svc-clutserip라는 이름으로 service를 생성하였습니다. <span style="color:DodgerBlue">kubectl run</span>명령어를 통해 임시 ubuntu pod를 만들고 출력된 CLUSTER-IP와 PORT(S)로 curl를 통한 http 요청을 보내면 응답받을 수 있습니다. 또한 service이름 자체로도 접근가능한데 이는 k8s가 application이 service나 pod를 쉽게 찾을 수 있도록 내부 DNS를 구동하고 있고 pod들은 자동으로 이 DNS을 사용된다. 

```
# run ubuntu pod and connect to it
kubectl run -i --tty --rm debug --image=ubuntu:16.04 --restart=Never -- bash

#install curl
sudo apt-get update
sudo apt-get install curl

# http request using cluster ip + port
curl 10.99.228.48:8080
# http request using inner DNS
curl svc-clusterip:8080
```

![svc_cluster](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/31.png){: .mx-auto.d-block width="50%" :}

서비스를 삭제하기 위한 명령어는 다음과 같습니다.

```
kubectl delete svc svc-clusterip
```
#### NodePort 

다음과 같은 svc-nodeport.yaml을 작성(ClusterIP와 비교했을 때 type만 다릅니다.)하고 apply시켜 service를 생성해봅니다.

```
# svc-nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-clusterip
spec:
  ports:
    - name: web-port
      port: 8080
      targetPort: 80
  selector:
    app: my-nginx
  type: NodePort
```

![nodeport](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/32.png){: .mx-auto.d-block width="70%" :}

Cluster IP와 다르게 PORT(S)항목에서 **32453**라는 숫자가 생겼고 이는 모든 노드에서 동일하게 접근 가능한 port를 의미합니다. (해당 Port는 랜덤으로 정해집니다.) 즉, 클러스터의 모든 노드에 내부 IP또는 외부 IP를 통해 32453 port로 접근하면 동일한 service에 연결가능합니다. 또한 가상머신을 아닌 제 맥북에서도 응답을 받을 수 있습니다. 당연히 nodeport를 삭제하면 연결은 끊기고 response를 받을 수 없을 것입니다.


![check_nodeport](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/33.png){: .mx-auto.d-block width="90%" :}

추가로 NodePort는 ClusterIP를 가지고 있음을 알 수 있는데 이는 NodePort는 ClusterIP의 기능을 포함하고 있기 때문이다. 다음은 NodePort의 정리 그림이다.


![nodeport_architecture](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/34.png){: .mx-auto.d-block width="100%" :}

- 외부에서 pod에 접근하기 위해 각 노드에 개방된 port로 요청을 전송함. 위 그림에서 32453 port로 들어온 요청은 service와 연결된 pod 중 하나로 라우팅됩니다.
- 클러스터 내부에는 ClusterIP의 service와 동일하게 접근할 수 있다. 

실제 운영에서는 NodePort service 그 자체를 통해 service를 외부로 제공하기 보다는 **인그레스(ingress)**라고 부르느 object에서 간접적으로 사용을 많이 합니다.

#### LoadBalancer

해당 type의 service는 로드밸러서를 동적으로 생성하는 기능을 제공하는 환경(AWS, GCP)에서만 사용가능하다. 지금 제가 하는 진행하는 가상 환경이나 on-premise에서는 사용이 힘들 수 있습니다. 개념만 알아두자면 nodeport와 유사하지만 External IP가 클라우드 플랫폼에 맞춰 설정된다는 점이 다릅니다.


#### externalTrafficPolicy: 트래픽 분배를 결정하는 service 속성

LoadBalanacer service를 사용하면 외부로부터 들어온 요청은 각 노드 중 하나로 보내지며 그 노드에서 다시 pod 중 하나로 전달됩니다. NodePort 타입을 사용했을 때도 각 노드로 들어오는 요청은 다시 pod 중 하나로 전달됩니다. 그렇지만 이러한 요청 전달 원리는 경우에 따라 효율적이지 않은데 해당 예시를 위해 다음과 같은 상황을 가정합니다.

![loadbalancer_situation](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/35.png){: .mx-auto.d-block width="70%" :}

모든 노드에서 31000번 port가 개방되어 pod에 접근할수 있으며, 워커 노드 A, B에 pod가 각각 생성되어 있다고 가정해봅시다. 이때 워커 노드 A로 들어오는 요청은 (1) A에 위치한 a pod 또는 (2) B에 위치한 b pod중 하나로 전달됩니다. 이 때 A 노드로 들어오는 요청이 굳이 a pod로 전달되지 않고 b pod로 전달된다면 네트워크 hob이 한단 계 더 발생합니다. 그리고 노드 간의 redirect가 발생하게 되어 트래픽이 출발지 주소가 바뀌는 SNAT현상이 발생하게 되고 이로 인해 client IP주소 또한 보존되지 않습니다. 


이러한 요청 전달 메커니즘은 service 속성 중 **exeternalTrafficPolicy** 항목에 정의되어있습니다. <span style="color:DodgerBlue">kubectl get -o yaml</span> 명령어로 service의 모든 속성을 출력해 보면 externalTrafficPolicy가 **Cluster**로 설정되어있는 것을 알 수 있습니다. 

![get_nodeport](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/36.png){: .mx-auto.d-block width="70%" :}

Cluster값은 default설정값으로 클러스터의 모든 노드에 랜덤한 port를 개방하는 기존 방식입니다. 다음 service YAML 파일처럼 externalTrafficPolicy를 **Local**로 설정하면 pod가 생성한 노드에서만 pod로 접근할 수 있게하며 이는 추가적인 네트워크 hob이 발생하지 않으며 전달되는 요청의 client IP또한 보존됩니다. 

```
# svc-local-nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-local-nodeport
spec:
  externalTrafficPilicy: Local
  ports:
    - name: web-port
      port: 8080
      targetPort: 80
  selector:
    app: my-nginx
  type: NodePort
```

위의 yaml로 service을 실행시키면 다음과 같은 차이점을 보이게 됩니다. 

![comparison_cluster_local](https://da2so.github.io/assets/post_img/2022-01-18-Docker_Kubernetes10/37.png){: .mx-auto.d-block width="85%" :}


#### ExternalName: 요청을 외부로 redirect하는 service

k8s를 외부 시스템과 연동해야할 때 사용하는 타입의 service입니다. External타입을 사용해 service를 생성하면 service가 외부 도메인을 가리키도록 설정가능하다. 예를 들어 아래의 설정대로 한다면 k8s 내부의 pod들이 externalname-svc라는 이름으로 요청을 보낼 경우, k8s의 DNS는 my.database.com으로 접근할 수 있도록 CNAME 레코드를 반환합니다. 즉, externalname-svc로 요청을 보내면 my.database.com에 접근하게 되는 것이다. 해당 service는 k8s와 별개로 레거시 시스템에 연동하는 경우에 사용된다.

CNAME은 Canonical Name의 약자로 도메인 주소를 또 다른 도메인 주소로 매핑 시키는 형태의 DNS 레코드 타입
{: .box-note}

```
# svc-external.yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-externalname
spec:
  type: ExternalName
  externalName: my.database.com
```
