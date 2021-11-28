---
title: Connect GPU to Minikube
date: 2019-07-25 21:56:53
tags: [Minikube, GPU, Kubernetes]
categories: [Software, Cloud]
keywords:
- kubernetes
- minikube
- gpu
- cloud
coverImage: cover.jpeg
thumbnailImage: cover.jpeg
thumbnailImagePosition: left
autoThumbnailImage: yes
photos: https://images.unsplash.com/photo-1512756290469-ec264b7fbf87?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1549&q=80
---

이번 포스팅에는 Minikube에서 GPU 컨테이너 사용이 가능하도록 설정하는 방법에 대해서 포스팅한다.

<!-- excerpt -->

<!-- toc -->

# Abstract

현재 쿠버네티스(Kubernetes, 이하 쿠버네티스)를 이용하여 *딥러닝을 지원하는 마이크로 아키텍쳐(Micro Architecture, 이하 마이크로 아키텍쳐) 프레임워크 개발 및 연구*를 진행 중이다.

일반적으로 개발 및 연구 프로젝트의 원활한 진행을 위해서는 개발한 기능을 테스트하기 위한 개발 환경 구축이 필요하게 된다. 필자는 이전에 개발한 기능을 테스트하기 위해서 쿠버네티스 기반으로 작성되어있던 프레임워크를 미니쿠베(minikube, 이하 미니쿠베)에서 구동시키고 테스트하는 작업을 수행했었다.

이번에는 GPU관련 기능을 테스트하기 위해서 **GPU 컨테이너 사용이 가능하게끔 미니쿠베를 설정하는 작업**을 했으나 정확하지 않은 공식 문서와 인터넷 상에 알파, 베타 버전등 파편화된 가이드 문서로 인해 미니쿠베에서 GPU 인식이 안되어 어려움을 겪었다.

본 포스팅은 아래 세 가지를 전달하기 위해 작성되었다.

1. GPU 컨테이너 사용을 지원하는 미니쿠베 설정 방법 소개
2. 순서 별로 작업 시 해당 작업이 잘 되었는지 테스트하는 방법
3. 몇 가지 에러와 해결 방법

# Introduction

필자는 쿠버네티스를 이용하여 *딥러닝을 지원하는 마이크로 아키텍쳐 프레임워크 개발을* 하고있으며 사내에서는 서비스 혹은 테스트용으로 GPU 컨테이너 사용을 지원하는 마이크로 아키텍쳐 프레임워크가 구축되어있다.

여러 명의 개발자와 협업하여 프로젝트를 진행 하다보면 일반적으로 따르는 프로세스가 있다.

1. 개발자가 기능 개발 수행
2. 개발 환경에서 개발된 기능을 테스트하고 이상이 없음을 확인한다
3. 기능 작동 및 테스트가 완료 되면 해당 기능은 프로젝트에 병합한다

이 때, 개발자의 개발 환경 혹은 테스트 환경은 서비스 배포 환경과 유사하게끔 설정되어야한다. 필자의 프로젝트가 딥러닝을 지원하는 마이크로 아키텍쳐 프레임워크 개발이기 때문에 필자의 개발 환경은 **GPU 컨테이너를 지원하는 미니쿠베 환경**을 갖춰야 했다.

# Related Work

미니쿠베의 GPU 지원 문서$^{[1]}$를 확인해보면 미니쿠베가 GPU를 지원하는 방법은 크게 두 가지로 나뉘어진다.

1. kmv2 가상환경을 이용해서 지원 (`--vm-driver=kvm2`)
2. 호스트의 운영체제 환경을 이용해서 지원 (`--vm-driver=none`)

## kvm2 가상환경(`--vm-driver=kvm2`)

kvm2환경에서 GPU사용이 가능한 미니쿠베 설정 방법을 소개한다. 이 과정은 기존에 운영체제에 연결되었던 GPU 장치를 kvm2 가상환경에 직접적으로 연결(GPU Passthrough)하는 작업들을 포함$^{[2-9]}$하고 있기 때문에 과정이 복잡하다.

필자의 경우 해당 방법이 두 가지 사항을 요구했기 때문에 선택하지 않았다.

1. 기존 nvidia 드라이버에 연결되어있던 GPU 연결 해제
2. nvidia 그래픽 드라이버 삭제 필요

필자의 업무는 마이크로아키텍쳐 프레임워크 개발 외에도 인공지능 모델 개발도 있다. kvm2 환경에서 요구하는 두 가지 사항은 cuda와 cudnn 등 인공지능 모델 개발을 위한 환경설정을 작동하지 않도록 만든다.

