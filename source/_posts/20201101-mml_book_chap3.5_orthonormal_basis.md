---
title: (MML Book 선형대수 Chapter 3.5) Orthonormal Basis
date: 2020-11-01 22:10:00
tags: [Math, Linear algebra]
categories: [Math]
keywords:
- Math
- Linear algebra
coverImage: cover.jpeg
thumbnailImagePosition: left
thumbnailImage: cover.jpeg
autoThumbnailImage: yes
photos: https://images.unsplash.com/photo-1509228468518-180dd4864904?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2100&q=80
---

MML book 스터디를 진행하고있습니다. 이번 포스팅은 MML book Chapter 3.5. Orthonormal Basis에 대해서 정리한 내용입니다.

Chapter 2.6.1에서 basis vector의 속성을 살펴봤었습니다. 이 때, $n$-차원 벡터 공간에서 서로 선형 독립인 $n$개의 basis vector가 필요함을 배웠었습니다.

또한 Chapter 3.3과 3.4에서는 inner product를 벡터의 길이, 벡터 간 각도를 구하기 위해서 사용했었습니다. 이번 Chapter에서는 basis vector가 서로 orthogonal하고 basis vector의 길이가 1인 orthonormal basis basis에 대해서 이야기할 것입니다.

이 orthonormal basis의 개념은 나중에 support vector machine과 PCA를 다루는 Chapter 12, Chapter10에서 활용하게 될 것입니다.

<!-- excerpt -->



<!--toc-->

## Orthonormal Basis

**Definition 3.9** (Orthonormal Basis)

$n$차원의 벡터 공간 $V$과 $V$에서의 basis $$\{b_{i},...,b_{n}\}$$ 를 생각했을 때, $\forall\ i,j=1,...,n$ 에서 다음을 만족하면 이를 orthonormal basis(ONB)라고 부릅니다.

$$\big< b_{i}, b_{j} \big> =0, \ \ for\ i \neq j\ \ \ \ (3.33)\\ \big< b_{i}, b_{j} \big> = 1\ \ \ \ (3.34)\\ $$

만약에 (3.33)만 만족한다면 이 basis는 orthogonal basis라고 부릅니다. (3.34)는 모든 basis vector의 length나 norm이 1임을 의미합니다. 따라서 orthonormal basis라는 것은 basis가 서로 orthogonal하면서, length나 norm이 1인 basis를 말하는 것입니다.

Chapter 2.6.1의 내용을 다시 생각해보면, vector들의 집합으로부터 스팬된 vector space에서의 basis를 찾기 위해서 가우시안 소거법을 사용했었습니다. 

우리에게 non-orthogonal이고 unnormalized basis vecotr의 집합인 $$\{\tilde{b}_{1},...,\tilde{b}_{n} \}$$이 주어졌다고 생각해보십니다.

이때, orthonormal basis를 얻기 위해서는 다음과 같은 과정을 반복합니다.

1. unnormalized basis vector들의 집합들을 concatenate해서 행렬 $$\tilde{B} = [\tilde{b}_{1},...,\tilde{b}_{n}]$$ 를 만듭니다.
2. concatenate해서 만든 행렬을 augmented matrix$$(\tilde{B}\tilde{B}^{T}|\tilde{B})$$ 형태로 변경합니다.
3. augmented matrix form의 행렬을 가우시안 소거법을 적용합니다.

위와 같이 반복적인 작업을 통해서 orthonormal basis $$\{b_{1},...,b_{n}\}$$를 구하는 방법을 Gram-Schmidt process라고 부릅니다.

![Example 3.8](example3.8.png)