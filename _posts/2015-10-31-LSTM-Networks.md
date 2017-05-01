---
layout: post
title:  "Long Short Term Memory Network ( LSTM )"
date:   2015-10-31 22:00:00
author: Soyoon
categories: ML
tags: LSTM
---

#### 목표 : RNN의 종류 중 하나인 LSTM 에 대한 개념을 이해

#### 내용 : [Understanding LSTM Networks](http://colah.github.io/posts/2015-08-Understanding-LSTMs/)를 보고 그냥 읽히는대로 정리 (오역 많을듯)

#### 결론 : RNN은 긴 앞 문맥을 학습하지 못한다. LSTM이 긴 문맥을 학습하는데 유용

### Understanding LSTM Networks

## Recurrent Neural Networks

사람들은 생각을 매 초마다 scratch해서 생각을 하지 않는다. 이 글을 읽을 때, 이전의 단어들의 이해를 기반으로 각 단어를 이해한다. 사람들은 모든 것을 지우고 다시 생각을 모으지 않는다. 생각에는 영속성이 있다.

전통적인 neural network는 이런 것을 할 수 없었고, 이게 주요 단점이었다. 예를들면, 영화의 각 장면에서 무슨 종류의 이벤트가 발생하는지를 분류하고 싶다고 생각해보자. 전통적인 neural network가 영화의 이전 이벤트를 나중의 이벤트에 알려주기 위한 추론을 어떻게 사용하는지는 불명확했다.

Recurrent neural networks는 이 이슈를 address 한다. RNN은 loop를 가지고 있는 네트워크이고, 그 loop가 정보를 persist 하게 만든다.

위의 뉴럴 네트워크의 한 chunk A에서 input은 x_i로, output은 h_i를 보자. A loop은 정보가 한 네트워크 단계에서 다음 네트워크로 지나가게 한다.

이 loop가 recurrent neural network를 모호한 것처럼 보이게 한다. 하지만 당신이 조금 더 생각해보면, rnn은 일반 뉴럴 네트워크와 크게 다르지 않음을 알 수 있다. RNN은 메시지를 다음 네트워크에 전달해주는 동일한 네트워크의 복사본으로 생각할 수 있다. 우리가 loop를 펼치면 어떻게 되는지 보자.

이 chain 같은 성질은 recurrent neural networks가 순서와 리스트에 긴밀하게 연관되어있다는 것을 나타낸다. 그것들은 데이터를 이용하기 위한 뉴럴 네트워크의 자연스러운 구조이다.

그리고 그것들은 확실히 사용되고 있다. 최근 몇년간, 다양한 문제들( 음성인식, 언어모델링, 번역, 이미지 캡쳐) 에 RNN이 성공적으로 적용되어왔다. 다른 분이 잘 적어주었기 때문에 더이상 얼마나 RNN이 대단한지에 대한건 생략하겠다.

이 성공에 가장 주요 원인은 RNN의 종류중 하나인 LSTM을 사용하였기 때문이다. LSTM은 많은 task에서 기존 버전보다 훨씬 잘 작동한다. recurrent neural network를 기반의 흥미로운 결과 대부분은 LSTM으로 달성한 것이다. 그래서 LSTM을 이 글에서 더 조사해 볼 것이다.

### The Problem of Long-Term Dependencies

RNN이 어필하는 것 중에 하나는 이전 비디오 프레임을 이용해서 현재 프레임을 이해하도록 알려주는것 처럼 이전 정보를 현재 task에 연결할 수 있다는 아이디어이다. 만약 RNN이 이걸 할 수 있었다면 굉장히 유용했을것이다. RNN이 할 수 있었을까? 대답은 그때그때 다르다 이다.

가끔, 우리는 현제 task를 수행하기 위해 최근 정보를 봐야할 필요가 있다. 예를들면, 언어모델이 이전 단어를 기반으로 다음 단어를 추측하는 경우를 생각해보자. “the clouds are in the sky” 이 문장에서는 우리는 마지막 단어가 sky일 것이라는 것은 꽤나 명백하기 때문에 더 앞의 문맥을 알 필요가 없다. 적절한 정보와 그게 필요로하는 부분 사이의 gap이 적은 경우에 RNN은 과거 정보를 이용할 수 있다.

하지만 우리가 더 많은 문맥을 필요로 하는 경우도 있다. “I grew up in France… I speak fluent French.” 의 마지막 단어를 예측하려고 하는 경우에는 한 문장만 보면 알 수 없어서 더 앞의 문맥을 보고 French를 예측할 수 있다. 적절한 정보와 그게 필요로하는 부분 사이의 간격이 매우 큰 경우에 가능하다.

불행히도, 간격이 늘어날수록 RNN은 정보를 연결하는것을 학습할 수 없게 되었다.

이론적으로, RNN은 long-term dependencies를 핸들링할 수 있다. 사람은 파라미터를 신중히 골라서 이런 형식의 문제를 해결할 수 있을 것이다. 슬프게도, 실제 RNN은 그렇게 학습할 수 있지 않아보인다. 이 문제는 왜 그것이 어려운가에 대한 기본 이유를 찾은 Hochreiter (1991) [German] and Bengio, et al. (1994)가 자세히 설명해두었다.

고맙게도, LSTM은 이런 문제를 가지고 있지 않다!

### LSTM Networks
Long Short Term Memory networks (우리는 LSTMs 로 부른다) 는 long-term dependencies도 학습 가능한 RNN의 한 종류이다. Hochreiter & Schmidhuber (1997)에 의해 소개되었고, 많은 사람들에 의해 정제되고 대중화되었다. LSTM은 크고 다양한 문제에 대해 매우 잘 작동하고, 지금은 광범위하게 사용된다.