필자가 kvm2환경을 사용하게 되면 업무 별로 nvidia 그래픽 드라이버 및 의존 패키지를 설치하고 삭제하는 작업을 반복적으로 수행해야한다. 이런 이유로 필자는 kvm2 환경을 선택하지 않았다.

## 호스트 운영체제 환경(`--vm-driver=none`)

필자는 공식 문서를 믿고 이를 따라서 미니쿠베 설정을 진행하였다. 적용 결과 기존에 작성된 마이크로 아키텍쳐  프레임워크에서 **GPU를 사용하는 파드(Pods)가 무한 대기(pending)에 빠지는 현상**이 발생했다.

확인 결과 미니쿠베에서 **GPU 디바이스를 인식하지 못했거나 GPU 디바이스는 인식했으나 관련 드라이버 및 패키지가 존재하지 않아서** 일어난 일이였다.

미니쿠베, 엔비디아-도커의 세부사항을 몰랐던 필자는 방대한 구글을 헤엄칠 수 밖에 없었다$^{[10-17]}$.

# Method

## Prerequisite

본 포스팅에서는 몇가지 준비 되어야하는 컴퓨터 환경과 작업이 있다. 필자는 아래의 패키지와 Ubuntu 18.04 환경에서 작업하였다.

- Docker-CE ( ≥ 18.09)

- Nvidia-Docker( ≥ 2.03)

- Nvidia GPU를 장작한 컴퓨터

- Nvidia 그래픽 드라이버 설치

- Minikube(≥ 1.2.0)

<br/>

- Nvidia-Graphics-Driver

        $ nvidia-smi
        Fri Jul 19 21:53:48 2019       
        +-----------------------------------------------------------------------------+
        | NVIDIA-SMI 390.116                Driver Version: 390.116                   |
        |-------------------------------+----------------------+----------------------+
        | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
        | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
        |===============================+======================+======================|
        |   0  GeForce GTX 108...  Off  | 00000000:01:00.0  On |                  N/A |
        | 33%   30C    P8    24W / 250W |    599MiB / 11175MiB |      1%      Default |
        +-------------------------------+----------------------+----------------------+
        |   1  GeForce GTX 108...  Off  | 00000000:02:00.0 Off |                  N/A |
        | 33%   31C    P8    16W / 250W |      2MiB / 11178MiB |      0%      Default |
        +-------------------------------+----------------------+----------------------+
      
        +-----------------------------------------------------------------------------+
        | Processes:                                                       GPU Memory |
        |  GPU       PID   Type   Process name                             Usage      |
        |=============================================================================|
        |    0      2117      G   /usr/lib/xorg/Xorg                            40MiB |
        |    0      2183      G   /usr/bin/gnome-shell                          50MiB |
        |    0      2975      G   /usr/lib/xorg/Xorg                           334MiB |
        |    0      3116      G   /usr/bin/gnome-shell                         170MiB |
        +-----------------------------------------------------------------------------+

- Docker

        $ docker version
        >>>
        Client:
         Version:           18.09.4
         API version:       1.39
         Go version:        go1.10.8
         Git commit:        d14af54266
         Built:             Wed Mar 27 18:35:44 2019
         OS/Arch:           linux/amd64
         Experimental:      false
      
        Server: Docker Engine - Community
         Engine:
          Version:          18.09.4
          API version:      1.39 (minimum version 1.12)
          Go version:       go1.10.8
          Git commit:       d14af54
          Built:            Wed Mar 27 18:01:48 2019
          OS/Arch:          linux/amd64
          Experimental:     false

- Nvidia-docker

        $ nvidia-docker version
        >>>
        NVIDIA Docker: 2.0.3
      
        ...

## Set docker default-runtime

미니쿠베에서는 기본 런타임을 Nvidia-Docker가 아닌 Docker-CE로 인식한다. GPU 사용을 위해서 도커의 기본 런타임을 Nvidia-Docker로 변경해준다.

`/etc/docker/daemon.json` 파일에서 `default-runtime`을 `nvidia`로 변경한다.

    {
        "default-runtime": "nvidia",
    
        "runtimes": {
            "nvidia": {
                "path": "nvidia-container-runtime",
                "runtimeArgs": []
            }
        }
    }

변경이 다 되었다면,  도커를 재시작한다.

    $ sudo service docker restart

## Minikube start

