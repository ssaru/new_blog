---
title: (MML Book 선형대수 Chapter 3.4) Angles and Orthogonality
date: 2020-11-01 21:34:00
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

MML book 스터디를 진행하고있습니다. 이번 포스팅은 MML book Chapter 3.4. Angles and Orthogonality에 대해서 정리한 내용입니다.

Angle과 Orthogonality가 무엇인지에 대해서 살펴봅니다.

<!-- excerpt -->



<!--toc-->

## Angle

inner product는 벡터의 length나, 두 벡터 간의 distance를 정의하게 하는 것 외에도 두 벡터 간의 각도 $\omega$를 정의할 수 있습니다.

이때, 두 벡터 $x, y$ 사이의 inner product space에서 각도 $\omega$를 정의하기 위해서 우리는 Cauchy-Schwarz inequality (3.17)를 사용합니다. 

>  Cauchy-Schwarz Inequality
> $$|\big< x,y \big>| \leqslant ||x||\ ||y||\ \ \ \ (3.17)$$

​        

$x \neq 0, y \neq0$을 가정할 때, Angle은 다음과 같습니다.

$$-1 \leqslant \frac{\big<x,y\big>}{||x||\ ||y||} \leqslant 1\ \ \ \ (3.24)$$

1. Cauchy-Schwarz Inequality에서 좌항의 절대값을 제거

    $$-||x||\ ||y|| \leqslant \big< x,y \big> \leqslant ||x||\ ||y||$$ 

2. 모든 항을 $||x||\ ||y||$ 로 나눔

   $$-1 \leqslant \frac{\big<x,y\big>}{||x||\ ||y||} \leqslant 1\ \ \ \ (3.24)$$

​    

이때,  $\omega \in [0, \pi]$는 다음과 같이 정의되며, Figure 3.4과 같이 표현할 수 있습니다.

$$cos \omega = \frac{\big< x,y \big>}{||x||\ ||y||}\ \ \ \ (3.25)$$

![Figure3.4](figure3.4.png)

​     

이때, $\omega$는 두 벡터 $x, y$간의 각도입니다. 각도는 직관적으로 두 벡터의 방향성이 얼마나 일치하는지에 대해서 알려줍니다. 예를들어 inner product가 dot product이고 벡터 $x, y=4x$가 있을 때, $y$는 $x$가 스케일된 벡터이므로 Angle은 0입니다.  Angle이 0이라면 방향이 같다는 의미가 됩니다.



![Example3.6](example3.6.png)

​    

##Orthogonality 

length, distance, angle을 유도하는 것 외에 inner product의 핵심 특징은 두 벡터가 orthogonal인지 아닌지에 대한  알 수 있게 해준다는 것입니다.

​    

**Definition 3.7** (Orthogonality)

만약 $\big< x, y \big> =0$이라면, 두 벡터 $x,y$는 orthogonal이며 우리는 이를 $x \perp y$라고 적습니다. 추가적으로 $||x||=1=||y||$라면 벡터는 unit vector라는 속성도 추가되어 orthonormal하다라고 이야기합니다.

이러한 정의가 함축하는 것은 $0$-벡터는 벡터 공간에서 모든 벡터와 orthogonal하다는 것을 의미합니다.

​    

**Remark**

Orthogonality는 inner product에 대한(bilinear forms) 수직(perpendicularity) 개념의 일반화입니다.

기하학적인 맥락에서는 두 벡터가 orthogonal이라면 두 벡터가 서로 직각인 벡터로써 생각할 수 있습니다.



![Example3.7_1](example3.7_1.png)

![Example3.7_2](example3.7_2.png)

​    

**Definition 3.8** (Orthogonal Matrix)

만약에 모든 column들 끼리 모두 orthonormal하다면 square matrix $A \in \mathbb{R}^{n \times n}$은 orthogonal matrix입니다.

$$AA^{T} = I = A^{T}A\ \ \ \ (3.29)$$

그리고, 이는 아래의 식과 같은 관계를 갖습니다.

$$A^{-1} = A^{T}\ \ \ \ (3.30)$$

즉, 역행렬을 transpose로 간단히 구할 수 있다는 의미가 됩니다.

​    

orthogonal transformation matrix $A$를 이용한 transformation($Ax$)은 벡터 $x$의 length가 변하지 않는다는 특성이 있습니다. inner product를 dot product라고 생각하면, $||Ax||^{2}$ 은 $||x||^{2}$ 와 같습니다.

$$||Ax||^{2} = (Ax)^{T}(Ax) = x^{T}A^{T}Ax = x^{T}Ix = x^{T}x = ||x||^{2}\ \ \ \ (3.31)$$

​    

거기에 두 벡터 $x, y$사이의 각도 또한 inner product로써 측정되므로 orthogonal matrix $A$를 이용해 transformation시 각도 또한 변하지 않습니다. dot product를 inner product로 가정하고 orthogonal matrix$A$ 가 있을 때, $Ax, Ay$ 간의 각도는 두 벡터 $x, y$ 와 같음을 확인할 수 있습니다.

$$cos \omega = \frac{(Ax)^{T}(Ay)}{||Ax||\ ||Ay||} = \frac{x^{T}A^{T}Ay}{\sqrt{x^{T}A^{T}Axy^{T}A^{T}Ay}} = \frac{x^{T}y}{||x||\ ||y||}\ \ \ \ (3.32)$$