---
title: (번역) RabbitMQ에서 connection과 channel의 관계
date: 2021-05-14 21:16:00
tags: [RabbitMQ, Message Queue]
categories: [Message Queue, Cloud]
keywords:
- RabbitMQ
- Message Queue
coverImage: cover.jpeg
thumbnailImagePosition: left
thumbnailImage: cover.jpeg
autoThumbnailImage: yes
photos: https://images.unsplash.com/photo-1434030216411-0b793f4b4173?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=1350&q=80
---



Alpine linux는 용량이 80MB이고, 컨테이너 이미지는 5MB밖에 안되는 초경량화된 리눅스 배포판입니다. alpine linux는 용량을 줄이기 위해 시스템의 기본 C runtime을 [glibc](https://ko.wikipedia.org/wiki/GNU_C_라이브러리) 대신 [musl libc](https://en.wikipedia.org/wiki/Musl) 를 사용하는데요. 이로 인해 제가 즐겨쓰는 ubuntu기반의 작업들이 동작하지 않는 경우가 있습니다. 그 중 ubuntu에서의 locale 명령어가 대표적입니다. ubuntu의 locale은 glibc 기반으로 구현되어있기 때문에 alpine linux에서는 `apk add locale` 명령어로는 설치할 수 없습니다. 이번 포스팅은 alpine linux에서 locale을 설정하는 방법에 대해서 다룹니다.



<!-- excerpt -->



<!--toc-->

우리는 사람들과 대화를 시작할 때, 인사를 하고 가볍게 하거나 농담을 건내면서 대화를 이어갑니다. 이와 유사한 일들은 RabbitMQ에서도 일어나는데요. RabbitMQ에서는 low-level의 TCP 연결을 통해 경량 채널을 노출합니다.

RabbitMQ는 원래 RabbitMQ broker에서 지원하는 core 프로토콜인 AMQP 0.9.1을 지원하기위해 개발되었습니다. ??

## What is a connection?

TCP connection은 client와 broker를 연결하는 것으로, 이 연결 작업들은 initial authentication, IP resolution, networking와 같은 network 작업들을 포함합니다.



![connection](connection.jpg)



## What is a channel?

연결은 단일 TCP 연결을 통해 다중화 될 수 있습니다. 즉, 응용 프로그램이 단일 연결에서 "경량 연결"을 열 수 있습니다. 이 "경량 연결"을 채널이라고합니다. 각 연결은 기본 채널 세트를 유지할 수 있습니다. 



많은 응용 프로그램은 브로커에 대한 다중 연결이 필요하며, 많은 연결을 갖는 대신 응용 프로그램은 연결을 재사용하는 대신 채널을 만들고 삭제할 수 있습니다. 많은 TCP 연결을 동시에 열어 두는 것은 시스템 리소스를 소비하므로 바람직하지 않습니다. 연결을위한 핸드 셰이크 프로세스도 매우 복잡하며 TLS가 사용되는 경우 최소 7 개의 TCP 패킷이 필요합니다. 



채널은 TCP 연결 내에서 가상 연결 역할을합니다. 채널은 새로운 TCP 스트림을 재 인증하고 열 필요없이 연결을 재사용합니다. 채널을 사용하면 리소스를보다 효율적으로 사용할 수 있습니다 (이 문서의 뒷부분에서 자세히 설명). 



모든 AMQP 프로토콜 관련 작업은 채널을 통해 발생합니다. 

![channel-in-connection](channel-in-connection.jpg)



연결은 대상 서버에 대한 물리적 TCP 연결을 열어 생성됩니다. 클라이언트는 핸드 셰이크를 보내기 전에 호스트 이름을 하나 이상의 IP 주소로 확인합니다. 그런 다음 수신 서버는 클라이언트를 인증합니다. 



![amqp-tcp](amqp-tcp.jpg)



메시지를 보내거나 큐를 관리하려면 클라이언트를 통해 채널을 설정하기 전에 브로커와 연결이 생성됩니다. 채널은 메시지를 패키징하고 프로토콜 작업을 처리합니다. 클라이언트는 채널의 basic_publish 메소드를 통해 메시지를 보냅니다. queue.create 및 exchange.create와 같은 AMQP 명령이 모두 채널에서 AMQP를 통해 전송되는 것과 같이 대기열 생성 및 유지 관리도 여기에서 발생합니다. 



## Publish a message to the RabbitMQ broker

Python 라이브러리 Pika의 간단한 예를 살펴 보겠습니다. 



1. 모든 클라이언트와 마찬가지로 TCP 연결을 설정합니다. 
2. 그 후 데이터를 전송하거나 큐 생성과 같은 다른 작업을 수행하기위한 논리 채널이 생성됩니다. 브로커가 연결별로이 정보를 확인하므로 BlockingConnection을 인스턴스화 할 때 인증 정보를 제공합니다. 
3. 메시지는 채널을 통해 대기열로 라우팅됩니다. 
4. 연결이 닫힙니다 (따라서 연결에있는 모든 채널이 있음). 



```python
connection = pika.BlockingConnection(connection_parameters)
channel = connection.channel()
channel.basic_publish(exchange="my_exchange",
  routing_key="my_route",
  body= bytes("test_message")
)
connection.close()
```



## Configuring the number of channels

연결 및 채널에 대해 운영자 제한을 사용하는 것이 좋습니다. 연결에서 허용되는 최대 채널 수를 구성하려면 channel_max를 사용하십시오. 이 변수는 새 구성 형식의 rabbit.channel_max에 해당합니다. 이 제한을 초과하면 치명적인 오류가 발생합니다. 허용되는 최대 연결 수를 구성하려면 connections_max를 사용하십시오. 



우리가 얻는 일반적인 질문은 RabbitMQ 연결 당 보유해야하는 채널 수 또는 최적의 채널 수입니다. 항상 설정에 따라 다르기 때문에 대답하기가 어렵습니다. 이상적으로는 각 새 스레드에 지정된 전용 채널을 사용하여 프로세스 당 하나의 연결을 설정해야합니다. 



channel_max를 0으로 설정하면 "무제한"을 의미합니다. 애플리케이션에 때때로 채널 누출이 있기 때문에 이것은 위험한 움직임 일 수 있습니다. 



## Avoiding connection and channel leaks

두 가지 일반적인 사용자 실수는 클라이언트가 수백만 개의 연결 / 채널을 열 때 메모리 문제로 인해 RabbitMQ가 중단되는 채널 및 연결 누수입니다. 이러한 문제를 조기에 발견 할 수 있도록 CloudAMQP는 활성화 할 수있는 경보를 제공합니다. 



종종 채널 또는 연결 누수는 완료되었을 때 닫히지 못한 결과입니다. 



![connection-alarms](connection-alarms.png)



## Recommendations for connections and channels

다음은 연결 및 채널을 사용하지 않고 사용하는 방법에 대한 몇 가지 권장 사항을 따릅니다. 



### Use long-lived connection

각 채널은 연결에 비해 클라이언트에서 상대적으로 적은 양의 메모리를 사용합니다. 연결이 너무 많으면 RabbitMQ 서버 메모리 사용에 큰 부담이 될 수 있습니다. 오래 지속되는 연결을 유지하고 필요한 경우 채널을 더 자주 열고 닫으십시오. 



각 프로세스는 하나의 TCP 연결 만 만들고 다른 스레드에 대해 해당 연결에서 여러 채널을 사용하는 것이 좋습니다. 



### Separate the connections for publishers and consumers

게시 용으로 하나 이상의 연결을 사용하고 각 앱 / 서비스 / 프로세스에 대해 하나를 사용합니다. 



RabbitMQ는 게시자가 서버에서 처리 할 너무 많은 메시지를 보낼 때 TCP 연결에 역압을 적용 할 수 있습니다. 동일한 TCP 연결을 사용하는 경우 서버가 클라이언트로부터 메시지 확인을받지 못하여 소비자 성능에 영향을 미칠 수 있습니다. 소비 속도가 낮 으면 서버가 압도됩니다. 



### Don’t share channels between threads

애플리케이션에서 스레드 당 하나의 채널을 사용하고 대부분의 클라이언트가 채널을 스레드로부터 안전하게 만들지 않으므로 스레드간에 채널을 공유하지 않도록합니다. 



CloudAMQP를 사용하면 수요를 충족하도록 인스턴스를 확장하는 동시에 누수 문제를 해결하는 메커니즘을 제공 할 수 있습니다.







애플리케이션이 다국어를 지원할 수 있도록 docker image에 다른 locale을 설치하는 것이 필요할 수 있습니다. 올바른 locale이 설치되지 않는다면 텍스트 데이터가 올바르게 표현되지 않는 문제를 겪을 수 있습니다. alpine docker image는 `locale`, `locale-gen` 명령어가 존재하지 않으며, `apk add locale`로 설치를 시도할 경우 locale이라는 패키지가 없다는 에러를 확인할 수 있습니다. alpine에서 locale을 사용하기 위해서는 alpine과 호환되는 locale 패키지를 별도로 설치해줘야합니다. 

(해당 포스팅에서는 명령어는 dockerfile을 기준으로 설명합니다.)



```dockerfile
FROM alpine:3.6

# ---not shown here---

# Install language pack
RUN apk --no-cache add ca-certificates wget && \
    wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://raw.githubusercontent.com/sgerrand/alpine-pkg-glibc/master/sgerrand.rsa.pub && \
    wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.25-r0/glibc-2.25-r0.apk && \
    wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.25-r0/glibc-bin-2.25-r0.apk && \
    wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.25-r0/glibc-i18n-2.25-r0.apk && \
    apk add glibc-bin-2.25-r0.apk glibc-i18n-2.25-r0.apk glibc-2.25-r0.apk

# install ko_KR locale
# Note that locale -a is not available in alpine linux, use `/usr/glibc-compat/bin/locale -a` instead
RUN /usr/glibc-compat/bin/localedef -i ko_KR -f UTF-8 ko_KR.UTF-8

RUN /usr/glibc-compat/bin/localedef -i ko_KR -f UTF-8 ko_KR.UTF-8 && \
    echo "export LC_ALL='ko_KR.UTF-8'" >> /etc/profile && \
    echo "export LANG='ko_KR.UTF-8'" >> /etc/profile && \
    echo "export LANGUAGE='ko_KR.UTF-8'" >> /etc/profile
ENV LC_ALL=ko_KR.UTF-8

# --- not show here---
```



### install locale package

```dockerfile
RUN apk --no-cache add ca-certificates wget && \
    wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://raw.githubusercontent.com/sgerrand/alpine-pkg-glibc/master/sgerrand.rsa.pub && \
    wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.25-r0/glibc-2.25-r0.apk && \
    wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.25-r0/glibc-bin-2.25-r0.apk && \
    wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.25-r0/glibc-i18n-2.25-r0.apk && \
    apk add glibc-bin-2.25-r0.apk glibc-i18n-2.25-r0.apk glibc-2.25-r0.apk
```

위 명령어는 alpine linux를 지원하는 `locale` 패키지를 설치합니다. 해당 명령어를 실행하고나면, `/usr/glibc-compat/bin` 디렉토리에서 locale관련 명령어인 `locale`과 `localedef`를 확인할 수 있습니다. 



### compile locale

`locale`관련 패키지를 설치하고나면, 현재 시스템이 사용중인 locale과 사용 가능한 locale을 `/usr/glibc-compat/bin/locale` 명령어와 `/usr/glibc-compat/bin/locale -a`로 확인할 수 있습니다. 이를 확인해보면, 내가 사용하고자하는 `ko_KR.UTF-8` 이 없음을 확인할 수 있습니다. 

locale을 정의한 파일들은 `/usr/glibc-compat/share/i18n/locales` 폴더 아래에 있고, charmap(캐릭터맵)에 대한 정보는 `/usr/glibc-compat/share/i18n/charmaps` 폴더 아래에 있습니다. 이 두 가지 정보가 `localedef`의 명령어로 컴파일 되며 컴파일을 하게되면 `/usr/glibc-compat/lib/locale` 폴더에 `locale-archive`라는 접두사(prefix)를 가진 파일이 생성되게 됩니다.

`ko_KR.UTF-8` locale을 사용하기 위해서 아래 명령어로 locale을 컴파일 합니다.

```dockerfile
RUN /usr/glibc-compat/bin/localedef -i ko_KR -f UTF-8 ko_KR.UTF-8
```



### Set locale

이제 locale 관련 패키지도 설치하였고 필요한 locale도 컴파일 해줬으니, 환경변수에 locale을 설정해봅시다.

```dockerfile
RUN /usr/glibc-compat/bin/localedef -i ko_KR -f UTF-8 ko_KR.UTF-8 && \
    echo "export LC_ALL='ko_KR.UTF-8'" >> /etc/profile && \
    echo "export LANG='ko_KR.UTF-8'" >> /etc/profile && \
    echo "export LANGUAGE='ko_KR.UTF-8'" >> /etc/profile
ENV LC_ALL=ko_KR.UTF-8
```

ubuntu는 기본적으로 bash shell을 사용하기 때문에, 환경변수를 로그인 쉘의 경우에는 `~/.bashrc`에 미 로그인 쉘에는 `/etc/profile`에 설정합니다. alpine은 bash shell이 아닌 ash를 제공하는 busybox를 기본 shell로 사용합니다. 따라서 환경변수 설정은 로그인 쉘의 경우 `~/.profile`, 미 로그인 쉘은 `/etc/profile`에 설정해야합니다.

(저의 경우는 미로그인 쉘을 사용했으므로 환경변수를 `/etc/profile`에 설정하였습니다.)



## Check locale setting

설정을 완료하고나면, `/usr/glibc-compat/bin/locale`와 `/usr/glibc-compat/bin/locale -a`를 통해 locale이 `ko_KR.UTF-8`로 잘 설정되었음을 확인할 수 있습니다.

```bash
$ /usr/glibc-compat/bin/locale
>>
LANG=ko_KR.UTF-8
LC_CTYPE="ko_KR.UTF-8"
LC_NUMERIC="ko_KR.UTF-8"
LC_TIME="ko_KR.UTF-8"
LC_COLLATE="ko_KR.UTF-8"
LC_MONETARY="ko_KR.UTF-8"
LC_MESSAGES="ko_KR.UTF-8"
LC_PAPER="ko_KR.UTF-8"
LC_NAME="ko_KR.UTF-8"
LC_ADDRESS="ko_KR.UTF-8"
LC_TELEPHONE="ko_KR.UTF-8"
LC_MEASUREMENT="ko_KR.UTF-8"
LC_IDENTIFICATION="ko_KR.UTF-8"
LC_ALL=ko_KR.UTF-8

$ /usr/glibc-compat/bin/locale -a
>>
C
POSIX
ko_KR.utf8
```



## Reference

1. [Setting up locale on alpine:3.6 docker image](https://gist.github.com/alextanhongpin/aa55c082a47b9a1b0060a12d85ae7923)
2. [컨테이너 접속시 locale 에러 해결](https://palyoung.tistory.com/12)
3. [Where to set system default environment variables in Alpine linux?](https://stackoverflow.com/questions/35325856/where-to-set-system-default-environment-variables-in-alpine-linux)