미니쿠베를 실행한다. 필요한 옵션에 대해서는 표에 설명해 두었다.

    $ sudo -E minikube start --vm-driver=none --apiserver-ips 127.0.0.1 --apiserver-name localhost --docker-opt default-runtime=nvidia --feature-gates=DevicePlugins=true --kubernetes-version v1.15.0
    >>>
    🟟  minikube v1.2.0 on linux (amd64)
    🟟  Creating none VM (CPUs=2, Memory=2048MB, Disk=20000MB) ...
    🟟  Configuring environment for Kubernetes v1.15.0 on Docker 18.09.4
        ▪ opt default-runtime=nvidia
        ▪ kubelet.resolv-conf=/run/systemd/resolve/resolv.conf
    🟟  Downloading kubeadm v1.15.0
    🟟  Downloading kubelet v1.15.0
    🟟  Pulling images ...
    🟟  Launching Kubernetes ... 
    🟟  Configuring local host environment ...
    
    ⚠️  The 'none' driver provides limited isolation and may reduce system security and reliability.
    ⚠️  For more information, see:
    🟟  https://github.com/kubernetes/minikube/blob/master/docs/vmdriver-none.md
    
    ⌛  Verifying: apiserver proxy etcd scheduler controller dns
    🟟  Done! kubectl is now configured to use "minikube"

| name                                | description                                                                               |
| ----------------------------------- | ----------------------------------------------------------------------------------------- |
| --docker-opt default-runtime=nvidia | 미니쿠베의 기본 도커를 엔비디아 도커로 설정한다                                                                |
| --feature-gates=DevicePlugins=true  | GPU 지원은 쿠버네티스에서 알바/베타 단계에 속한다. 따라서 이를 사용하기 위해서는 feature-gates 옵션을 이용해서 GPU 사용 옵션을 변경해줘야한다 |
| --kubernetes-version v1.15.0        | NVIDIA 드라이버를 쿠버네티스와 연결해주는 k8s-device-plugin$^{[18]}$은 1.10이상의 쿠버네티스 버전을 요구한다              |

미니쿠베 작동을 확인한다.

    $ kubectl get pods --all-namespaces
    
    NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
    kube-system   coredns-5c98db65d4-nks96               1/1     Running   0          149m
    kube-system   coredns-5c98db65d4-ns9dr               1/1     Running   0          149m
    kube-system   etcd-minikube                          1/1     Running   0          148m
    kube-system   kube-addon-manager-minikube            1/1     Running   0          148m
    kube-system   kube-apiserver-minikube                1/1     Running   0          148m
    kube-system   kube-controller-manager-minikube       1/1     Running   0          148m
    kube-system   kube-proxy-wdhfd                       1/1     Running   0          149m
    kube-system   kube-scheduler-minikube                1/1     Running   0          148m
    kube-system   storage-provisioner                    1/1     Running   0          149m

<br>

**CrashLoopBackOff Error**

만약 kube-system의 coredns에서 CrashLoopBackOff Error가 발생한다면, coredns 설정에서 Corefile안의 loop를 삭제한다.

    $ kubectl -n kube-system edit configmap coredns
    >>>
    # Please edit the object below. Lines beginning with a '#' will be ignored,
    # and an empty file will abort the edit. If an error occurs while saving this file will be
    # reopened with the relevant failures.
    #
    apiVersion: v1
    data:
      Corefile: |
        .:53 {
            errors
            health
            kubernetes cluster.local in-addr.arpa ip6.arpa {
               pods insecure
               upstream
               fallthrough in-addr.arpa ip6.arpa
               ttl 30
            }
            prometheus :9153
            forward . /etc/resolv.conf
            cache 30
            loop -> remove this line
            reload
            loadbalance
        }
    kind: ConfigMap
    metadata:
      creationTimestamp: "2019-07-19T10:34:27Z"
      name: coredns
      namespace: kube-system
      resourceVersion: "189"
      selfLink: /api/v1/namespaces/kube-system/configmaps/coredns
      uid: 8aa1d75a-0986-457f-81d6-d0339308a98a

이제 기존의 포드(Pods)를 삭제하고 새로운 설정이 적용된 파드를 생성한다.

    $ kubectl -n kube-system delete pod -l k8s-app=kube-dns

<br>

**Check GPU status**

미니쿠베의 GPU 마운트 상태를 확인한다. 지금은 GPU가 `<none>`인 것을 확인할 수가 있다.

    $ kubectl get nodes "-o=custom-columns=NAME:.metadata.name,GPU:.status.allocatable.nvidia\.com/gpu"
    >>>
    NAME       GPU
    minikube   <none>

