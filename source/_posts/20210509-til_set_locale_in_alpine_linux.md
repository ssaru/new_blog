---
title: (TIL) Alpine Linux에서 locale 설정하기
date: 2021-05-09 16:14:00
tags: [Alpine Linux, Locale]
categories: [TIL]
keywords:
- Locale
- Alpine Linux
coverImage: cover.jpeg
thumbnailImagePosition: left
thumbnailImage: cover.jpeg
autoThumbnailImage: yes
photos: https://images.unsplash.com/photo-1434030216411-0b793f4b4173?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=1350&q=80
---



Alpine linux는 용량이 80MB이고, 컨테이너 이미지는 5MB밖에 안되는 초경량화된 리눅스 배포판입니다. alpine linux는 용량을 줄이기 위해 시스템의 기본 C runtime을 [glibc](https://ko.wikipedia.org/wiki/GNU_C_라이브러리) 대신 [musl libc](https://en.wikipedia.org/wiki/Musl) 를 사용하는데요. 이로 인해 제가 즐겨쓰는 ubuntu기반의 작업들이 동작하지 않는 경우가 있습니다. 그 중 ubuntu에서의 locale 명령어가 대표적입니다. ubuntu의 locale은 glibc 기반으로 구현되어있기 때문에 alpine linux에서는 `apk add locale` 명령어로는 설치할 수 없습니다. 이번 포스팅은 alpine linux에서 locale을 설정하는 방법에 대해서 다룹니다.



<!-- excerpt -->



<!--toc-->

## Installing locale in alpine Linux

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