LSTM은 long-term dependency 문제를 피하기 위해 고안되었다. 학습하려고 노력하는게 아니라 오랜 기간 전의 정보를 기억하는것이 LSTM의 기본적인 행동이다.

모든 recurrent neural networks는 뉴럴 네트워크 모듈을 반복하는 체인 형식을 가지고 있다. 기본 RNN에서 이 반복하는 모듈은 tanh 처럼 매우 간단할 것이다.

LSTM 역시 체인 같은 구조를 가지지만, 반복되는 모듈 구조는 다르다. single neural network layer를 가지는 대신 특별한 방법으로 상호작용하는 4개의 층으로 구성된다.

step by step으로 나중에 볼테니 LSTM이 어떻게 작동하는지 세부적인 내용은 걱정하지 말자. 우선 지금은 우리가 사용하려는 개념에 익숙해지도록 하자.

위의 그림에서 각 line은 한 노드의 output에서부터 다른 input으로 전체 벡터를 이동시킨다. pink circle는 vector 덧셈 같은 point wise operation을 나타낸다.노란 박스는 학습된 뉴럴 네트워크 층이다. Line merging은 concatenation(연결)을 나타내고, line forking은 내용이 복사되고 각 복사본은 다른 위치로가는 것을 의미한다.

### The Core Idea Behind LSTMs
LSTM의 key는 그림의 제일 위를 수평으로 지나가는 cell state이다.

cell state는 컨베이어 벨트 같은 종류이다. 이것은 일부 minor한 선형 상호작용과 함께 전체 체인을 쭉 동작한다. 정보가 변하지 않게 그걸 따라가는건 매우 쉽다.

LSTM은 gate라고 부르는 구조에 의해 정보를 cell state에 지우고, 추가할 수 있다.

Gate는 선택적으로 정보를 통과시키는 길이다. Gate는 sigmoid neural net layer과 pointwise 곱셈 연산으로 구성되어 있다.

sigmoid layer는 얼마나 많이 각 component를 통과시킬지 결정하는 0과 1사이의 숫자를 출력한다.0은 아무것도 통과시키지 말라는 의미이고, 1은 모두 통과시키라는 의미이다.

LSTM은 cell state를 보존하고 통제하기 위해 세가지 gate를 가지고 있다.

### Step-by-Step LSTM Walk Through

LSTM의 첫번째 단계는 cell state에서 어떤 정보를 버릴지 결정하는 단계이다. 이 결정은 "fotget gate layer" 로 불리는 sigmoid layer에 의해 결정된다.h_t-1 과 x_t 값을 보고 cell state의 C_t-1에 0과 1 사이의 값을 출력한다. 1은 "completely keep this"를 의미, 0은 "completely get rid of this" 를 의미.

언어모델로 돌아오자. cell state가 현재 주어의 gender를 포함해서 그에 맞는 대명사를 쓸 수 있는 문제가 있다고 해보자. 우리가 새 주어를 보면 우리는 이전 주어의 gender를 지우고 싶다.

다음 단계는 어떤 정보를 cell state에 저장할지 결정하는 단계이다. 이건 두 부분으로 구성되는데 하나는 어떤 값을 업데이트할지 결정하는 "input gate layer"라고 불리는 시그모이드 layer이고, 다른 하나는 state에 추가될 수 있는 새 후보 값들의 벡터를 생성하는 tanh layer이다. 다음 단계에서 우리는 이 둘을 결합해서 state를 업데이트한다.

언어모델 예시에서, 우리는 새 주어에 gender를 추가하고 지운 이전 gender를 대체한다.

이제 이전 cell state C_t-1를 새 cell state C_t로 업데이트할 때이다.
이전 단계에서 무엇을 할지 결정했으므로 실제로 진행하면 된다.

old state에 f_t를 곱해서 이전에 어떤걸 지울지 결정한 것을 지운다. 그리고  i_t ∗ C̃_t 를 더한다. 이건 각 스테이트 값에 업데이트를 얼마나 할지 계산된 후보 값들이다.

언어 모델에서는, 실제 이전 주어의 성별 정보를 지우고 앞 단계에서 결정한 새 정보를 추가하는 부분이다.

마지막으로, 우리는 output 값을 결정한다. 이 출력값은 cell state에 기반하지만 filter된 버전이다.
먼저, sigmoid layer를 돌려서 cell state에서 어떤 부분을 출력할지 정한다. 그리고 cell state를 tanh를 통과시켜서 -1 과 1 사이 값으로 나오게 한다. 그리고 이 결과에 sigmoid 출력값을 곱해서 실제 출력값을 만든다.

언어모델에서, 주어를 방금 봤기 때문에 다음에 오는 출력 정보는 동사일 수 있다. 예를들어 주어가 단수 또는 복수인경우 동사의 복합된 형식을 알 수 있다.

### Variants on Long Short Term Memory

지금까지 설명한 내용은 꽤나 일반적인 LSTM이다. 하지만 모든 LSTM들이 위와 같지는 않다. 실제로 LSTM들은 각각 살짝 다른 버전을 사용한다. 차이는 매우 minor하고 언급할 필요는 없다.

하나 유명한 LSTM 은 Gers & Schmidhuber (2000)에 의해 소개되었는데 이 LSTM은 gate layer를 cell state로 보는 “peephole connections.”를 추가했다.

== 이후 부터는 넘어감 ==