## k8s-device-plugin

k8s-device-plugin$^{[18]}$을 미니쿠베에 적용한다. k8s-device-plugin은 미니쿠베에서 GPU 디바이스를 인식할 수 있게 해준다.

    $ kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/1.0.0-beta/nvidia-device-plugin.yml
    $ kubectl get pods --all-namespaces
    >>>
    NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
    ...
    kube-system   nvidia-device-plugin-daemonset-4xlfc   1/1     Running   0          146m
    ...

k8s-device-plugin이 Running상태로 바뀌였다면, GPU 상태를 다시 한번 확인해본다. GPU의 개수가 2개로 변경되었음을 확인할 수 있다.

    $ kubectl get nodes "-o=custom-columns=NAME:.metadata.name,GPU:.status.allocatable.nvidia\.com/gpu"
    >>>
    NAME       GPU
    minikube   2

## GPU-demo.yaml

GPU 컨테이너를 생성할 yaml파일을 생성한다. 이는 실제로 컨테이너가 미니쿠베에서 실행되었을 때, 제대로 작동하는지 확인하기 위함이다.

**GPU-demo.yaml**

    apiVersion: v1
    kind: Pod
    metadata:
      name: gpu
    spec:
    
      containers:
      - name: gpu-container
        image: nvidia/cuda:9.0-runtime
        command:
          - "/bin/sh"
          - "-c"
        args:
          - nvidia-smi && tail -f /dev/null
    
        resources:
          requests:
            nvidia.com/gpu: 2
          limits:
            nvidia.com/gpu: 2

 미니쿠베에 컨테이너를 띄워보자.

    $ kubectl apply -f GPU-demo.yaml
    >>>
    pod/gpu created
    
    $ kubectl get pods -n default
    >>>
    NAME   READY   STATUS    RESTARTS   AGE
    gpu    1/1     Running   0          157m
    
    $ kubectl logs gpu
    >>>
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 390.116                Driver Version: 390.116                   |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |===============================+======================+======================|
    |   0  GeForce GTX 108...  Off  | 00000000:01:00.0  On |                  N/A |
    | 33%   30C    P8    24W / 250W |    599MiB / 11175MiB |      1%      Default |
    +-------------------------------+----------------------+----------------------+
    |   1  GeForce GTX 108...  Off  | 00000000:02:00.0 Off |                  N/A |
    | 33%   31C    P8    16W / 250W |      2MiB / 11178MiB |      0%      Default |
    +-------------------------------+----------------------+----------------------+
    
    +-----------------------------------------------------------------------------+
    | Processes:                                                       GPU Memory |
    |  GPU       PID   Type   Process name                             Usage      |
    |=============================================================================|
    |    0      2117      G   /usr/lib/xorg/Xorg                            40MiB |
    |    0      2183      G   /usr/bin/gnome-shell                          50MiB |
    |    0      2975      G   /usr/lib/xorg/Xorg                           334MiB |
    |    0      3116      G   /usr/bin/gnome-shell                         170MiB |
    +-----------------------------------------------------------------------------+

### Access container

조금 더 확실하게 하기 위해서 미니쿠베에 배포한 컨테이너에 접속해보자. 접속 한 후에는 `nvidia-smi`와 `nvcc -V`명령어로 GPU가 잘 연결되어있는지 확인한다.

    $ kubectl exec gpu -it g -- /bin/bash
    >>>
        root@gpu:/# nvcc -V
        >>>
        nvcc: NVIDIA (R) Cuda compiler driver
        Copyright (c) 2005-2017 NVIDIA Corporation
        Built on Fri_Sep__1_21:08:03_CDT_2017
        Cuda compilation tools, release 9.0, V9.0.176
    
        root@gpu:/# nvidia-smi
        +-----------------------------------------------------------------------------+
        | NVIDIA-SMI 390.116                Driver Version: 390.116                   |
        |-------------------------------+----------------------+----------------------+
        | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
        | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
        |===============================+======================+======================|
        |   0  GeForce GTX 108...  Off  | 00000000:01:00.0  On |                  N/A |
        | 33%   30C    P8    24W / 250W |    599MiB / 11175MiB |      1%      Default |
        +-------------------------------+----------------------+----------------------+
        |   1  GeForce GTX 108...  Off  | 00000000:02:00.0 Off |                  N/A |
        | 33%   31C    P8    16W / 250W |      2MiB / 11178MiB |      0%      Default |
        +-------------------------------+----------------------+----------------------+
    
        +-----------------------------------------------------------------------------+
        | Processes:                                                       GPU Memory |
        |  GPU       PID   Type   Process name                             Usage      |
        |=============================================================================|
        |    0      2117      G   /usr/lib/xorg/Xorg                            40MiB |
        |    0      2183      G   /usr/bin/gnome-shell                          50MiB |
        |    0      2975      G   /usr/lib/xorg/Xorg                           334MiB |
        |    0      3116      G   /usr/bin/gnome-shell                         170MiB |
        +-----------------------------------------------------------------------------+

