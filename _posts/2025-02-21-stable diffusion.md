---
layout: article
title: "High-Resolution Image Synthesis with Latent Diffusion Models"
date: 2025-02-21 17:44 +0900
category: Paper_Review
tags: [Paper_Review]
aside:
  toc: true
sidebar:
  nav: paper_review
---
## Abstract

- Diffusion 모델은 이미지 생성 과정에 연속적으로 denoising autoencoder를 적용하여 분해함으로써, 이미지 합성에서 SOTA를 달성했다.
- 하지만 이런 diffusion모델은 일반적으로 pixel 공간에서 직접적으로 연산되기에 많은 자원이 필요하다.
- 그래서 해당 연구에서는 제한된 자원에서 quality와 flexibility를 보존하여 Diffusion 모델을 학습하기 위해 **강력한 사전 학습된 autoencoder의 latent 공간에 Diffusion 모델을 적용**하였다. 이를 통해 이전과 달리 Diffusion Model 학습에서 **복잡도 감소와 detail 보존 사이에서 한번에 거의 최적점 근처로 닿을 수 있었고, 시각적 fidelity도 향상**되었다.
- 또한 모델 구조에 **cross-attention layer를 추가**하여 text나 bounding box같은 일반적인 조건 입력에 대해서도 **더 좋고 유연한 generator로 바꾸**었고, **high-resolution 합성은 convolution 방법으로 가능**하게 했다.
- **결론적으로 Latent Diffusion Model(LDM)은 pixel based DM 모델보다 계산 요구 사항도 많이 줄고, 다양한 생성 task에 대해서 SOTA를 달성**했다.

---

## Introduction

- likelihood-based 모델은 high-resolution 합성에 있어서 지배적이나 AR transformer에서 파라미터가 많고, GAN 계열 모델은 보장된 성능을 보이지만 적대적 학습은 모델링 복잡성과 멀티 모달 분포를 확장하기 어렵기에 다룰 수 있는 데이터가 한정적이다.
- Diffusion모델은 다양한 생성 task에 대해서 SOTA를 달성했었고, likelihood-based 모델이기에, **GAN처럼 mode값 붕괴, 학습 불안정성이 나타나지 않고**, 파라미터 공유를 통해 **natural 이미지의 복잡한 분포를 AR모델에서 많은 파라미터 없이도 모델링할 수 있다.**

### Democratizing High-Resolution Image Synthesis

- 하지만 Diffusion model은 mode-covering특성으로 인해 데이터의 눈에 띄지 않는 detail을 모델링하는데 많은 컴퓨팅 자원을 소모하는 경향이 있다. 이를 위해 초기 denoising 단계에서는 샘플링을 하지 않게 하여 해결하려 했지만, **모델을 훈련하고 평가하는 데는 고차원 RGB 이미지 공간에서 반복적으로 function evaluation을 하기에 여전히 많은 컴퓨팅 자원을 소모**한다.
- 이것은 2가지 결과로 이어진다.
    1. Diffusion 모델을 훈련하는 데는 방대한 컴퓨팅 자원이 필요하지만, 이는 해당 분야의 일부만이 방대한 컴퓨팅 자원을 사용할 수 있고, 이런 방대한 컴퓨팅 자원 사용으로 인해 막대한 탄소 발자국을 남긴다. → **즉, accessibility가 낮고, 환경에도 안 좋다.**
    2. 많은 step동안 연속적으로 동일한 모델 구조를 실행하기에 시간적, 메모리적으로 비용이 크다.
- 해당 연구에서는 컴퓨팅 자원 사용량을 줄이면서도 강력한 모델의 접근성을 높이기 위해선, training과 sampling 둘 모두의 compuatational-complexity를 줄여야 한다. 그리고 **Diffusion model의 성능을 유지하면서도 computing demand를 줄이는 것이 핵심**이다.

### Departure to Latent Space

- 해당 연구에서 접근법은 우선 pixel 공간에서 사전 학습된 Diffusion 모델을 분석하는 거에서 시작한다.
- 위에서 언급했듯이 Diffusion 모델은 likelihood-based모델이고, 이 모델의 학습 단계는 크게 2가지로 구분된다.
    
    ![image.png](attachment:91aa6deb-98e8-4b69-9bab-152698fffd79:image.png)
    
    1. high-frequency detail를 없애지만, 약간의 semantic variation을 학습하는 **perceptual compression 단계**
    2. 실제 generative model이 데이터의 semantic과 conceptual composition를 학습하는 **semantic compression 단계**
