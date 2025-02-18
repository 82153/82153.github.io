---
layout: article
title: "Qwen-Audio: Advancing Universal Audio Understanding via Unified Large-Scale Audio-Language Models"
date: 2025-01-13 16:21 +0900
category: Paper_Review
tags: [Paper_Review]
aside:
  toc: true
sidebar:
  nav: paper_review
---
## Abstract

- Instruction-following 오디오- 언어 모델은 많은 집중을 받았지만, 다양한 오디오 type과 task를 다룰 수 있는 사전 학습된 모델이 없어 발전이 느렸다. → 그래서 Qwen-Audio 모델을 개발하여, 30개 이상의 작업 및 다양한 오디오 유형을 포함하도록 오디오-언어 사전 훈련을 확대하여 보편적인 오디오 이해 능력을 촉진함으로써 이 한계를 해결하려 함
- 하지만 서로 다른 데이터 셋과 관련된 텍스트 라벨이 task focus, 언어, 주석의 수준, 텍스트 구조의 차이로 인해 상당한 variation을 가지기에 **Interference(간섭) 문제**가 있음
- 간섭 문제를 해결하기 위해, 각각 **공유 및 지정된 태그를 통해 간섭을 피하고** **디코더에 계층적 태그 시퀀스를 조건으로 하여 지식을 공유**하는 multi-task training 프레임워크를 설계함 → 이를 통해 fine tunning 없이도 괜찮은 성능을 얻음

---

## Introduction

- LLM 모델은 텍스트가 아닌 모달리티(오디오, 이미지 등)에 대해 인지 능력이 떨어진다.
- LLM이 오디오와의 상호작용을 위해 오디오 신호를 이해하고 인지할 수 있게 하는 것이 많은 관심을 받았다.
- LLM의 능력을 상속 받고, 사용자의 의도에 맞게 모델의 능력을 활성화하기 위해 Fine-Tunning을 함 → 하지만 다양한 오디오 유형과 작업을 처리할 수 있는 **사전 학습된 오디오-언어 모델의 부족**으로 인해 오디오 상호작용 능력에 제한을 받음
- 또한 기존의 존재하던 오디오 - 언어 multi-task 언어 모델들은 특정 오디오 타입에 제한되어 있다.
- 그래서 audio-text multimodal 의사소통을 위해, **large scale audio-language 모델인 Qwen-Audio**를 개발했다.
- Qwen-Audio는 audio와 text 입력을 조건으로 하는 multi-task 언어 모델이다. 이 모델은 Qwen-7B에 single 오디오 인코더를 연결하여 효율적으로 오디오를 인지하는 언어 모델이다.
- Qwen-Audio 모델은 이전의 모델들과 달리 특정 오디오 타입이나 task에 초점을 둔 것이 아닌 다양한 유형의 오디오를 다루는 수십 개의 데이터셋으로 훈련을 확장하여 보편적인 오디오 이해 능력을 발전시켰다.
- Multi-task, Multi-dataset 학습에서 중요한 것은 다른 데이터셋과 연관된 textual label에서의 variation이다. 이것은 작업 목표, 언어, 주석의 세분성, 그리고 텍스트 구조에서 비롯된다.
- 이것을 극복하기 위해, 해당 논문에서는 디코더에 **sequence of hierarchical tag**를 조건으로 하는 학습 Framework를 설계했다. 공유되고 구분된 tag를 통해 지식의 공유를 돕고, 간섭을 완화 시키는 것을 돕는다.

---

## Methodology

