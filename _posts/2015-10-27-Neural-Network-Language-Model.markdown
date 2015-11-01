---
layout: post
title:  "Languege Model, Recurrent Neural Network"
date:   2015-10-27 22:00:00
categories: Neural Network
---

##Recurrent Neural Network Language Model

####1. 언어 모델이란 ? 
- 텍스트 데이터를 이용하여 학습시킨 모델
- 문장 시퀀스에 대한 확률값 or 스코어를 계산
- 문장을 구성하는 다음 단어를 추측할 수 있다.

####2. 언어 모델의 응용
- 문장에 대한 확률 값 계산
- 새로운 문장의 생성

####3. 언어 모델 종류
- ngram 기반 : SRILM, MITLM
- DNN 기반 : RNNLM, Kaldi

==*RNN 기반 언어모델을 공부해야겠다.*==

####3. RNN이란?
- 이전 단계의 결과가 현재 계산에 영향을 주는 Neural Network. 
- Sequence 정보를 처리할 수 있다.
- Hidden state : 네트워크의 메모리
- RNN의 종류 중 하나인 LSTM이 가장 많이 사용 됨.
- 입력 : 단어들의 시퀀스
- 출력 : 추측된 단어들의 시퀀스

###3. RNN 학습
- Backpropagation Through Time( BPTT)사용
-- 각 출력 부분의 gradient가 현재 시간 스텝에만 의존하지 않고 이전 시간 스텝에도 의존.
-- Vanishing / exploding gradient 문제 때문

###4. RNN 확장 모델
- Bidirectional RNN : 이후 시간 스텝에서 들어오는 입력값에도 영향 받음
- Deep (Bidirectional) RNN : 매 시간 스텝마다 여러 layer가 있는 Bidirectional RNN
- LSTM : hidden state 계산 방식이 다름. RNN의 뉴런 대신 메모리 셀을 이용하여 계산. ( 제일 많이 쓰임)


####출처 : [Recurrent Neural Network (RNN) Tutorial - Part 1](http://aikorea.org/blog/rnn-tutorial-1/)