- 따라서 본 논문은 고해상도 이미지 합성을 위해 **인지적으로(perceptually) 동등하지만, 계산적으로 더 안정적인 space를 찾는 것을 목표**로 한다. 이를 위해 해당 연구에서는 학습 시에 독립적은 두 단계를 거친다.
    1. 인지적으로 **data 공간과 동등하고 저차원이기에 효율적인 representation을 제공하는 auto-encoder를 학습**한다. 
    2. 1단계에서 얻어낸 latent space에서 Diffusion 모델을 학습한다.
- 여기에서 중요한 것은 **학습된 latent 공간에서 Diffusion 모델을 학습하기에 과도한 공간적 압축에 의존하지 않아도 된다**는 것이고, 또한 감소된 복잡성이 **single network pass에서 latent 공간에서 효율적인 이미지 생성이 제공**된다. → 이런 모델을 **Latent Diffusion Model(LDM)**이라 한다.
- 이 LDM의 놀라운 장점은 한 번에 전체 auto-encoder학습 할 수 있고, 이로 인해 다중 Diffusion Model 학습을 위해 재사용하거나, 완전히 다른 task도 탐색할 수 있다.
- 이는 다양한 이미지 간 및 텍스트 간 작업을 위한 많은 수의 diffusion모델을 효율적으로 탐색할 수 있게 하고, 이후에 해당 연구에서는 transformer를 Diffusion 모델의 U-Net backbone과 연결하여 임의의 유형의 토큰 기반 조건 메커니즘을 가능하게 하는 아키텍처를 설계했다.

### **LDM의 contribution**

1. transformer기반의 접근과 달리 해당 연구의 방법은 **고차원의 데이터에 대해 더 잘 확장**할 수 있고, 이에 따라 **더 신뢰할 수 있고, 디테일한 reconstruction을 제공하는 compression level에서 작동**할 수 있고, 고해상도 이미지의 **high-resolution 합성을 더 효율적으로 적용**할 수 있다.
2. pixel-based Diffusion에 비해 상당히 **감소된 연산 비용을 사용하여 다양한 task와 데이터 셋에 대해 경쟁력 있는 성능을 달성**했다.
3. encoder, decoder, score-based prior를 동시에 학습하는 이전 연구들에 비해, **reconstruction과 생성 능력에 대해 섬세한 가중치를 요구하지 않는다**. 이것은 **신뢰할 만한 reconstruction을 보장**하고 **latent 공간에서 매우 적은 규제를 요구**한다.
4. super-resolution, inpainting, semantic synthesis 같은 조건 작업에 대해 우리 모델이 **convolution 방식으로 적용될 수 있으며 약 1024 x 1024의 크고 일관된 이미지를 생성**할 수 있다.
5. Cross Attention 기반으로 general purpose conditioning 매커니즘을 설계하여 **multi modal 학습이 가능**하다.

---

## Method

![image.png](attachment:c1fa0191-1d86-4e6f-9c04-b43089df799b:image.png)

- Diffusion모델에서 high-resolution 합성에서 연산 요구량을 줄이기 위해 해당 loss term을 undersampling하여 인지적으로 부적절한 detail은 무시하였지만, 여전히 pixel 공간에서 evaluation을 진행했기에 연산 비용이 많았다.
- **생성 학습 단계에서 압축하는 단계를 명시적으로 분리**하여 위의 단점을 해결하고자 했다. 이를 통해 **연산량은 많이 감소시키, 인지적으로 데이터 공간과 동일한 공간(=latent 공간)에서 학습하는 auto-encoding모델**을 사용하여 아래와 같은 이점을 얻었다.
    1. 저차원 공간에서 샘플링이 수행되므로 계산적으로 이득을 볼 수 있다.
    2. U-Net 구조로부터 상속된 Diffusion 모델의 inductive bias를 활용하여 공간 구조를 가진 데이터에 효과적이고 이전 접근 방식과 달리 품질이 낮은 압축 수준이 필요 없다.
    3. 잠재 공간으로부터 여러 생성 모델을 학습할 때 활용할 수 있고, 여러 down stream task에 활용될 수 있는 범용 압축 모델을 얻는다.

### Perceptual Image Compression

