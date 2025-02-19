---
layout: article
title: "DreamFusion: Text-To-3D Using 2D Diffusion"
date: 2025-02-19 20:39 +0900
category: Paper_Review
tags: [Paper_Review]
aside:
  toc: true
sidebar:
  nav: paper_review
---
## Abstract

![스크린샷 2025-02-19 203402](https://github.com/user-attachments/assets/8d054fd9-6147-4dc7-a662-4b9baee62635)

- 최근 수십억개의 image-text쌍으로 학습된 text-to-image 합성을 위한 Diffusion모델이 발전해왔다. 이것을 3D에도 적용하고자 했지만, **라벨링된 대규모의 3D 데이터셋과 효율적인 모델 구조가 없었다**.
- 해당 연구에서는 이런 한계를 극복하기 위해 사전 훈련된 2D Text-to-Image Diffusion 모델을 사용하여 Text-3D 합성을 수행하고자 한다.
- Probability density distillation에 기반한 loss를 도입하여, 2D 확산 모델을 매개변수화된 image generator의 최적화를 위한 prior로 사용할 수 있게 한다.
- 이 loss를 사용함으로써, 3D모델을 최적화하여 랜덤한 각도로부터 **2D Rendering시에 낮은 loss**를 달성했고, 모**든 각도에서 볼 수 있으며, 임의의 조명으로 재조명하거나 어떤 3D 환경에 통합**하였다.
- 이러한 방법은 **3D데이터 셋과 image diffusion 모델의 변경 없이 가능**하다.

---

## Introduction

- Text 기반 image 생성모델은 high-fidelity, 다양성과 control가능한 이미지 합성을 지원한다. 이러한 quality 향상은 **대규모 aligned text-image 데이터셋과 확장 가능한 생성 모델 구조** 덕분에 가능하다.
- 이 중 Diffusion 모델은 안정적이고 확장 가능한 denoising objective를 가진 고품질 image generator를 학습하는 데 특히 효과적이다. 또한 다른 modality에 대해서도 성공적이지만 **modality에 특화된 대규모의 학습 데이터가 필요**하다.
- 그래서 해당 연구에서는 **3D 데이터 없이, 사전 학습된 2D image-text diffusion모델을 변환하여 3D 객체를 합성**하는 기술을 개발했다.
- image 생성이 넓게 적용 가능하지만, simulator와 비디오 게임과 영화 같은 digital media는 많은 3D 자원이 필요하다. 이 3D자원은 수작업으로 디자인 되기에 많은 시간과 전문 지식이 필요하다.
- 기존의 3D 생성모델
    - voxel이나 point cloud는 구조의 explicit 표현을 기반으로 학습 가능하지만 필요한 3D 데이터는 풍부한 2D 이미지에 비해 상대적으로 부족하다.
    - GAN은 control가능한 3D generator를 학습할 수 있지만, 특정 도메인만 가능하다.
    - Dream Field의 경우에는 CLIP의 image-text joint embedding을 사용하고, NeRF를 학습하기 위해 optimization-based 접근 방식을 채택하여 2D image-text 모델이 3D 합성에도 활용될 수 있음을 보였다.
- DreamFusion의 경우, Dream Field와 유사하지만 CLIP 대신에 2D Diffusion 모델의 distillation으로부터 나온 loss를 사용한다.
- 이 loss는 probability density distillation을 기반으로 하고, diffusion의 forward process를 기반으로 하여 **shared mean을 가진 가우시안 분포와 사전 학습된 diffusion 모델로 학습된 score function의 KL-Divergence를 최소화**하는 방향으로 학습한다.
- 결과적으로 **Score Distillation Sampling(SDS)**는 ****미분 가능한 이미지 파라미터화에서 최적화를 통한 샘플링이 가능하고 NeRF 변형과 결합하여 DreamFusion은 다양한 사용자가 제공한 text-prompt에 대해 high-fidelity로 일관된 3D객체와 장면을 생성한다.

---

## Diffusion Models And Score Distillation Sampling

### Diffusion Model

- Diffusion 모델은 샘플을 생성할 때, **노이즈를 점진적으로 제거**하며 데이터를 복원하는 방식의 생성 모델이다.
1.  **Forward Process($q$)**
    - 원본 데이터 x 에 점진적으로 가우시안 노이즈를 추가하여 구조를 제거하는 과정
    - 이를 통해 점점 더 **랜덤한 분포**로 변환됨
    - 수학적으로, t 번째 단계의 노이즈 분포는 아래처럼 정의함
    - $q(z_{t}∣x)=N(α_{t}x,σ_{t}^{2}I)$

1. **Reverse Process($p$)**
    - 모델이 데이터 분포를 학습한 후, 노이즈에서 데이터를 복원하는 과정
    - 샘플링을 통해 점진적으로 노이즈를 제거함
    - **Reverse** 과정은 최적의 MSE 기반 노이즈 예측기를 학습하는 방식으로 이루어짐
    - $p_{\theta}(z_{t-1}|z_{t}) = q(z_{t}|z_{t}, x=x_{\theta}(z_{t};t))$

### How can we sample in parameter space, no pixel space?

- Diffusion 모델은 모델이 학습한 데이터와 같은 타입과 차원을 가진 데이터만 샘플링한다. 또한 전통적으로 pixel(이미지)을 주로 샘플링하여 왔다. 하지만 우리는 3D를 생성하는 것이 목표이기에 pixel에는 관심없다.
- 우리가 관심을 가지는 것은 **랜덤한 각도에서 rendering할 때, 좋은 이미지처럼 보이는 3D모델을 만드는 것이 목표**이다.
- 이러한 모델을 미분 가능한 generator $g$가 이미지 $x=g(\theta)$를 생성하기 위한 파라미터 $\theta$를 반환하는 **differentiable image parameter(DIP)**로 세분화 될 수 있다. 3D를 기준으로 봤을 때, **$\theta$는 3D Volume, g는 volumetric render**로 볼 수 있다. 이 파라미터($\theta$)를 학습하기 위해 우리는 diffusion 모델에 적용할 수 있는 loss function이 필요하다.
- 해당 논문의 접근 방식은  $x = g(\theta)$가 frozen diffusion model로부터 sampling된 것처럼 보이는 방향으로 최적화를 통해, tractable한 샘플링을 가능하게 한다.
- 이 최적화를 위해 필요한 loss : 그럴듯한 것은 loss가 작고, 그럴듯 하지 않은 것은 loss가 큰 것(= 학습된 조건부 밀도$p(x|y)$의 mode를 찾는 loss)
- 첫 번째로 Diffusion의 loss를 사용하였을 때, 고차원의 생성 모델의 mode가 일반적이지 않을 때, diffusion 모델 학습의 단계적인(multiscale) 성질이 이 병리를 피하는데 도움을 줄 수 있다.
- diffusion training loss로 생성된 데이터포인트 $x = g(\theta)$에 대해 최소화하면 $\theta^*={arg min}_{\theta} L{_{Diff}}(\phi, x = g(\theta))$가 된다.
- 하지만 실제로, 우리는 이 손실 함수가 $x = \theta$인 항등 DIP를 사용할 때조차도 현실감 있는 샘플을 생성하지 않음을 발견했고, 다른 연구에서는 이 방법이 신중하게 timestep 스케쥴러를 만들 수 있다는 것을 보였지만 해당 연구에서는 objective를 다루기 힘들고, timestep 스케쥴도 다루기 어렵다는 것을 찾았다. → U-Net Jacobian term이 연산량이 많기 때문에

![스크린샷 2025-02-18 184915](https://github.com/user-attachments/assets/52d5d95c-4748-4ed8-a7d7-8725644815e6)

- 직관적으로 이 loss는 t에 따라 x를 랜덤한 양의 noise로 손상시키며, 높은 밀도 영역으로 이동하기 위해 diffusion model의 score fuction을 따르는 업데이트된 방향을 추정한다. Diffusion model을 사용하여 DIP를 학습하기 위한 이 기울기는 diffusion model의 학습된 score fuction을 사용하는 가중 probability density distillation loss의 기울기임을 보여준다.

![스크린샷 2025-02-18 184956](https://github.com/user-attachments/assets/233bdeed-3c58-4f7f-8b90-17020a6d11a8)

- 이러한 샘플링 접근 방법을 Score Distillation Sampling(SDS)라고 한다.
- SDS는 distillation과 연관있지만, density대신에 score함수를 사용한다.
- 확산 과정에서 t→0 이 되면 노이즈가 사라지고, 최적의 3D 샘플을 얻을 수 있다. 즉, SDS는 **최적의 3D 모델을 찾아주는 과정**이라고 볼 수 있기에 sampler라고 말한다.
- 또한 Diffusion 모델은 업데이트 방향만 예측하는 것으로 역전파가 필요없다.(frozen상태)
- $L_{SDS}$의 mode 탐색 특성을 감안할 때 이 loss를 최소화하면 좋은 샘플이 생성되는지 여부가 불분명할 수 있다. 저자들은 경험적으로 classifier-free guidance의 guidance 가중치 ω를 큰 값으로 설정하면 품질이 향상된다는 것을 발견했다.
- SDS는 ancestral sampling에 필적하는 디테일을 생성하지만 파라미터 공간에서 작동하기 때문에 새로운 전이 학습 (transfer learning) 애플리케이션이 가능하다.

![스크린샷 2025-02-18 190203](https://github.com/user-attachments/assets/757ca39e-2f97-4997-b926-4f41085d4348)

---

## DreamFusion Algorithm

![스크린샷 2025-02-19 161618](https://github.com/user-attachments/assets/083635f8-23a0-4c6e-b8e8-af11e27880f3)

- 해당 부분에서는 text로부터 3D asset을 생성해 내는 특별한 알고리즘에 대해서 설명할 것이다.
- Diffusion Model은 64x64를 생성하는 base **Imagen 모델**(Super resolution 제외)을 수정 없이 사용했다.
- text로부터 장면을 합성하기 위해 랜덤한 카메라 위치와 각도에서 NeRF 같은 모델로 반복적으로 Rendering을 한다.(이 때, NeRF같은 모델은 랜덤한 weight로 초기화)
- 이런 Rendering은 Imagen을 둘러싼 score distillation loss function의 input으로서 사용되고, 이런 방식을 사용한 간단한 gradient descent은 결국 텍스트와 유사한 NeRF로 parameterize된 3D 모델을 생성하게 된다.

### Neural Rendering of A 3D Model

- 해당 연구에서는 aliasing을 감소시킨 발전된 버전의 NeRF인 **mip-NeRF 360**을 사용했다.
- mip-NeRF 360는 이미지로부터 3D reconstruction을 위해 설계되었지만, aliasing감소가 text-to3D task에도 도움이 되었다고 한다.
- 기존의 NeRF같은 모델은 ray의 방향에 따른 빛을 방출하는 반면 DreamFusion의 **MLP는  우리가 조절하는 조명에 따라 표면의 색을 parameterize 한다.** 이 과정을 **shading**이라 한다.
- 이 MLP를 통해 우리는 밀도($\tau$)와 albedo($\rho$)를 구할 수 있다.

![스크린샷 2025-02-19 174148](https://github.com/user-attachments/assets/be7181f9-a587-4fa5-a2c9-3510ae027d2d)

- 이전의 NeRF-like 모델들은 여러 reflectance(반사) 모델을 활용했는데, 본 연구에서는 각 점에 대해 **RGB albedo *ρ*(material의 색상)**를 활용한다.
- 3D 포인트의 최종 shaded color를 계산하기 위해서는 albedo($\rho$), normal vector(n), 3D 좌표($\mu$), 빛의 위치($l$), 색상($l_{p}$),  주변 색상($l_{a}$)을 통해 구한다.

![스크린샷 2025-02-19 173706](https://github.com/user-attachments/assets/65db9400-53a1-4553-9318-ed8c3f203b3f)

- $n = -\nabla_{\mu}\tau / |\nabla_{\mu}\tau|$로 구한다.(밀도의 negative gradient를 정규화 한 것)
- 위와 같이 계산된 색상과 밀도를 사용하여 표준 NeRF에서 사용되는 렌더링 가중치 $w_{i}$로 볼륨 렌더링 적분을 근사화한다. Text-to-3D 생성에 대한 이전 연구들에서는 albedo($\rho$)를 흰색 (1, 1, 1)으로 대체하여 "**텍스쳐가 없는**" shaded 출력을 생성한다.

![스크린샷 2025-02-19 175323](https://github.com/user-attachments/assets/69ea379c-dbc0-471f-958b-72c170fc9fd0)

- 이렇게 하면 **모델이 텍스트 조건을 충족하기 위해 평면 geometry에 scene 콘텐츠를 그리는 저하된 solution을 생성하는 것을 방지**할 수 있다.
- DreamFusion은 몇몇 복잡한 장면을 생성할 수 있지만, 해당 연구에서 **고정된 bounding sphere에서 NeRF 표현을 쿼리하는 것**과 위치 관련 인코딩 ray 방향 사용하여 background color 계산하는 두번째 MLP에서 생성된 **environment map을 사용하는 것**이 도움이 되는 것을 발견했다.
- 또한 누적된 알파 값을 사용하여 이 배경색 위에 렌더링된 광선 색상을 합성함으로써, NeRF 모델이 **카메라에 매우 가까운 밀도로 공간을 채우는 것을 방지**하면서 생성된 s**cene 뒤에 적절한 색상이나 배경**을 칠할 수 있다.
- DreamField처럼 불필요한 공간을 빈 공간을 채우는 것을 방지하기 위해 **정규화 peanlty를 포함**했고, Normal vector가 카메라 반대쪽을 향하는 density field에서의 문제를 방지하기 위해 Ref-NeRF에서 제안된 **orientation loss의 수정된 버전을 사용**한다.
- 이런 페널티는 텍스쳐가 없는 shading을 포함할 때 중요한데, 포함하지 않으면 density field가 orient normals을 카메라에서 멀어지도록 하려 하기에 shade가 더 어두워지게 된다.

### Text-To-3D Synthesis

- **DreamFusion의 최적화 과정**
    1. 카메라와 조명을 랜덤하게 샘플링
        - $\phi$(고도각) = [-10, 90], $\theta$(방위각) = [0, 360], $r$(거리) = [1, 1.5] 사이에서 랜덤하게 샘플링한다.
        
        ![스크린샷 2025-02-19 202014](https://github.com/user-attachments/assets/95b99217-585e-4f69-8291-e512c40d4710)

        - 샘플링한 점의 위치와 원점 주변의 look-at point와 up vector를 샘플링하여 결합하여 **camera pose matrix**를 만든다.
        - Focal length는 w(=width=64) 에 0.7~1.35로 균등하게 나눈 상수 λ값을 곱해서 사용했고, Light의 위치는 카메라 위치를 중심으로 한 확률 분포로부터 샘플링했다.
        - 여기에서 **다양한 카메라 위치를 사용하는 것이 일관된 3D scene을 합성**하는 데 중요하며 **카메라 거리는 학습된 scene의 해상도를 개선**하는 데 도움이 된다는 것을 발견했다.
    2. 해당 카메라에서 NeRF의 이미지를 렌더링하고 조명으로 shading 처리
        - 카메라 포즈와 광원 위치가 주어지면, 64×64에서 shaded NeRF model을 렌더링한다.
        - 조명이 있는 컬러 렌더링, 텍스처 없는 렌더링, shading이 없는 알베도 렌더링 중에서 랜덤하게 선택한다.
    3. NeRF 파라미터에 대한 SDS loss의 기울기를 계산
        - 텍스트 프롬프트는 객체의 표준 view는 잘 설명하지만, 다른 view를 샘플링할 때는 좋은 description이 아니다. 따라서 랜덤하게 샘플링된 카메라의 위치에 기반하여 view에 따라 다른 텍스트를 원래 텍스트에 추가하는 것이 좋다.
        - 이를 위해 고도각이 60도 초과인 곳에는 ‘overhead view’를 추가하고, 60도 이하인 곳에서는 ‘front view’, ‘side view’, ‘back view’ 를 방위각을 고려하여 추가하기 위해 text embedding의 가중치 조합을 사용한다.
        - 여기서 weighting 함수로 $w(t) = \sigma_{t}^{2}$를 사용했는데 균등한 weight를 사용한 것과 유사했기에, Noise level이 너무 크거나(t=1) 작은 경우(t=0)에 **수치상 instability가 있었기 때문에, t는 0.02~0.98로 균등하게 샘플링**하였다.
        - classifier-free guidance에서는 $w$를 100으로 설정하였고, $w$가 클수록 샘플의 퀄리티가 향상되는 것을 발견했다. 너무 적은 $w$를 사용할 경우, object를 표현하는 중간값을 찾기 때문에 over smoothing되는 현상이 발생한다.
    4. Optimizer를 사용하여 NeRF 파라미터를 업데이트

---

## Discussion

- DreamFusion은 **3D 또는 다중 뷰 훈련 데이터를 필요로 하지 않으며, 오직 2D 이미지로 훈련된 사전 훈련된 2D 확산 모델**만으로 새로운 Score Distillation Sampling(SDS) 접근 방식과 NeRF 유사 렌더링 엔진을 통해 확장 가능하고 고품질의 2D 이미지 확산 모델을 3D 도메인으로 전이했다.

### 한계점

1. **SDS 손실 함수의 문제점**
    - SDS는 이미지 샘플링 과정에서 완벽한 손실 함수가 아니며, **oversaturation 및 oversmoothing** 문제를 유발한다.
    - Dynamic thresholding 기법이 2D 이미지에서는 이러한 문제를 일부 완화하지만, NeRF에서는 이를 해결하지 못했다.
2. **다양성 부족**
    - SDS를 이용해 생성한 2D 이미지 샘플은 **기존의 Ancestral Sampling 방식보다 다양성이 부족**하다.
    - 3D 결과물 역시 Random seed**에 따른 차이가 거의 없는 경향**을 보인다. 이는 **역방향 KL 발산**을 활용한 방식의 **mode-seeking 성질** 때문일 가능성이 크다
3. **세부 디테일 부족**
    - DreamFusion은 **64×64 해상도의 Imagen 모델**을 사용하므로, 생성된 3D 모델이 세밀한 디테일을 포함하지 못한다.
    - 더 높은 해상도의 확산 모델과 **더 큰 규모의 NeRF**를 사용하면 이를 해결할 수 있지만, **연산 속도가 지나치게 느려지는 문제**가 발생할 수 있다. 따라서 향후 **더 효율적인 확산 모델과 신경 렌더링 기법의 발전이 필요**하다.
4. **3D reconstruction의 본질적인 모호함**
    - **2D 이미지에서 3D 구조를 복원하는 문제는 본질적으로 ill-posed 문제**이다.
    - 같은 2D 이미지를 기반으로 할 때, **다양한 3D 해석이 가능**하므로 최적화 과정에서 **local minima 문제**가 발생할 수 있다.
