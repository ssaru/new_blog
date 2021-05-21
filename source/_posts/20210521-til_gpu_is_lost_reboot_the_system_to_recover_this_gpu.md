---
title: (TIL) GPU is lost. Reboot the system to recover this GPU Error잡기
date: 2021-05-21 22:49:00
tags: [GPU, RTX-3090]
categories: [TIL]
keywords:
- GPU
- Error
coverImage: cover.jpeg
thumbnailImagePosition: left
thumbnailImage: cover.jpeg
autoThumbnailImage: yes
photos: https://images.unsplash.com/photo-1434030216411-0b793f4b4173?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=1350&q=80
---





최근 셋팅한 제 회사 컴퓨터에는 RTX-3090이 장착되어있습니다. 제 컴퓨터에서 딥러닝 모델을 학습하다보면 매번 학습 중에  `GPU is lost. Reboot the system to recover this GPU` 에러를 출력하면서 학습이 중단되었습니다. 이럴 때마다 매번 컴퓨터를 리부팅해야하는 상황이 반복되어 매우 불편했는데요. 이 에러에 대해서 제가 어떻게 작업했는지 공유합니다.



<!-- excerpt -->



<!--toc-->

## GPU is lost

최근 셋팅한 제 회사 컴퓨터에는 RTX-3090이 장착되어있습니다. RTX-3090을 장착한 김에 딥러닝 모델의 Hyperparameter search를 제 컴퓨터에서 수행하였는데요. 매번 프로그램을 실행할 때마다 `GPU is lost`라는 에러를 출력하고 프로그램이 중단되곤했습니다. 이 에러가 나오면 더 이상 OS에서 GPU에 접근할 수 없기 때문에 매번 OS를 재부팅해야하는 번거로움이 있었습니다.



```bash
$ python3 train.py
...
(중략)
...
Unable to determine the device handle for GPU 0000:05:00.0: GPU is lost. Reboot the system to recover this GPU
```



## What is problem?

이 에러를 마주하고나서는 제가 세운 가설은 다음과 같았습니다.

1. 파워 서플라이의 용량이 컴퓨터의 전력 수요를 맞출 수 없다. 혹은 GPU의 발열이 너무 심하다.
2. GPU가 PCI 슬롯에 제대로 장착되지 않았다.
3. 하드웨어 호환성 혹은 하드웨어에 문제가 있을 것이다.



### Insufficient power

컴퓨터에 장착된 파워 서플라이의 용량이 GPU가 장착된 컴퓨터의 전력 수요를 맞출 수 없다면, `nvidia-smi`에서 GPU가 사용하는 전력량을 제한할 수 있습니다. 

GPU의 발열이 심한 경우에도 GPU가 사용하는 전력량을 제한하여 온도상승을 제한할 수 있습니다.

GPU가 사용할 수 있는 전력량을 제한하는 명령어는 아래와 같습니다.

```bash
# -pm --persistence-mode=
# Set persistence mode: 0/DISABLED, 1/ENABLED
$ sudo nvidia-smi -pm 1
>>
Enabled persistence mode for GPU 00000000:02:00.0.
All done.


# -pl --power-limit
# Specifies maximum power management limit in watts.
$ sudo nvidia-smi -pl 290
>>
Power limit for GPU 00000000:02:00.0 was set to 290.00 W from 390.00 W.
All done.
```

먼저 `nvidia-smi` 명령어를 통해서 nvidia 커널 모듈이 지정된 GPU에 대해서 항상 활성화 상태가 되도록 강제합니다. nvidia 커널 모듈은 그래픽 GPU가 작동하기 위해 필요한데, X윈도우 상태이거나 CUDA 애플리케이션이 작동할 때에 커널 모듈이 활성화됩니다. X윈도우가 종료되거나 CUDA 애플리케이션이 더 이상 작동하지 않는 경우에는 커널 모듈이 비활성화 상태가 됩니다.

`nvidia-smi` 명령어를 통한 전력제한 옵션은 nvidia 커널 모듈이 활성화되어있을 때만 적용됩니다. 즉, nvidia 커널 모듈이 활성화된 상태에서 전력제한이 적용되지만 비활성화된 상태에서는 전력제한이 풀려서 원래의 값으로 복원되게 됩니다. 전력제한이 동적으로 적용되는 것을 방지하기 위해서 persistence mode를 적용하여 nvidia 커널 모듈을 활성합니다.



RTX-3090에 전력제한을 하고, GPU와 CPU의 온도를 모니터링하면서 같은 에러가 발생하는지 테스트하였습니다.

GPU가 허용하는 온도는 아래의 명령어를 확인할 수 있습니다.

```bash
$ nvidia-smi -q | grep -i temp
>>
    Temperature
        GPU Current Temp                  : 67 C
        GPU Shutdown Temp                 : 98 C
        GPU Slowdown Temp                 : 95 C
        GPU Max Operating Temp            : 93 C
        GPU Target Temperature            : 83 C
        Memory Current Temp               : N/A
        Memory Max Operating Temp         : N/A
```

GPU가 Shutdown되는 온도는 98도임을 확인할 수 있고, 권장 최대 온도는 83도임을 확인할 수 있습니다.



