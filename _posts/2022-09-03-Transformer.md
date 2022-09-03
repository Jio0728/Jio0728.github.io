---
title: Transformer 기본 개념 정리
use_math: true
comment: true
---

### Table of Contents
1. Attention이란?
2. Self-Attention
3. Multi-Head Attention
  a. Multi-Head Attention: Splitting
  b. Multi-Head Attention: Attention Operation
  c. Multi-Head Attention: Merging
4. Transforemrs
  a. Encoder
  b. Decoder

* * *

# Attention이란?
Attention은 한 마디로 정의하자면, __각 구성요소들이 서로 얼마나 연관되어 있는지__ 이다. 그 대상은 이미지의 다른 부분일 수도, 또는 문장 내의 서로 다른 단어들일 수도 있다.   
우리는 이미지 또는 문장을 이루는 구성요소들의 상호연관성을 파악하여 모델 학습 시 어디에 집중할지를 참조시킬 수 있다.   
Attention 모듈은 Q(Query), K(Key), V(Value) 행렬을 활용하여 구성요소들 간 관계들의 중요도를 파악한다. Self Attention과 Multi-Head Attention은 세부적인 계산사항은 다르지만, 비슷한 논리로 Attention(구성요소 간 관계들의 중요도)을 계산한다.
지금부터 Attention의 notaion을 정의하고, Self Attention과 그것의 확장판인 Multi-Head Attention이 각각 어떻게 Attention을 계산하는지 살펴보도록 하겠다.   
<br>
다음은 지금부터 사용할 기호들의 정의이다.
n: time step/sequence   
N: Vocabulary size   
d: Embedding size   
<br>
X: Input tensor(n \* N matrix)   
Q: Query(n \* d_1 matrix)   
K: Key Matrix(n \* d_2 matrix)   
V: Value Matrix(n \* d_3 matrix)   
<br>
W1, W2, W3는 X에 곱해져서 각각 Q, K, V를 만들어내는 weigh matrix이다.    
K = W_1 \* X   
Q = W_2 \* X   
V = W_3 \* X
즉, W_i는 N \* d_i의 차원이다.   
위에 적은 세 개의 weight matrix가 attention 모듈, 그리고 transformer에서 업데이트하는 **유일한 learnable parameter** 이다.


# Self Attention이란?
## Self Attention의 기본적인 작동방법
기본적인 스텝은 다음과 같다.   
1. Score Matrix를 다음과 같은 식으로 계산한다.
  S = Q\*K^T
2. Score Matrix를 정규화(normalization)한다. 이는 gradient descending/exploding을 방지하기 위함이다.
  S_n = S/\sqrt{d_k}
3. Score Matrix를 softmax 함수를 활용하여 확률값으로 변환한다.
  P = softmax(S_n)
4. Weighted Value Matrix인 Z를 구한다.   
  Z = V\*P   
<br>
이제 각 단계들을 찬찬히 살펴보며 각 단계와, 그 단계의 결과물인 행렬들의 의미에 대해서 이해하는 시간을 가져보자.   
그 전에, Q, K, V에 대해 감을 잡아보자.   
구글에 "흰색 셔츠에서 기름 때 제거하는 법"이라고 검색했다고 가정해보자. 구글에서는 이 검색내용과 비슷한 여러가지 검색결과를 내보일 것이다. 그 중에는 "흰색 셔츠에서 김치 국물을 제거하는 법", "흰색 셔츠 다리는 법"이라는 제목의 블로그들이 있을 것이다.
그리고 그 중 "셔츠에서 기름 때 제거하기" 라는 제목의 블로그 글을 보인다. 우리는 그 블로그 글이 우리가 검색한 것과 가장 비슷하다고 판단하고, 해당 블로그에 들어간다.   
이 때 우리가 검색한 내용, "흰색 셔츠에서 기름 때 제거하는 법"은 Q matrix에 해당한다. 즉, Q는 우리가 찾으려는 것이다.   
그리고 구글이 선보이는 여러 비슷한 결과들은 K matrix에 해당한다. 즉, K는 연관성 있는 것들의 집합이다.    
우리가 결국 선택한 "셔츠에서 기름 때 제거하기"라는 블로그는 V matrix에 해당한다. 즉, V는 우리가 찾는 것과 가장 비슷한 것을 의미한다.   
해당 사례로 조금이나마 감을 잡으면 다음 step에 대한 이해가 조금 더 쉬워질 것이다.

#### Score Matrix
Score Matrix는 결국 단어들 간의 연관도를 나타내는 행렬이다.   
Q\*K를 함으로써 우리는 단어들 간의 연관도를 계산한다. 그리고 score matrix를 정규화함으로써 gradient의 안정화를 추구한다.   
안정화된 score matrix를 softmax 함수에 넣으면 한 행의 sum 값은 1이 된다. 그 결과를 P라고 지칭한다.   
P 행렬의 각 행을 이루고 있는 값들 중에는 최댓값이 존재할 것이다. 이 때, P 행렬이 softmax의 결과값이기 때문에 이 최댓값은 **최대 확률** 을 나타낸다.   
<br>
step4에서 이 P 행렬을 V 행렬과 곱하는데, 이 단계에서 잠시 행렬곱의 얘기를 하겠다. 