- perceptual compression 모델은 perceptual loss와 patch-based adversarial objective의 결합으로 학습된 auto-encoder로 구성된다. 이 모델은 지역적 realism을 강화함으로써 reconstruction은 **image manifold에 제한**되고, $L_{2}$, $L_{1}$같은 **pixel 공간에서의 loss에만 의존하여 흐릿함을 피할 수 있게** 한다.
- 또한 임의로 변동성이 높 latent 공간을 피하기 위해 KL-reg와 VQ-reg라는 2가지의 규제에 대해 실험하였다. KL-reg는 VAE와 유사하게 학습된 latent 공에서 standard normal에서 약간의 KL 패널티를 부과하는 반면에 VQ-reg.는 디코더 내에서 vector quantization layer를 사용한다.
- 또한 이전 연구들에서 임의의 1차원을 따라 latent 공간(z)의 내재된 구조를 무시했지만, 해당 연구에서는 $z=E(x)$로 학습된 2 차원 구조로 설계된 연속된 Diffusion 모델로 적은 압축률을 사용하여 좋은 reconstruction을 해냈다.

### Latent Diffusion Models

- Diffusion 모델은 length가 T인 고정된  마르코프 체인의 역 과정으로 normal분포에서 점진적으로 noise를 제거해나가면서 데이터 분포 $p(x)$를 학습하는 확률 모델이다. 이런 모델은 input으로 부터 noise가 제거된 variant를 예측하면서 학습한다.
    
    ![image.png](attachment:00c62b79-4227-40f6-adc8-9abf43d7c446:image.png)
    
- $E$(Encoder)와 $D$(Decoder)로 구성되어 학습된 perceptual compression model을 통해  high-frequency, imperceptible details가 추상화 되는 latent space에 접근할 수 있다.
- 고차원 pixel 공간과 비교했을 때, latent 공간은 **중요하고 semantic한 데이터의 부분에 집중**할 수 있고, **저차원에서 학습하기에 연산적으로 더 효율적인 공간**이기에 likelihood-based모델에 더 적합한 공간이다.
- LDM에서 neural backbone $\epsilon_{\theta}$는 time-conditional U-Net으로 구현했다. 또한 forward 과정이 고정되었기에 학습동안 encoder에서 $z_{t}$를 효율적으로 얻을 수 있고, $p(z)$로부터 샘플을 하나의 Decoder를 통과하여 image 공간으로 decode할 수 있다.

### Conditioning Mechanisms

- 다른 Diffusion모델이 조건부 분포를 모델링 할 수 있는 것과 유사하게 LDM 또한 조건부 denoising auto-encoder $\epsilon_{\theta}(z_{t}, t, y)$로 적용할 수 있다. 여기에서 y는 text, semantics map 등으로 이미지 합성 과정의 조절 할 수 있게 해준다.
- 해당 연구에서는 Diffusion모델을 보다 유연한 조건부 이미지 generator로 바꾸기 위해 **U-Net backbone에 cross-attention 메커니즘을 추가**하여 다양한 modality에 대해 효율적으로 학습할 수 있게 하였다.
- 다양한 modality에서 y를 전처리하기 위해 우리는 도메인 특정 인코더 $\tau_\theta$를 도입하여 y를 중간 표현 $\tau_\theta (y) \in \mathbb{R}^{M \times d_\tau}$로 매핑한 후, cross-attention레이어를 통해 U-Net의 중간 레이어에 매핑한다.

![image.png](attachment:ab8012c7-8154-4434-a793-dbba4d180d84:image.png)

---

## Limitations

1. GAN보다 여전히 느린 속도
2. high precision이 필요한 경우 LDM의 사용이 의심스러울 수 있다.
3.  pixel space에서 미세한 정확도가 필요한 작업에는 reconstruction capability가  bottle-neck을 일으킬 수 있다. 해당 연구에서는 super-resolution model모델도 이와 관련해 이미 제한이 있다고 추정한다.

---

## Conclusion

- 성능을 저하 시키지 않고 Denosing Diffusion Model의 학습 및 샘플링 효율성을 크게 향상 시키는 간단하고 효율적인 방법인 Latent Diffusion Model(LDM)을 소개했다. 또한 여기에 Cross-**attention conditioning 메커니즘을 추가**하여, 특정 task에 특화된 구조 없이 다양한 conditional 이미지 합성 task에서 SOTA와 비교하여 좋은 성능을 보였다.