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
보통 embedding 차원의 값은 통일하기 때문에 이 블로그에서도 d라고 나타내겠다. (d_1 = d_2 = d_3 = d)
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
  Z = P\*V   
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
S = Q\*K^T   
위 식의 Q와 K행렬은 둘 다 n * d 차원의 행렬이다. n이 time step 또는 sequence를 의미한다고 봤을 때, 이 행렬의 차원이 의미하는 것은 다음과 같다.   
두 행렬 모두 각 행은 한 단어를 d차원의 embedding 벡터로 표현하는 것이다.   
예를 들어보자. Input이 I am a boy라는 문장이 있다고 생각해보자. Q, K 행렬의 첫번째, 두번째, 세번째, 네번째 행은 각각 I, am, a, boy를 상징한다. 그리고 각 행은 d차원의 임베딩 벡터로 표현되는 것이다.     
그렇다면 Q와 K^T를 곱하면 무슨 일이 일어날까? 행렬곱은 아래의 이미지와 같이 일어난다.
![274D6B4152FB2FB105](https://user-images.githubusercontent.com/87808237/188271389-e0011dd9-f566-4ee9-a10f-48067dafc2db.png)
즉, Q의 각 행이 K^T의 각 열에 곱해진다. 다른 말로 하면, Q의 각 행이 K의 각 행에 곱해진다. 즉, Q행렬에서 각 단어를 나타내는 d차원의 벡터가 K행렬에서 각 단어를 나타내는 d차원의 벡터와 내적하는 것이다.   
우리는 내적 값이 크면 두 벡터 간 코사인 거리가 가깝다는 것을 알고 있다. 즉, Q와 K의 행렬곱을 통하여 각 행렬에서 각 단어들의 거리, 다른말로는 연관성을 계산하는 것이다.    
그렇게 계산된 결과는 P 행렬이고 P 행렬은 n\*n의 크기이다. 그리고 [P]~ij~(P행렬의 i번째 row의 j번째 columns의 값)은 input의 i번째 단어와 j번째 단어 간의 연관성을 의미한다.
즉, [P]~12~값은 I와 am의 연관도를 의미한다.
<br>
이후 P 행렬은 softmax 함수를 통과할 것이고, 각 행의 sum값이 1인 형태로 변환될 것이다. P 행렬의 각 행을 이루고 있는 값들 중에는 최댓값이 존재할 것이다. 이 때, P 행렬이 softmax의 결과값이기 때문에 이 최댓값은 **최대 확률** 을 나타낸다.   
<br>
step4에서 이 V 행렬을 P 행렬과 곱하는데, 이 단계에서 잠시 행렬곱의 얘기를 상기하자.    
P\*V 계산식을 살펴보면, P의 각 행을 V의 각 열에 곱한다는 것을 알 수 있다. P 행렬은 단어들 간 연관성을 나타내는 행렬이다. 한 단어와 다른 단어들 간의 연관성을 나타내는 벡터를 V벡터의 각 열 벡터와 내적함으로써 각 P벡터의 가중치를 V벡터의 각 단어의 embedding 값에 곱하여 단어들의 weighted sum을 구하게 된다. 이를 통해서 weight가 큰 단어의 임베딩 벡터에 가깝게 V벡터의 각 행이 구성될 것이고 그 결과인 Z 행렬은 결국 가중치의 관점에서 바라본 V행렬의 lookup table이 된다.
<br>
<br>
#### Masking






