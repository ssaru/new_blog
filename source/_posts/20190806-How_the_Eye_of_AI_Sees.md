---
title: How the Eye of A.I Sees
date: 2019-08-06 16:23:55
keywords:
- CNN
- AI
- Neural Network
- Deep learning
- shape bias
- texture bias
coverImage: cover.jpeg
thumbnailImage: cover.jpeg
autoThumbnailImage: yes
---

<!--toc-->

이번 포스팅은 논문 [ImageNet - Train CNNs are biased towards texture; increasing shape bias improves accuracy and robustness](https://openreview.net/pdf?id=Bygh9j09KX)을 읽고 읽은 내용을 포스팅한다.

# 무엇이 맞는 말일까??

## 컨볼루션 뉴럴 네트워크(Convolutional Neural Network)가 무엇에 편향되는지 바라보는 두 가지 관점

형상 정보 가설과 질감 정보 가설(Shape Hypothesis vs Texture Hypothesis)

### 형상정보 가설(Shape Hypothesis)

많은 사람들은 컨볼루션 뉴럴 네트워크(Convolutional Neural Network; 이하 CNN)가 *"레이어가 깊어질 수록 저-레벨(low level)의 특징(feature)을 복잡하게 조합해서 복잡한 모양의 특징. 즉, 고-레벨(high level) 특징(feature)을 이용하여 객체인식을 할 것이다."* 이라는 직관에 동의할 것이다.

![](layer12-de7c9aef-a43f-4314-9c83-0034c314fa4a.png)

![](layer3-78e973d7-5554-4638-a0b5-51f93ad0df7f.png)

[그림 1] ZF-Net으로 확인한 CNN의 각 layer에서 복원된 특징

CNN이 객체 분류를 어떻게 하는지에 대한 공통된 해석은 여러 문헌에서도 나타나는데 Kriegeskorte과 LeCun은 그들이 저술한 [논문](https://pdfs.semanticscholar.org/fd88/2a24391a05bf4cf92e8259101f7944d88a56.pdf) 혹은 [아티클](https://www.researchgate.net/profile/Y_Bengio/publication/277411157_Deep_Learning/links/55e0cdf908ae2fac471ccf0f/Deep-Learning.pdf)에서 아래와 같이 언급했다.

> CNN은 각 카테고리와 연관된 모양과 같은 복잡한 지식을 얻는다.  ... (중략) ... 고-레벨(high level)의 유닛들은 실제 영상(생성되거나 조작되지 않은 영상)에서 표현되는 형상(shape; 이하 형상) 정보를 배우는 것 처럼 보인다.

> CNN의 중간 레이어들은 객체들의 유사한 부분을 인식한다. ...(중략) ... 객체를 검출하는 것은 이러한 부분들의 조합이다.

이제 이렇게 CNN이 형상 정보를 학습한다는 주장을 **형상 정보 가설 (Shape Hypothesis)**부른다.

형상 가설은 수많은 실험을 통해서 지지를 받는데, 대표적으로 [ZF-Net](https://cs.nyu.edu/~fergus/papers/zeilerECCV2014.pdf)이라고 불리는 CNN이 있다. [ZF-Net](https://cs.nyu.edu/~fergus/papers/zeilerECCV2014.pdf)은 CNN의  중간 레이어로부터 특징들을 추출한다. 이렇게 추출된 특징들은 역-컨볼루션(De-Convolution)이라는 기법을 통해서 어떻게 시각적으로  보이는지 확인할 수 있다. [그림 1]은 ZF-Net에서 추출한 특징을 시각적으로 보여준다.

Kubilius라는 연구자는 CNN을 사람의 [시각 인지 모델로써 제안](https://journals.plos.org/ploscompbiol/article/file?id=10.1371/journal.pcbi.1004896&type=printable)했으며 Ritter는 [CNN이 아이들과 유사하게 형상 정보를 개발해나간다](https://arxiv.org/pdf/1706.08606.pdf)는 것을 밝혀냈다. Ritter가 이야기하는 형상 정보를 개발한다는 것의 의미는 색감(Colour) 정보 보다는 형상 정보가 객체 분류를 하는데 더 중요한 역할을 한다는 것을 의미한다. 

몰론 이에 대해서 반대되는 논문도 있다. 간단히 소개하자면 원본 영상과 색반전 영상을 같이 추론 한다고 가정하자. 만약 CNN이 형상 정보를 학습한다면 색감 정보가 반전된 영상을 같은 객체로 인식할 수 있어야한다. 하지만 그 결과는 그렇지 않았다. (해당 논문은 [여기](https://arxiv.org/pdf/1803.07739.pdf)를 참조하자)

![](Screenshot_from_2019-07-25_10-11-01-029fc274-3f82-4d21-a915-8efc1e722061.png)

[그림 2] 색감 정보 왜곡으로 인한 인식성능 저하

실제로 형상 정보 가설은 사람의 인지 체계와 매우 비슷하기도 하다. 사람은 모양이 바뀌면 다른 카테고리로 분류하지만 같은 모양에서 질감, 크기의 변화에는 영향을 받지 않는다는 것을 [연구 결과](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.331.3386&rep=rep1&type=pdf)를 통해서 확인되었다.

![](Experimental-Stimuli-A-Examples-of-stimuli-used-in-Experiment-1-Scenes-and-objects-4291356f-3c4c-46d0-8e7d-987059291c32.png.jpeg)

[그림 3] 사람은 질감과는 관계없이 모양에 따라 같은/다른 물체로 인식한다.

**Summary**

- 형상정보 가설(Shape Hypothesis)는 CNN이 객체의 거시적인 형상을 보고 인식을 한다는 가설이다.
- 가설을 지지하는 실험들이 여러 논문들을 통해서 확인되었다.

    ZF-Net, A Shape Bias Case Study, 

- 이러한 형상정보 가설은 사람의 인지체계와도 제일 유사한 측면을 갖는다.

### 질감 정보 가설(Texture Hypothesis)

몇몇 연구 결과들은 형상 정보 가설과 상반되는 결과를 얻기도 했다. 대표적으로 연결이 부분적으로 끊어진 네트워크에서 높은 인식 성능을 달성하기 위해서 훈련된 CNN 모델에서는 질감 정보(Texture; 이하 질감)가 중요한 역할을 한다는 연구가 있다.  [연구 결과](https://www.researchgate.net/profile/Alexander_Ecker/publication/319940454_Texture_and_art_with_deep_neural_networks/links/59d814150f7e9b12b3612c60/Texture-and-art-with-deep-neural-networks.pdf) CNN은 전반적인 형상 정보가 없어도 질감 정보를 이용해서 영상을 잘 분류할 수 있다. 오히려 질감 정보가 없는 형상 정보(스케치 그림)만으로 학습한 CNN은 나쁜 인식 성능을 갖는다는 연구결과가 있다.

![](Screenshot_from_2019-07-26_15-42-51-76e6b3c4-a54d-49d0-ad13-bd29cd6e7614.png)

[그림 4] CNN의 질감 정보만을 이용한 분류 예시

![](Screenshot_from_2019-07-26_15-52-57-28ac9b25-f52b-47bf-9d2d-de62245dc131.png)

![](Screenshot_from_2019-07-26_15-54-14-9bc52e1c-cf20-4470-9476-1ddee077e191.png)

[그림 5] 질감 정보가 없이 형상 정보만으로 모델을 학습해 데이터를 분류한 결과

두 가지 결과는 질감 정보와 같은 국부적인 정보만을 이용해서 ImageNet 데이터셋에서 객체 분류 문제를 충분히 해결할 수 있다는 것을 시사한다. 이를 증명한 연구결과는 [여기](https://papers.nips.cc/paper/5633-texture-synthesis-using-convolutional-neural-networks.pdf)에서 확인할 수 있다. 

그 외의 연구 결과에서는 질감 정보를 학습하고 이를 생성하는 네트워크를 만들었다. 이렇게 질감 정보를 종류별로 학습하고 생성한다는 것은 기본적으로 아래 세 가지의 기능을 수행한다는 의미를 갖는다.

1. 각 종류별 질감 정보를 분류한다.
2. 종류별 질감 정보의 통계적 분포를 학습한다.
3. 해당 분포를 생성 해낸다.

*해당 네트워크는 분류(Classification)를 하는 네트워크가 아니다. 하지만 질감 정보를 종류별로 생성한다는 것은 "질감 정보의 분포가 서로 다름", "서로 다른 질감 정보를 분류할 수 있음"을 내포한다. 이런 맥락에서 논문의 저자는 질감 정보만으로 객체 분류 문제를 풀 수 있다고 이야기하는 듯 하다.*

![](Screenshot_from_2019-07-26_16-04-33-a8313feb-2a86-42ed-b50c-be0dc81d9680.png)

[그림 6] 질감정보를 생성하는 네트워크의 결과

그 외에도 [다른 연구](https://openreview.net/pdf?id=SkfMWhAqYQ)에서는 최대 리셉티브 필드의 크기에 제한을 둔 BagNet을 설계했다. BagNet은 최대 리셉티브 필드 크기의 제한으로 CNN이 국부적인 정보만 볼수 있게끔 시야가 제한이 되었다. 그럼에도 불구하고 ImageNet 데이터에서 놀라울 정도로 높은 정확도를 갖는 결과를 얻었다. 이러한 결과들을 보았을 때 국부적인 질감 정보는 객체 분류를 하는데 충분한 정보를 가지고 있음을 알 수 있다. CNN은 이런 질감 정보을 추론하는데 이를 **질감 가설(Texture Hypothesis)**라고 부른다.

![](Screenshot_from_2019-07-26_17-17-05-c8099ac0-ddd8-4d8f-a93f-e75eaddb5ffc.png)

[그림7] 최대 리셉티브 필드의 크기가 제한된 Bag Net의 최종 Layer에서 확인된 Feature들

**Summary**

- 질감정보 가설(Texture Hypothesis)는 CNN이 객체의 국부적인 질감정보를 보고 인식을 한다는 가설이다.
- 가설을 지지하는 실험들이 여러 논문을 통해서 확인되었다.

---

지금까지 형상 정보 가설(Shape Hypothesis)와 질감 정보 가설(Texture Hypothesis)를 살펴보았다. 이렇게 상반되는 두 가지 가설을 해결하는 것은 인공지능 커뮤니티와 뇌 과학자 커뮤니티 모두에게 중요하다. 

해당 논문에서는 [StyleGAN](https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Gatys_Image_Style_Transfer_CVPR_2016_paper.pdf)을 이용하여 질감-형상 정보가 모순된 이미지(Cue Conflict)를 만들었다. 이를 이용하면 CNN과 사람의 시각 능력에 대해서 정량적으로 측정할 수 있게 된다. 이렇게 만든 질감-형상 정보가 모순된  이미지(Cue Conflict)를 이용해서 총 97명의 실험 참가자와 함께 9종류의 정신 물리학 실험(Psychophysical Experiments) 을 진행하였다. 총 실험의 횟수는 48,560번이었다.

본 논문의 기여는 아래와 같다.

- CNN과 사람의 인지 차이를 확인함
- CNN의 편향의 변경할 수 있음을 확인함
- 편향의 변경으로 생기는 효과를 확인함

# 실험

CNN과 사람의 시각 시스템의 차이를 확실하게 확인하기 위해서 해당 논문에서는 여러가지 실험을 진행했다.
<br/>
1. CNN과 사람의 시각 시스템은 서로 어떻게 다를까?(Psychophysical Experiments)
2. 데이터셋
3. 모순된 영상을 보여준다면? (Cue Conflict)
4. 실험 결과
5. CNN이 사람과 비슷하게 보게 하려면? 비슷하게 보게 된다면?

## CNN과 사람의 시각 시스템은 서로 어떻게 다를까? (Psychophysical Experiments)

CNN과 사람이 어떻게 물체를 인식하느냐(질감 정보를 기반으로 인식하느냐? vs 형상 정보를 기반으로 인식하느냐?)를 확인하기 위해서 정신 물리학(Psychophysical Experiments; 이하 정신 물리학) 실험을 진행하였다.

실험 내용은 16-Class-ImageNet이라는 데이터 셋을 사람과 CNN에게 똑같이 보여주고 이미지 분류 작업 결과를 확인하였다. 실험 참가자는 총 97명이었으며 16-Class-ImageNet의 데이터수는 49K(49,000)였다.

사람과 CNN이 본질적으로 큰 차이가 있기 때문에 실험은 굉장히 신중하게 설계 되었어야 했다. CNN은 단일의 영상 정보를 한번만 네트워크에 입력하지만 사람의 시각 시스템은 연속된 정보(동영상)을 획득하고 뇌에서는 이를 기반으로 각 신경끼리 피드백을 주고받는다. 따라서 단순하게 영상을 보여주고 분류하는 실험은 CNN과 사람에게 **단 한장의 영상만 보고 분류를 한다**라는 실험 조건에서 *단 한장*이 서로 다를 수 있다.

실험을 최대한 공평하게 진행하기 위해서는 사람과 CNN의 실험 조건을 같게 설정 해야했다. 사람의 시각 시스템에서 발생하는 피드백을 영상을 제공하는 순서와 타이밍을 이용해 최소화하였다. 이러한 방법은 이미 정신 물리학적 실험에서는 잘 알려진 사실이라고 논문에서는 언급한다. 하지만 *의학적인 참고자료가 제시되어있지 않아서 더 자세한 근거는 확인하지 못했다.*

1. 200ms 동안 분류작업을 할 영상을 보여준다.
2. 300ms 동안 고정된 사각형 이미지를 보여준다.
3. 1/f 스펙트럼을 가진 노이즈 마스크를 200ms동안 보여준다.

![](screenshot-from-2018-05-17-20-24-45-00c8ba94-f94d-422b-aa73-cc123fe604e4.png)

[[그림 8]](https://neurdiness.wordpress.com/2018/05/17/deep-convolutional-neural-networks-as-models-of-the-visual-system-qa/) CNN과 사람의 시각 시스템의 비교

## 데이터 셋

해당 연구의 실험에서는 사람과 CNN이 서로 어떤 가설을 기반으로 인지를 하는지 확인하는 것이 목적이므로 16-Class ImageNet 데이터를 왜곡하여서 진행하였다. 

- Original
- GreyScale
- Silhouette
- Edges
- Texture
- Cue Conflict (모순된 영상)

![](Screen_Shot_2019-08-05_at_12-5e21295c-52d8-4807-8d6e-87b8800647cf.44.50_AM.png)

[그림 9] 16-Class ImageNet 예시

## 모순된 영상(Cue Conflict)

기본적인 이미지 왜곡 외에도 서로 다른 형상 정보와 질감 정보가 섞여 있어 모순을 일으키는 Cue Conflict 영상으로도 실험을 진행하였다.

![](Screen_Shot_2019-08-05_at_12-f4524e3d-8c7d-469a-8bd1-2d246a21a446.50.32_AM.png)

[그림 10] Silhouettes를 이용한 Cue Conflict 영상(위에서 3번째)과 Style transfer를 이용한 Cue Conflict 영상(위에서 네 번째)

Cue Conflict 영상을 만들기 위해서 두 가지 방법을 적용하였다.

1. 세그멘테이션 맵을 이용해 다른 영상 정보를 혼합하는 방법(filled silhouettes)
2. Style GAN을 이용하여 영상 정보를 혼합하는 방법(style transfer)

연구자들이 막상 이러한 영상을 만들고 나니까 문득 든 생각이 **해당 영상들이 실제 형상 정보와 질감 정보가 완전히 모순 되어 있다는 것을 어떻게 증명할 수 있을까?** 였다. 이를 증명하기 위해서 연구자들은 앞서 언급했던 Bag Net를 사용하였다.

Bag Net는 최대 리셉티브 필드의 크기를 제한하여 CNN이 이미지의 전체 형상을 보지 못하게 만든 모델이다. 국부적인 특징(local feature)들만 이용했음에도 불구하고 BagNet은 굉장히 좋은 성능을 나타내었는데 만약 국부적인 특징(Texture)이 전체적인 특징(Shape)과 모순되어있다면(Cue conflict) Bag Net의 성능이 급격히 하락할 것이다.

실제로 Bag Net을 이용하여 Cue conflict 영상을 추론해본 결과 기존 ImageNet 데이터에서 성능이 잘 나오던 모델이 약 85% 정도의 성능 하락 발생한 것을 할 수 있었다.(ImageNet, 70.0% top-5 accuracy → Cue conflict, 10.0% top-5 accuracy)

이를 토대로 생성된 Cue conflict 영상이 형상 정보와 질감 정보가 모순되어 있다는 것을 확인할 수 있었다.

## 실험 결과

실험 결과 사람과 CNN이 물체를 분류하는 작업에서 큰 차이를 볼 수 있었다. 사람은 굉장히 형상 정보에 편향되어 있고, CNN은 질감 정보에 편향되어 있음을 확인할 수 있었다.

이는 고양이 형상에 코끼리 가죽 질감 정보가 섞여있는 Cue conflict 영상을 사람과 CNN에게 보여주었을 때, CNN은 이를 **"코끼리"**로 사람은 이를 **"고양이"**로 인식한다는 이야기가 된다.

![](Screen_Shot_2019-07-24_at_2-3f1bbca5-9002-432e-a761-3e9cf949f3dc.56.43_PM.png)

[그림 11] CNN과 사람의 시각 시스템의 차이. 왼쪽은 형상 정보 편향을 의미하고 오른쪽으로 질감 정보 편향을 의미한다. 해당 그림에서 사람은 형상 정보에 편향 되어있음을 CNN은 질감 정보에 편향 되어있음을 확인할 수 있다.

## CNN이 사람과 비슷하게 보게 하려면? 비슷하게 보게 된다면?

연구자들은 이제 **CNN이 사람과 비슷하게 형상 정보에 편향되면 어떻게 될까?**가 궁금해졌다. 그래서 기존의 **IN**(ImageNet) 데이터와 **SIN**(Sytle transfered ImageNet)데이터를 이용하여 학습을 진행하였다.

학습 방법에 대해서는 다양한 실험을 진행했다

1. IN 학습 → IN으로 평가
2. IN 학습 → SIN으로 평가
3. SIN 학습 → IN으로 평가
4. SIN 학습 → SIN으로 평가
5. IN과 SIN을 혼합해서 학습 → IN으로 평가
6. IN과 SIN을 혼합하여 학습 후 IN으로 파인튠 → IN으로 평가

학습 후에는 모델이 효과적으로 형상 정보 편향이 되었는지 확인하기 위해 16-Class-ImageNet 데이터를 이용하여 이를 확인하였다. 전부는 아니지만 많은 클래스에 대해서 CNN이 질감 정보 편향에서 형상 정보 편향으로 이동 하였음을 확인할 수 있었다.

![](Screen_Shot_2019-07-24_at_3-922d85d2-ebec-4437-9ae7-773da82ff745.51.49_PM.png)

[그림 12] IN/SIN 데이터를 학습한 모델과 사람의 차이. 데이터를 혼합해서 학습한 모델이 그림 11. 과 비교했을 때 상대적으로 형상 정보 편향으로 이동되었음을 확인할 수 있다.

결론적으로 해당 실험에서 **SIN 데이터**(형상 정보 편향)만으로는 성능을 더 개선시킬 수는 없었다. 하지만 **IN** 데이터와 **SIN**데이터를 혼합해서 학습하는 경우에는 ImageNet 데이터 셋만 이용해서 학습한 것보다는 성능이 더 좋아진다는 것을 확인하였다. 또한 형상 정보 편향이 된 CNN을 이용해 객체 검출(Object Detection) 모델에 전이 학습(Transfer learning)했을 때, 기존의 객체 검출 모델보다 성능이 더 개선 되었음을 확인할 수 있었다. 이는 **SIN**데이터를 혼합해서 학습하는 것이 모델의 일반화(Generalization)에 더 기여한다고 해석할 수 있다.

![](Screen_Shot_2019-07-24_at_3-7591f4fc-6465-4e77-a853-0821aa698abd.52.15_PM.png)

[그림 13] IN데이터와 SIN 데이터를 이용한 CNN 학습 결과

그 외에 노이즈를 이용한 영상 왜곡 실험을 추가로 진행했다. 이 경우에도 형상 정보에 편향된 CNN이 노이즈 왜곡에도 모델이 더 강인해짐을 확인할 수 있었다.

![](Screen_Shot_2019-07-24_at_5-2d3778ca-9b7d-43a9-ac91-217353e79e1e.38.52_PM.png)

[그림 14] 추가 영상 왜곡 실험에서 사용한 노이즈 예시

![](Screen_Shot_2019-07-24_at_5-edcf9de5-0d18-47aa-a2f0-abe4fcacb91a.34.05_PM.png)

[그림 15] 형상 정보, 질감 정보에 편향된 CNN이 노이즈 왜곡에 따라 나타내는 성능

# 요약

- 기존의 CNN이 이미지 분류를 할 때, 형상 정보를 기반으로 인식을 한다는 **형상 정보 가설(Shape Bias Hypothesis)**와 질감 정보를 기반으로 인식한다는 **질감 정보 가설(Texture Bias Hypothesis)**가 존재했다.

- 사람과 CNN이 왜곡된 16-Class-ImageNet 데이터를 어떤 관점에서 분류하는지 확인하기 위해 정신 물리학 실험을 수행했다. 해당 실험을 통해서 사람은 형상 정보에 편향 되어있고, CNN은 질감 정보에 편향 되어  있음을 확인할 수 있었다.

- 모델을 SIN(Style transfered ImageNet) 데이터를 ImageNet 데이터와 혼합하여 학습하게 되면 모델을 형상 정보로 편향시킬 수 있다. 형상 정보로 편향된 모델이 기존의 ImageNet 데이터로만 학습된 모델보다 더 좋은 성능을 나타냄과 동시에 전이 학습(Transfer learning)에서도 유효함을 확인할 수 있었다. 따라서 본 논문에서 제안한 방법이 인공지능 모델의 일반화(Generalization)에 기여할 수 있다.

# 참고자료

1. [IMAGENET-TRAINED CNNS ARE BIASED TOWARDS TEXTURE; INCREASING SHAPE BIAS IMPROVES ACCURACY AND ROBUSTNESS](https://arxiv.org/pdf/1811.12231.pdf)

# Thanks to

- [정미연](https://lovablebaby1015.wordpress.com/)
- [김형섭](https://www.facebook.com/profile.php?id=100024472417238)
- [권해용](https://gogyzzz.blogspot.com/)
- [김철환](https://github.com/kch8909)([markkch@naver.com](mailto:markkch@naver.com))
- [김보섭](https://aisolab.github.io/)