![스크린샷 2025-01-13 232315](https://github.com/user-attachments/assets/37e28e89-6e98-491c-9964-509c00333c7f)

### Model Architecture

- 모델의 구조는 **Audio-Encoder**와 **LLM**으로 구성된다.
- 이 모델은 **audio sequence($a$)**와 **text sequence($x$)**가 주어졌을 때, **다음 text token을 예측**하는 것이다. 여기서 text sequence는 현재 시점 이전의 모든 $x$를 기반으로 한다.

![스크린샷 2025-01-13 232419](https://github.com/user-attachments/assets/898a7b41-d7ea-45eb-8f81-645deb3c5c81)

- 여기서 $\phi$는 Encoder의 파라미터, $\theta$는 LLM의 파라미터로 둘 다 학습 가능하다.

![스크린샷 2025-01-14 001608](https://github.com/user-attachments/assets/69a72fdd-4600-4ca1-b573-b4be12a43bf0)

- **Audio-Encoder**
    - Whisper-large-v2를 사용함
    - Whisper-large-v2는 2개의 down-sampling convolution layer와 32 layer Transformer로 구성된 모델이다.
    - Whisper-large-v2는 speech recognition과 번역을 위해 지도 학습된 모델이지만 Whisper의 인코딩된 표현은 배경 소음과 같은 풍부한 정보도 포함하고 있고, 이 인코딩된 표현은 원본 speech를 복원할 수도 있다.
    - **audio-encoder(Whisper)에서 전처리**
        - 16kHz의 주파수로 resampling
        - Window size는 25ms, hop size는 10ms를 사용하여 80 채널의 mel-spectrogram으로 원본 waveform을 변환
        - 보폭이 2인 pooling layer를 통해 오디오 representation의 길이를 줄임
        - 결론적으로 인코더 출력의 각 프레임은 원래 오디오 신호의 약 40ms Segment에 해당되게 된다.
        - 여기에 추가적으로 SpecAugment도 사용됨
- **LLM**
    - LLM은 **Qwen-7B**를 사용한다.
    - Qwen-7B 모델은 32 Layer의 Transformer Decoder모델이다.

---

## Training Process

### **Multitasking Pretraining**

- Qwen-Audio는 다양한 오디오 데이터셋을 활용하여 공동 훈련(co-training)을 수행하는 것을 목표로 하고, 그 목적은 모든 오디오 작업을 지원할 수 있는 통합 모델을 훈련하여, 서로 다른 작업을 수행할 때  모델 전환을 없애는 것이다.
- 더 중요한 것은, 공동 훈련 동안 유사한 작업들이 서로의 이익을 나눌 수 있다는 것이다.
    - 오디오 신호에 내재된 기본적인 정보에 대해 공통적인 초점을 공유하기에, 비슷한 작업들은 지식 공유와 협업 학습을 통해 이익을 얻을 수 있다.
    - 낮은 수준의 인지 능력에 의존하는 작업들은 더 높은 수준의 이해나 추론 능력을 요구하는 작업을 지원할 수 있다.
- 기존에는 데이터 셋을 섞어서 co-training을 하거나, 비슷한 task끼리 묶기, 데이터 셋 ID를 지정하는 방법을 사용했지만 더 개선할 필요가 있음
- 그래서 Whisper에서 사용한 것처럼 Decoder에 input token sequence에 tag와 조건을 추가하는 방식을 사용함(Whisper는 단순히 speech translation과 인지 task만 가능했음)

![스크린샷 2025-01-14 004443](https://github.com/user-attachments/assets/a6fc0ba8-7520-4fd9-b5a3-5677385dab82)

| Tag | Description |
| --- | --- |
| Transcription Tag | Prediction의 시작을 나타내는 Tag (Transcription이 필요한 task인지 아닌지 나타냄) |
| Audio Language Tag | Audio에서 나오는 언어를 나타내는 Tag, 8개에 포함 안된 언어나 음악이나 자연의 소리는 UNKNOWN |
| Task Tag | 수행할 구체적인 Task를 나타내는 Tag |
| Text Language Tag | 출력으로 나올 Text의 언어를 나타내는 Tag |
| Timestamps Tag | 모델이 timestamp를 예측할 필요가 있는지 나타내는 Tag |
| Output Instruction | 세부 Task 및 출력 형식을 나타내는 Tag |
- Whisper에서는 문장 단위의 timestamp를 사용했지만, Qwen-audio는 단어 단위의 timestamp를 지정→SRWT(Speech Recognition with Word level Timestamp) token이 transcription 되기 전에 start time 예측, transcription 후에 end time 예측

![스크린샷 2025-01-14 141236](https://github.com/user-attachments/assets/d0d84c5f-33d4-4d3b-847e-89377d4b3488)

- SRWT는 모델의 audio 신호와 timestamp를 정렬하는 능력을 향상 시킴. 이로 인해 speech 신호를 종합적으로 이해하는데 기여했고, speech 인식과 QA같은 많은 task에서 눈에 띄는 발전을 함
- 결론적으로 이런 Framework를 사용함으로써, **공유된 tag를 통해 비슷한 task들 사이의 지식의 공유를 최대화**하고, 모델의 one-to-many 문제를 피하기 위해 **서로 다른 task와 output 형식을 구별할** 수 있게 되었다.
- Multitask 모델의 광범위한 사전 훈련은 모델이 오디오에 대한 폭넓은 이해를 제공하게 되었다. 이를 기반으로 모델이 **사람의 의도에 맞게 align하는 능력**을 향상 시키기 위해 **instruction based fine-tunning**을 적용했다. → 이게 Qwen-audio-chat이다.

### Supervised Fine-tunning

- 각 task별로 수동으로 raw text label, question, answer으로 구성된 demonstraion을 만들었고, 제공된 raw text label을 통해 GPT-3.5를 통해 추가적으로 question과 answer를 더 만들었다.
- 또한 manual annotation, model generation, strategy concatenation을 통해 audio-dialogue 데이터 세트를 생성
- 이런 데이터 셋은 모델에 reasoning, story generation 및 multi image 이해 능력을 통합하는 데 도움을 준다.
- multi-audio dialogue와 multiple audio 입력을 효율적으로 다루기 위해, 각 오디오를 "Audio id:"로 Labeling하는 규칙을 도입했으며, 여기서 id는 오디오 입력 대화의 순서를 나타낸다.
- ChatML 형식을 사용하여 instruction tunning dataset을 dialogue형식으로 변환→<im_start>와 <im_end> 토큰이 statement에 추가된 형식이다.
- Multi-turn dialogue에서  audio와 순수 text modality의 다양한 입력을 지원하기 위해, 위에서 언급한 audio-centric instruction data와 순수 text instruction data를 결합하여 훈련 과정을 진행했고, 이러한 접근 방식은 모델이 다양한 형태의 입력을 원활하게 처리할 수 있도록 한다.

---

## Experiments

- Qwen-audio 학습 시에는 LLM을 Freeze하고, Audio-Encoder만 학습
- Qwen-audio-chat 학습 시에는 Audio-Encoder를 Freeze하고, LLM만 학습

![스크린샷 2025-01-14 142431](https://github.com/user-attachments/assets/bb01c2b6-3562-4578-b74e-2a83b641fa6a)

---

## Conclusion

- Qwen-audio는 universal한 오디오 이해 능력을 갖춘 large-scale audio-language model 세트이다.
- 다양한 유형의 오디오를 co-training에 통합하기 위해, 유사한 작업 간의 지식 공유를 촉진하고 서로 다른 텍스트 형식으로 인해 발생하는 one-to-many mapping 문제를 방지하는 통합된 multi-task 학습 프레임워크를 제안
- 어떤 작업에 대해서도 fine-tunning 없이, 결과적인 Qwen-Audio 모델은 다양한 벤치마크에서 이전 모델 보다 더 나은 성능을 보인다. 이는 Qwen-Audio가 universal한 오디오 이해 능력을 가졌다는 것을 입증한다.