## Tip of Testing & Debugging

### Helpful command

필자는 쿠버네티스와 도커환경이 익숙하지 않아서 생각보다 많이 헤맸는데, 매 스텝 스텝 작업할 때마다 에러가 발생하면 `kubectl describe`, `kubectl logs`, `minikube logs` `lspci -nn | grep -i nvidia` 를 열심히 사용해서 디버깅을 진행하였다.

- `kubectl describe`, `kubectl logs`는 미니쿠베의 파드(Pods)의 상태 및 로그 메세지를 확인할 수 있다
- `minikube logs`는 미니쿠베 자체의 로그 정보를 확인할 수 있다
- `lspci -nn | grep -i nvidia`는 연결되어있는 디바이스 정보를 확인할 수 있다. 컨테이너에서 이를 확인하기 위해서는 별도의 Dockerfile을 작성해서 lspci util을 설치해야 컨테이너 내부에서 사용이 가능하다

### Configuration file clearning

미니쿠베로 작업을 하다가 잘못되어서 혹은 재현을 위해서 재설치를 하는 경우가 종종 있다. 이 때 기존의 설정 파일 삭제를 해줘야한다. 그렇지 않으면 이전에 설정들이 같이 따라와서 이전 작업에서 발생한 에러가 그대로 발생하는 경우가 있다. 대표적으로 아래 파일들을 삭제했는지 꼭 확인하자.

- `~/.kube`
- `~/.minikube`
- `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf`

# Reference

1. [(Experimental) NVIDIA GPU support in minikube](https://github.com/kubernetes/minikube/blob/master/docs/gpu.md)
2. [How to enable IOMMU on Ubuntu 18.04](https://www.reddit.com/r/linuxquestions/comments/bgbpim/how_to_enable_iommu_on_ubuntu_1804/)
3. [Ubuntu 18.04 - KVM/QEMU Windows 10 GPU Passthrough](https://blog.zerosector.io/2018/07/28/kvm-qemu-windows-10-gpu-passthrough/)
4. [\[GUIDE\] Linux PCI GPU VFIO Passthrough](https://linustechtips.com/main/topic/978579-guide-linux-pci-gpu-vfio-passthrough/)
5. [Testing VFIO with GPU](https://github.com/intel/nemu/wiki/Testing-VFIO-with-GPU)
6. [KVM: GPU Passthrough](https://www.server-world.info/en/note?os=Ubuntu_18.04&p=kvm&f=11)
7. [KVM 기반의 GPU Passthrough 환경](https://www.nepirity.com/blog/kvm-gpu-passthrough/)
8. [USERSPACE I/O와 VFIO](https://smallake.kr/?p=9712)
9. [Passthrough](http://iusethis.keizie.net/collection/virtualization/passthrough)
10. [0/1 nodes are available: 1 Insufficient nvidia.com/gpu](https://github.com/NVIDIA/k8s-device-plugin/issues/33)
11. [minikube - GPU support](https://github.com/kubernetes/minikube/issues/2115https://github.com/kubernetes/minikube/issues/2115)
12. [Kubeflow - Troubleshooting](https://www.kubeflow.org/docs/other-guides/troubleshooting/)
13. [Running TensorFlow Kubernetes](https://medium.com/jim-fleming/running-tensorflow-on-kubernetes-ca00d0e67539)
14. [Kubernetes - Schedule GPUs](https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/)
15. [Kubernetes - Feature Gates](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/)
16. [Kubeflow Setup](https://nvaitc.github.io/workstation-setup-guide/kubeflow-setup.html)
17. [Kubernetes에서 gpu pod 생성](https://likefree.tistory.com/15)
18. [NVIDIA device plugin for Kubernetes](https://github.com/NVIDIA/k8s-device-plugin)

# Thanks to

## 검수자

- [문동욱](https://evan-moon.github.io/)
- [정미연](https://lovablebaby1015.wordpress.com/)
- [김수정](https://github.com/soodevv)
- [김보섭](https://aisolab.github.io/)