CPU 의 온도를 모니터링하기 위해서는 [lm-sensor](https://askubuntu.com/questions/15832/how-do-i-get-the-cpu-temperature)를 이용하였습니다.

```bash
$ watch -n 0.1 sensor
>>
Every 0.1s: sensors                                 ketimartin3090: Fri May 21 23:39:24 2021

iwlwifi_1-virtual-0
Adapter: Virtual device
temp1:            N/A

pch_cometlake-virtual-0
Adapter: Virtual device
temp1:        +46.0°C

acpitz-acpi-0
Adapter: ACPI interface
temp1:        +16.8°C  (crit = +20.8°C)
temp2:        +27.8°C  (crit = +119.0°C)

coretemp-isa-0000
Adapter: ISA adapter
Package id 0:  +65.0°C  (high = +100.0°C, crit = +100.0°C)
Core 0:        +49.0°C  (high = +100.0°C, crit = +100.0°C)
Core 1:        +47.0°C  (high = +100.0°C, crit = +100.0°C)
Core 2:        +65.0°C  (high = +100.0°C, crit = +100.0°C)
Core 3:        +45.0°C  (high = +100.0°C, crit = +100.0°C)
Core 4:        +47.0°C  (high = +100.0°C, crit = +100.0°C)
Core 5:        +47.0°C  (high = +100.0°C, crit = +100.0°C)
Core 6:        +43.0°C  (high = +100.0°C, crit = +100.0°C)
Core 7:        +45.0°C  (high = +100.0°C, crit = +100.0°C)

nvme-pci-0400
Adapter: PCI adapter
Composite:    +54.9°C  (low  = -273.1°C, high = +84.8°C)
                       (crit = +84.8°C)
Sensor 1:     +54.9°C  (low  = -273.1°C, high = +65261.8°C)
Sensor 2:     +78.8°C  (low  = -273.1°C, high = +65261.8°C)
```



학습을 돌려놓고, 확인해본 결과 전력제한을 걸어 전력소모와 발열량을 줄였음에도 같은 에러가 발생하는 것을 확인하였습니다.



### GPU inserted not properly in PCIE slot

RTX-3090은 지금까지 나왔던 GPU 중에 제일 크고 무게가 있는 GPU입니다. 메인보드가 세로로 세워져있는 컴퓨터 케이스에 RTX-3090을 장착하게되면 PCIE slot에 장착된 GPU가 무게 때문에 내려앉습니다. 이로 인해 GPU가 잘 장착되지 않거나 메인보드에 PCIE 슬롯이 망가질 수 있습니다. 이를 방지하기 위해서 GPU 지지대를 사용하고있습니다만, 혹시 모르는 마음에 GPU를 탈착하고 재 장착하였습니다.



GPU를 재장착하고 테스트한 결과 같은 에러가 발생하는 것을 확인할 수 있었습니다.



### Hardware compatibility

문제에 대해서 계속 검색하다가 리눅스 부팅 메세지를 확인하는 것을 통해서 GPU의 상태를 추적할 수 있다는 사실을 알았습니다. 부팅메세지는 `dmesg`명령어를 통해서 확인할 수 있었는데, 여기서 몇가지 키워드들을 확인할 수 있었습니다.

```bash
$ dmesg
...
(중략)
[ 189.427290] NVRM: Xid (PCI:0000:01:00): 79, GPU has fallen off the bus.
...
(중략)
PCIE Bus Error: Severity=Corrected after booting into Ubuntu
...
```



저는 이 키워드들에 집중했고, 모종의 하드웨어 호환 이슈로 인해 발생하는 PCIE Bus Error의 문제는 매우 흔하고, 이를 kernel parameter를 수정하여 해결할 수 있다는 것을 확인하였습니다. 



우분투는 `/etc/default/grub`에서 이를 설정할 수 있습니다. 해당 파일을 열어 아래의 내용을 추가해줍니다.

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash pci=nomsi"
```

해당 내용을 추가하였다면, `sudo update-grub`으로 수정된 내용을 반영해주고 컴퓨터를 재부팅하면 됩니다.



저는 최종적으로 위 내용을 반영하고나서, 위의 에러가 다시 발생하지 않는다는 것을 확인하였습니다.



## Reference

1. [Unable to determine the device handle for GPU :GPU is lost](https://forums.developer.nvidia.com/t/unable-to-determine-the-device-handle-for-gpu-gpu-is-lost/57641)
2. [Xid 79, GPU has fallen off the bus.](https://forums.developer.nvidia.com/t/xid-79-gpu-has-fallen-off-the-bus/49452)
3. [GPU has fallen off the bus (Nvidia)](https://askubuntu.com/questions/868321/gpu-has-fallen-off-the-bus-nvidia)
4. [How do i get the CPU temperature](https://askubuntu.com/questions/15832/how-do-i-get-the-cpu-temperature)
5. [nvidia smi 꼭 알아야 할 TIP - temp,serial,topology matrix](https://kyumdoctor.co.kr/11)
6. [Troubleshooting PCIe Bus Error severity Corrected on Ubuntu and Linux Mint](https://itsfoss.com/pcie-bus-error-severity-corrected/)
7. [NVIDIA 우분투 마이닝 FAQ](https://www.ddengle.com/board_FAQ/2719454)

