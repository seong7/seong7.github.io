---
layout: post
title: "Kubernetes 란?"
date: 2021-08-01
categories: kubernetes
---

![Untitled](https://user-images.githubusercontent.com/52827441/163086106-7de7c1d4-421b-4603-9b6f-d43d7fab01ac.png)

## Kubernetes 란,

K8s 라고도 불리는 Kubernetes 는 Container (대표적으로 Docker Container) 들을 다양한 방법으로 관리해주는 Container Orchestration 툴 중 하나이다.

[Docker](https://www.docker.com/) 사용에 익숙한 분들이라면 원하는 Base Image 위에 App 을 얹어 Image 를 구성하고 이를 실행한 Container 를 다뤄본 경험이 있을 것이다.

![Untitled 1](https://user-images.githubusercontent.com/52827441/163086088-6700f6f0-a32d-4aea-aaf3-e1ea849b2352.png)


그리고 DB + Backend + Frontend + Nginx 등의 Container 들을 하나의 서비스로서 관리하기 위해 [Docker Compose](https://docs.docker.com/compose/) 를 사용해본 경험도 있을 수 있다.

> Docker Compose 는 종속 관계가 있는 각 App 들의 Image 를 한번에 build 하고 Container 로 실행하기 용이하게 해주며, 동일한 network 대역 아래에서 각 Container 들이 통신할 수 있도록 구성해준다.
> 

![Untitled 2](https://user-images.githubusercontent.com/52827441/163086094-92768d25-e9fc-4096-aa32-a757777761c3.png)

그럼 K8s 는 Docker, Docker Compose 와 어떻게 다를까?

## Docker / Docker Compose 와의 차이점

지금까지 설명한 Docker 와 Docker Compose 의 기능들은 어느 Host 환경이든 종속되지 않고 Image 에 선언된대로 Container 의 환경을 구축해 App 을 실행할 수 있다는 점에서 개발과 배포 및 운영에서 다양한 장점을 제공해준다.

하지만, 실제 Product 를 운영하다보면 더 다양한 문제들을 만나게 된다.

예를 들어,

- Backend App 이 미처 handling 하지 못한 Exception 으로 인해 죽어 사용 불가 상태가 되는 경우
- 서버의 갑작스러운 부하 증가 대비를 위한 Over Resourcing 과 Cloud 과금 최소화 사이의 딜레마가 발생하는 경우
- Update 를 위해, old Container 들을 죽인 후 새로 생성한 new Container 들이 실행되기까지 App 을 사용할 수 없게 되는 문제

~~더 다양한 문제들이 있겠지만, 내 지식으로 설명할 수 있는 위의 세 가지에 대해서만 다뤄보겠다.~~

그럼 Kubernetes 를 사용하면 위의 문제들을 어떻게 해결할 수 있는지 알아보자.

## K8s 의 유용한 기능들

### 1. App 의 Health Check 와 Restart

K8s 에서 각각의 컴퓨터들을 Node 라고 부르는데 이 Node 내에서 각각의 App (Container) 을 감싸고 있는 instance 를 Pod 이라고 부른다.

![Untitled 3](https://user-images.githubusercontent.com/52827441/163086098-c85b1e28-203e-47a4-a656-5d8cbbdf6cd6.png)

그리고 K8s 에는 Container 들의 Liveness 와 Readiness 를 Probe (탐침) 하는 기능이 있다.

> - Liveness 는 Container 가 정상적으로 실행되고 있는지를 의미하고 Pod 을 restart 시킬지 판단하는 기준이 된다.
- Readiness 는 Container (app) 이 traffic 을 받을 수 있는 상태인가를 의미하는 용어이다.
> 

K8s 는 Liveness Probe 을 통해 어떤 App 이 handling 하지 못한 Exception 으로 인해 죽을 경우 **자동으로** 새로운 Pod 을 띄워 문제가 발생한 Pod 을 대체시킨다.

또는 Dead Lock (교착상태) 와 같은 버그가 발생했을 때도 문제를 알아차리고 위와 같은 방식으로 App 을 새로 실행시켜준다. 즉, 모든 버그에 대응할 수 있는 것은 아니지만, 어느 정도 버그로부터 발생할 수 있는 운영 상의 문제들을 피할 수 있다.

### 2. Auto Scaling 기능

K8s 에서는 Scaling 목적으로 Container 를 담고있는 Pod 을 여러개로 복제하여 운영을 할 수 있는데, 이 복제된 Pod 들을 **Replica** 라고 부른다.

Cloud 환경은 사용한 만큼 과금이 되기 때문에 변동이 많은 수요에 대응하는 운영 방식이 필요해진다.

> 만약 너무 많은 Resource 를 사용해 Pod 들을 띄우고 있다면, 과금이 너무 높아질 수 있는 위험이 있고, 너무 적은 Resource 를 사용할 경우 서버의 부하를 감당하지 못하는 Under Resourcing 문제가 발생할 수 있다.
> 

K8s 에는 Auto Scaling 기능이 있다. Min Replica 수와 Max Replica 수를 지정해두면 CPU 사용률 등을 체크하여 Pod 의 개수 (Replica 수) 를 **자동으로** 늘려주고 줄여준다. Cloud 환경에서 서비스를 운영하기에 매우 적합한 이유이다.

### 3. Rolling Update 기능

Container 들이 실행되고 있는 중, App 의 버전이 업데이트 된다고 가정해보자.

Old 버전의 Container 들을 모두 내리고 New 버전의 Container 를 띄우게 될 것 이다. 하지만, Docker Image 로 Container 를 실행시켜본다면 알겠지만, Container 를 띄우는 것은 그리 짧은 시간에 완료되는 작업이 아니다. 즉, Container 가 실행되기까지 그 짧지 않은 시간동안 우리의 서비스는 접속 불가 상태가 된다.

![Untitled 4](https://user-images.githubusercontent.com/52827441/163086102-d876f84a-26c5-4ecf-b991-013fb374e970.png)

이를 방지하기 위한 Deployment 전략으로 K8s 는 Rolling Update 기능을 **자동으로** 제공한다.

한 Pod 을 내린 후에도, 다른 Pod 에서 여전히 Traffic 을 처리할 수 있으며, 곧 바로 새로운 버전의 Container 를 담은 Pod 을 띄운다. 

이 과정을 반복하여 모든 Pod 을 새로운 Pod 으로 대체하면 위의 방법과는 달리 서비스의 접속 불가 상태를 방지할 수 있다.

![Untitled 5](https://user-images.githubusercontent.com/52827441/163086105-d77d6d92-fd67-44c8-8c28-8b49cf0eeac6.png)

> 실제로 Rolling Update 중 해당 Pod 내의 Backend App (Container) 에 API 요청을 보내보면 보낼 때마다 Old Version 과 New Version 의 응답이 번갈아가며 반환되는 것을 확인할 수 있다.
> 

## Reference

- [https://medium.com/dtevangelist/k8s-kubernetes의-hpa를-활용한-오토스케일링-auto-scaling-2fc6aca61c26](https://medium.com/dtevangelist/k8s-kubernetes%EC%9D%98-hpa%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-%EC%98%A4%ED%86%A0%EC%8A%A4%EC%BC%80%EC%9D%BC%EB%A7%81-auto-scaling-2fc6aca61c26)
- [https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-liveness-command](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-liveness-command)
