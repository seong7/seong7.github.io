---
layout: post
title: "NginX - Reverse Proxy 로 CORS 우회하기"
date: 2021-08-15
author: Seongjin
categories: etc
---

![Untitled](https://user-images.githubusercontent.com/52827441/163087090-649e4ffc-4651-4a56-9b28-c10098ae2130.png)

## 서론

Web Frontend 를 개발 하다보면 필연적으로 CORS 문제를 접하게 된다.

> CORS 는  Cross-Origin Resource Sharing 의 약자이며... (생략)
자세한 내용은 [여기](https://beomy.github.io/tech/browser/cors/)와 같이 다양한 블로그에서 참고할 수 있다.
> 

CORS 를 해결하기 위한 가장 정석 방법은 브라우저가 응답 받은 Data (Resource) 의 출처인 **Server** 에서 **클라이언트의  origin (Scheme + Domain + Port)** 를 Allow list에 추가해주는 방법이다.

*하지만, Frontend 개발자 입장에서 하염 없이 Backend 의 CORS 허용 업데이트를 기다리는 것은 종종 지치는 일이다.*

만약 Frontend (개발 서버 또는 로컬 등..) 에서 직접 그리고 간단하게 CORS 를 우회하는 방법을 알고 있다면 병목을 최소화 할 수 있을 것이다.

> 물론 Production 에서는 정석적인 방법으로 CORS 문제를 해결하는 것이 보안에 좋다.
> 

## Frontend 에서 CORS 를 우회하는 몇가지 방법들

1. Local 에 **Proxy Server** 띄우기
(Server → Server 통신의 경우 CORS 문제가 발생하지 않는다.)
    
    ![Untitled 1](https://user-images.githubusercontent.com/52827441/163086951-b3e1745c-edf8-49fd-a6e7-02c605fcc7b7.png)
2. **NginX 로 reverse proxy 하기**
지금부터 이 방법을 사용해 CORS 를 우회하는 법을 알아볼 것이다.

## NginX 란?

HTTP, Reverser Proxy, IMAP/POP PROXY server를 제공하는오픈소스 웹서버 프로그램이다.

~~자세한 설명은 [여기](https://m.blog.naver.com/jhc9639/220967352282)에서..~~

## Reverse Proxy 란?

proxy 에는 foward proxy 와 reverse proxy 가 있다.

- **Forward Proxy**
    
    ![Untitled 2](https://user-images.githubusercontent.com/52827441/163087069-ce9de8c0-53b0-40d5-b466-de350f713752.png)
    
    
    Proxy Server (NginX) 가 Origin Server 의 바로 앞단에서 Proxy 를 담당하는 구조이다.
    
    해당 구조에서 Client 가 NginX 로부터 받는 응답의 data 는  `xxx.com` 출처를 갖게 된다.
    

- **Reverse Proxy**
    
    ![Untitled 3](https://user-images.githubusercontent.com/52827441/163087076-7a7f5904-e08e-4775-98c3-b1c67d6492ef.png)
    
    Proxy Server (NginX) 가 Client 의 바로 뒷단에서 Proxy 를 담당하는 구조이다.
    
    해당 구조에서 Client 가 NginX 로부터 받는 응답의 data 는  `localhost:9999` 의 출처를 갖게 된다.
    

## CORS 우회하기

NginX 없이 `localhost:3000` 에서 `https://www.google.com/` 으로 Get Request 를 보내면 아래와 같이 CORS error 가 발생한다. 
([Create React App](https://create-react-app.dev/) 의 default 개발 서버인 3000 port 로 예시를 들었다.)

![Untitled 4](https://user-images.githubusercontent.com/52827441/163087079-629a8f8e-b60a-431b-b977-e1e6a4da1a21.png)

그럼 이제 Reverse Proxy 를 통해 CORS 를 어떻게 우회하는지 알아보자

### 1. NginX config 생성

`nginx.config` 파일을 생성한다.

구조는 아래와 같다.

- 80 port 를 listen port 로 열어두고,
- `/dev` path 는 `http://localhost:3000` port 로 proxy,
- `/google` path 는 `https://www.google.com/` 로 proxy 해준다.

```bash
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.html;
    }

    location /dev {
				rewrite /dev/(.*) http://localhost:3000/$1 break;
				 # proxy 될 url 에서는 ~~/dev 을 ~~/ 로 rewrite 해준다.

        proxy_set_header Host localhost;
        proxy_set_header Referer localhost;

        proxy_set_header User-Agent $http_user_agent;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Accept-Encoding "";
        proxy_set_header Accept-Language $http_accept_language;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_pass http://host.docker.internal:3000;
    }

    location /google {
         rewrite /google/(.*) /$1 break;
				 # proxy 될 url 에서는 ~~/google 을 ~~/ 로 rewrite 해준다.

         proxy_set_header Host www.google.com;
         proxy_set_header Referer https://www.google.com;

         proxy_set_header User-Agent $http_user_agent;
         proxy_set_header X-Real-IP $remote_addr;
         proxy_set_header Accept-Encoding "";
         proxy_set_header Accept-Language $http_accept_language;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

         proxy_pass https://www.google.com/;

         sub_filter /images/ https://www.google.com/images/;
         sub_filter_once off;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

### 2. Docker 로 NginX container 띄우기

아래 dockerfile 로 image 를 build 하고 container 를 9090 port 로 띄운다.

```docker
FROM nginx
COPY ./nginx.conf /etc/nginx/conf.d/default.conf
# 위에서 작성한 nginx.config 를 사용한다.
```

```bash
docker build -t reverse-proxy .

docker run --rm -it -p 9090:80 reverse-proxy
# localhost:9090 과 NginX container 의 80 port 를 mapping 한다.
```

### 3. 구조

지금까지의 구조를 도식화 하면 아래와 같다.

- 우리는 `localhost:8080/dev` 에서 React 개발 서버에 접속해 작업을 한다.
- `localhost:8080/google` 로 요청을 보내 Google server 로부터 응답을 받더라도 그 Origin 은 `localhost:8008` 이라 Same Origin 이므로 CORS 를 우회할 수 있게 된다.

![Untitled 5](https://user-images.githubusercontent.com/52827441/163087086-178ce77c-d376-4e58-8b1a-fb69622ebb86.png)

### 4. 결과

NginX reverse proxy 를 통해 `localhost:8080/dev` 에서 `localhost:8080/google`  로 Request 를 보내면 아래와 같이 CORS 를 우회하고 HTML text 를 응답 받을 수 있다.

![Untitled 6](https://user-images.githubusercontent.com/52827441/163087088-de171ee8-0511-467b-9d28-2d5b95c78dd0.png)

(React Logo 가 깨지는 문제는 static file 인 logo svg 파일을 proxy 된 host `localhost:8080` 에는 존재하지 않기 때문인데.. static file 들은 기존 host `localhost:3000` 에서 찾도록 NginX 를 설정하는 방법을 찾아봐야겠다..)

> 혹시 부족한 부분이 있다면 Comment 를 통해 지적해주시면 감사드리겠습니다. 🙏
>
