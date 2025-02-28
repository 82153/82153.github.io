---
layout: article
title: "[NLP] Tokenization"
date: 2025-02-28 16:15 +0900
category: NLP
tags: [NLP]
aside:
  toc: true
sidebar:
  nav: NLP
---
## Tokenization(토큰화)

- 기본적으로 자연어 데이터는 각 timestep에 대해서 **word나 character의 Sequence**로 표현된다.
- 이런 식으로 주어진 자연어 데이터(Text)를 **Token단위로 분리하는 방법이 Tokenization**이라 하고 이 Token을 NLP 모델의 입력으로 사용한다.
- 이 때, 모델이 처리할 수 있는 단어는 **단어 사전**(**Vocabulary**)에 정의되어 있다. 이 사전을 통해 Token을 **one-hot vector**로 표현할 수 있다.

---

## Tokenization 방식

1. **Word-Level Tokenization**
    - 일반적으로 단어는 **띄어쓰기를 기준**으로 구분한다. 하지만 띄어쓰기가 없는 경우에는 **형태소를 기준**으로 단어를 구분하기도 한다.
    - 만약 사전에 없는 단어가 등장한 경우에는 모두 “Unknown” 토큰으로 처리한다. 이런 경우를 **Out-of-vocabulary(OOV)**라고 한다.
    - OOV는 사전에 없는 단어를 모두 “Unknown” 토큰으로 처리하기에 **모델이 Text의 의미를 제대로 이해하고 처리할 수 없다**는 문제가 생긴다.
    - Ex
        
        **단어 기준**
        
        - I am spider man → [’I’, ‘am’,  ‘spider’, ‘man’]
        
        **형태소 기준**
        
        - 나는 밥을 먹었다 → [’나’, ‘는’., ‘ ’, ‘밥’, ‘을’, ‘먹’, ‘었다’]
        - 문장 안의 공백도 하나의 입력으로 볼 수 있다.
2. **Character-Level Tokenization**
    - Token을 **철자(Character) 단위**로 구분하는 방식이다.
    - 이 방식은 단어가 아닌 철자이기에 **같은 문자 체계를 사용하는 다양한 언어를 처리**할 수 있고, 또한 문자가 정해진 문자 체계에서 벗어날 수 없기에 **OOV문제도 발생하지 않는다.**
    - 하지만 Text에 대한 Token의 개수가 지나치게 많아져 연**산량이 증가**하고, 각 Token들이 유의미한 정보를 가진다고 보기 어렵기에 일반적으로 **모델의 성능이 낮다**고 한다.
    - Ex
        
        I am spider man → [’I’, ‘ ’, ‘a’, ‘m’, ‘ ’, ‘s’, ‘p’, ‘i’, ‘d’, ‘e’, ‘r’,  ‘ ‘, ‘m’, ‘a’, ‘n’]
        
3. **Subword-Level Tokenization**
    - Token을 **Sub-word 단위**로 구분하는 방식이다. 주어진 하나의 단어라도 각각의 의미를 가지는 subword로 쪼개어 한 단어를 대상으로 여러 의미의 Token으로 나눌 수 있다.
    - 이 방식은 **Character-Level Tokenization보다 Token의 개수가 적고, OOV문제가 없다.** 또한 성능도 가장 좋았다고 한다.
    - Ex
        
        The devil is in the details → [’The’, ‘ ’, ‘de’, ‘vil’, ‘ ‘, ‘is’, ‘ ‘, ‘in’, ‘ ‘, ‘the’, ‘ ’, ‘de’, ‘tail’, ‘s’]
        

---

## Byte Pair Encoding(BPE)

- **대표적인 Subword-Level Tokenization**이다.
- 우선 Character 단위로 단어 목록을 만들고, 가장 빈도수가 높은 단어 pair를 Token으로 추가해가며, 내가 사전에 정의한 vocabulary의 길이까지 Token을 추가한다.
- 이렇게 얻는 vocabulary에서 입력에 대해 vocabulary에서 가장 긴 문자열을 토큰으로 매칭한다.
- Ex
    
    **학습 데이터**
    
    | **단어** | **빈도 수** |
    | --- | --- |
    | low | 5 |
    | lower | 2 |
    | newest | 6 |
    | widest | 3 |
    | lost | 1 |
    
    **초기 단어 목록** → [l, o, w, e, r, n, s, t, ,i, d]
    
    **단어 Pair**
    
    | **pair** | **빈도** | **pair** | **빈도** | **pair** | **빈도** |
    | --- | --- | --- | --- | --- | --- |
    | l, o | 8 | n, e | 6 | w, i | 3 |
    | o, w | 7 | e, w | 6 | i, d | 3 |
    | w, e | 8 | e, s | 9 | d, e | 3 |
    | e, r | 2 | s, t | 10 | o, s | 1 |
    
    가장 빈도수가 많은 st를 단어 목록에 추가
    
    **새로운 단어 목록** → [l, o, w, e, r, n, s, t, ,i, d, st]
    
    **새로운 단어 pair**
    
    | **pair** | **빈도** | **pair** | **빈도** | **pair** | **빈도** |
    | --- | --- | --- | --- | --- | --- |
    | l, o | 8 | n, e | 6 | w, i | 3 |
    | o, w | 7 | e, w | 6 | i, d | 3 |
    | w, e | 8 | e, st | 9 | d, e | 3 |
    | e, r | 2 | o, st | 1 |  |  |
    
    새로운 단어 pair에서 다시 빈도수가 높은 pair 추가
    
    **새로운 단어 목록** → [l, o, w, e, r, n, s, t, ,i, d, st, est]
    
    이러한 과정을 내가 사전에 정의한 vocabulary 사이즈가 될 때까지 반복
    
- 이 외에도 학습 데이터 내의 **우도 값을 최대화하는 word 쌍을 vocabulary에 추가하는 WordPeic**e와 **Subword가 띄어쓰기 뒤에 등장하는지, 가운데에 등장했는지를 구분**해서 사전을 구축하는 **SentencePiece**가 있다.