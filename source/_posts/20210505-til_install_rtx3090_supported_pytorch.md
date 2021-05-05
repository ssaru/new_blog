---
title: (TIL) RTX 3090을 지원하는 PyTorch 버전설치
date: 2021-05-05 22:27:00
tags: [PyTorch, RTX3090]
categories: [TIL]
keywords:
- PyTorch
- RTX3090
coverImage: cover.jpeg
thumbnailImagePosition: left
thumbnailImage: cover.jpeg
autoThumbnailImage: yes
photos: https://images.unsplash.com/photo-1434030216411-0b793f4b4173?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=1350&q=80
---



2021.05.05 현재 RTX3090은 CUDA11 이상을 지원하는 딥러닝 프레임워크에 버전에서만 사용할 수 있습니다. 하지만 단순하게 `pip install torch==1.7.1 torchvision==0.8.2` 형태로 설치하면 `CUDA error: no kernel image is available for execution on the device` 에러를 마주할 수 있습니다. 이 때에는 반드시 `pip install torch==1.7.1+cu110 torchvision==0.8.2+cu110 -f https://download.pytorch.org/whl/torch_stable.html `형태로 설치해주어야합니다.



<!-- excerpt -->



<!--toc-->

## Installing PyTorch with RTX 3090 support

2021.05.05 현재 RTX3090은 CUDA11 이상을 지원하는 딥러닝 프레임워크에 버전에서만 사용할 수 있습니다. 하지만 단순하게 `pip install torch==1.7.1 torchvision==0.8.2` 형태로 설치하면 `CUDA error: no kernel image is available for execution on the device` 에러를 마주할 수 있습니다.

```bash
$ python3 -m pip install torch==1.7.1 torchvision==0.8.2
Collecting torch==1.7.1
  Using cached torch-1.7.1-cp38-cp38-manylinux1_x86_64.whl (776.8 MB)
Collecting torchvision==0.8.2
  Using cached torchvision-0.8.2-cp38-cp38-manylinux1_x86_64.whl (12.8 MB)
Requirement already satisfied: numpy in /home/ubuntu/.pyenv/versions/3.8.10/lib/python3.8/site-packages (from torch==1.7.1) (1.20.2)
Requirement already satisfied: typing-extensions in /home/ubuntu/.pyenv/versions/3.8.10/lib/python3.8/site-packages (from torch==1.7.1) (3.10.0.0)
Requirement already satisfied: pillow>=4.1.1 in /home/ubuntu/.pyenv/versions/3.8.10/lib/python3.8/site-packages (from torchvision==0.8.2) (8.2.0)
Installing collected packages: torch, torchvision
Successfully installed torch-1.7.1 torchvision-0.8.2

$ python3
Python 3.8.10 (default, May  4 2021, 22:52:00)
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import torch
>>> torch.cuda.is_available()
True
>>> torch.randn(3,3).to("cuda:0")
/home/ubuntu/.pyenv/versions/3.8.10/lib/python3.8/site-packages/torch/cuda/__init__.py:104: UserWarning:
GeForce RTX 3090 with CUDA capability sm_86 is not compatible with the current PyTorch installation.
The current PyTorch install supports CUDA capabilities sm_37 sm_50 sm_60 sm_70 sm_75.
If you want to use the GeForce RTX 3090 GPU with PyTorch, please check the instructions at https://pytorch.org/get-started/locally/

  warnings.warn(incompatible_device_warn.format(device_name, capability, " ".join(arch_list), device_name))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/home/ubuntu/.pyenv/versions/3.8.10/lib/python3.8/site-packages/torch/tensor.py", line 179, in __repr__
    return torch._tensor_str._str(self)
  File "/home/ubuntu/.pyenv/versions/3.8.10/lib/python3.8/site-packages/torch/_tensor_str.py", line 372, in _str
    return _str_intern(self)
  File "/home/ubuntu/.pyenv/versions/3.8.10/lib/python3.8/site-packages/torch/_tensor_str.py", line 352, in _str_intern
    tensor_str = _tensor_str(self, indent)
  File "/home/ubuntu/.pyenv/versions/3.8.10/lib/python3.8/site-packages/torch/_tensor_str.py", line 241, in _tensor_str
    formatter = _Formatter(get_summarized_data(self) if summarize else self)
  File "/home/ubuntu/.pyenv/versions/3.8.10/lib/python3.8/site-packages/torch/_tensor_str.py", line 89, in __init__
    nonzero_finite_vals = torch.masked_select(tensor_view, torch.isfinite(tensor_view) & tensor_view.ne(0))
RuntimeError: CUDA error: no kernel image is available for execution on the device
```



이 때에는 반드시 `pip install torch==1.7.1+cu110 torchvision==0.8.2+cu110 -f https://download.pytorch.org/whl/torch_stable.html `형태로 설치해주어야합니다.

(RTX3090을 지원하는 PyTorch는 python3.8이상을 요구한다고 하여 해당 예제는 python3.8에서 테스트하였습니다)

```bash
$ python3 -m pip install torch==1.7.1+cu110 torchvision==0.8.2+cu110 -f https://download.pytorch.org/whl/torch_stable.html 
python3 -m pip install torch==1.7.1+cu110 torchvision==0.8.2+cu110 -f https://download.pytorch.org/whl/torch_stable.html
Looking in links: https://download.pytorch.org/whl/torch_stable.html
Collecting torch==1.7.1+cu110
  Using cached https://download.pytorch.org/whl/cu110/torch-1.7.1%2Bcu110-cp38-cp38-linux_x86_64.whl (1156.8 MB)
Collecting torchvision==0.8.2+cu110
  Using cached https://download.pytorch.org/whl/cu110/torchvision-0.8.2%2Bcu110-cp38-cp38-linux_x86_64.whl (12.9 MB)
Requirement already satisfied: numpy in /home/ubuntu/.pyenv/versions/3.8.10/lib/python3.8/site-packages (from torch==1.7.1+cu110) (1.20.2)
Requirement already satisfied: typing-extensions in /home/ubuntu/.pyenv/versions/3.8.10/lib/python3.8/site-packages (from torch==1.7.1+cu110) (3.10.0.0)
Requirement already satisfied: pillow>=4.1.1 in /home/ubuntu/.pyenv/versions/3.8.10/lib/python3.8/site-packages (from torchvision==0.8.2+cu110) (8.2.0)
Installing collected packages: torch, torchvision
Successfully installed torch-1.7.1+cu110 torchvision-0.8.2+cu110

$ python3
Python 3.8.10 (default, May  4 2021, 22:52:00)
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import torch
>>> torch.cuda.is_available()
True
>>> torch.randn(3,3).to("cuda:0")
tensor([[-0.5210,  0.2223,  1.7470],
        [ 0.5932,  0.3055,  0.8472],
        [-0.1898, -1.0950,  1.6593]], device='cuda:0')
```





