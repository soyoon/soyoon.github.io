---
layout: post
title:  "음성신호를 학습데이터로 만들기"
date:   2017-05-04 18:30:00
author: Soyoon
categories: ASR
tags: SignalProcessing
---

# 음성신호를 학습 데이터로 만드는 과정 용어 정리

Wav 파일의 아날로그 신호를 디지털 신호로 변경하고, 이 신호를 어떻게 Feature 추출하여 벡터화 하는지?
컴퓨터공학 학부 지식으로는 알기 어려운 내용이라 연휴 쉬는 틈에 MFCC , Filter bank, windowing 등의 용어를 정리해보려고 합니다.

## Mel Frequency Cepstral Coefficient (MFCC) tutorial
[MFCC tutorial 참고](http://www.practicalcryptography.com/miscellaneous/machine-learning/guide-mel-frequency-cepstral-coefficients-mfccs/)

자동 음성 인식 시스템의 첫번째 단계는 feature를 추출하는 겁니다. 즉, 언어 콘텐츠를 식별하고, 다른 것들은(background noise, emotion 등의 정보를 담은) 제거하는 오디오 신호를 식별하는겁니다.
Speech를 이해하는데 중요한 점은 인간이 만든 소리는 혀, 치아 등을 포함하는 성도(성대에서 입술 또는 콧구멍에 이르는 통로)의 모양에 의해 결정된다라는 것입니다.
이 모양은 소리가 어떻게 나오는지를 결정합니다. 만약 우리가 성도의 모양을 정확히 확인할 수 있다면 우리는 생성되는 음소의 정확한 실현을 할 수 있게 될 겁니다.
성도의 모양은 그 자체가 단시간 파워 스펙트럼의 포락선을 나타내고, MFCC는 이 포락선을 정확히 나타내는 역할을 합니다. 이 페이지는 MFCC의 짧은 튜토리얼을 제공할 겁니다.

Mel Frequency Cepstral Coefficents (MFCCs)는 자동 음성 인식, 화자 인식에서 널리 쓰이는 feature 입니다. MFCCs는 1980년대 Davis and Mermelstein 에 의해 소개되었고 지금도 계속 사용되고 있습니다.
MFCCs의 소개 전에는 Linear Prediction Coefficients (LPCs) 와 Liniear Prediction Cepstral Coefficients (LPCCs) 가 automatic speech recognition (ASR) 의 주 feature 였습니다. 특히 HMM의 classifier 에서 많이 사용되었습니다.
이 페이지에서는 MFCCs의 주요 특징인 왜 MFCC는 ASR을 위한 좋은 feature를 만드는지, 어떻게 MFCC를 구현하는지 에 대하여 살펴볼 것입니다.

### Steps at a Glance

1. Frame the signal into short frames.
2. For each frame calculate the periodogram estimate of the power spectrum.
3. Apply the mel filterbank to the power spectra, sum the energy in each filter.
4. Take the logarithm of all filterbank energies.
5. Take the DCT of the log filterbank energies.
6. Keep DCT coefficients 2-13, discard the rest.


### Why do we do these things?

우리는 조금 더 천천히 단계들을 보고 왜 각 단계들이 필요한지 설명할 겁니다.

오디오 신호는 끊임없이 변합니다. 이를 단순화하기 위해서 우리는 짧은 시간의 오디오 신호는 많이 변하지 않는다고 가정합니다.
( 우리가 변하지 않는다고 말하는 것의 의미는 통계적으로 고정되었다는 것을 의미합니다. 물론 샘플은 짧은 시간에도 끊임없이 변합니다.)
이것이 우리가 신호를 20-40ms 의 프레임으로 만드는 이유입니다.
만약 프레임이 더 짧다면 우리는 신뢰할만한 스펙트럼 추정을 얻을 수 있는 충분한 샘플을 가질 수 없고, 프레임이 더 길다면 신호는 프레임 안에서 너무 많이 변합니다.


다음 단계는 각 프레임의 파워 스펙트럼을 계산하는 것 입니다. 이 것은 인간의 들어오는 소리의 주파수에 따라 다른 부분이 진동하는 cochlea (귀에 있는 기관)을 보고 고안 되었습니다.
cochlea 가 진동하는 위치에 따라 다른 세포들이 자극되어 뇌에 특정 주파수가 있다고 알립니다. 우리의 periodogram (신호처리 용어로, estimate of the spectral density of a signal) estimate 는 어떤 주파수가 이 프레임에 있는지 식별하는 cochlea와 유사한 역할을 합니다.


periodogram spectral estimate는 여전히 ASR에 필요하지 않은 많은 정보를 담고 있습니다. 특히 cochlea는 두개의 가까운 주파수의 차이를 구별하지 못합니다. 이 점은 주파수가 올라갈 수록 더 명백해 집니다.
이러한 이유로 우리는 다양한 주파수의 영역에 얼마나 에너지가 있는지 아이디어를 얻기 위해 periodogram 뭉치를 모아서 더했습니다.
이것은 Mel filterbank 로 실행되었습니다.
첫번째 필터는 매우 좁고 얼만큼의 에너지가 0 Hertz 근처에 있는지를 나타냅니다.
주파수가 높아질 수록 필터는 넓어지게 되고 우리는 변동에 대한 걱정을 줄일 수 있습니다.
우리는 대략적으로 얼만큼의 에너지가 각 스팟에 등장하는지에 관심이 있습니다.
Mel scale은 정확이 어떻게 우리 filterbank의 간격을 배치할지, 얼마나 넓게 만들지를 알려줍니다. 아래에 어떻게 공간을 계산하는지 보십시오.


filterbank 에너지를 얻게되면, 여기에 로그를 적용시킵니다. 이는 인간의 듣기( 우리는 선형 스케일로 소리의 강도를 듣지 않습니다.)에 영향을 받았습니다.
일반적으로 감지된 소리의 볼륨을 2배로 얻기 위해 우리는 소리에 들어있는 에너지의 8배를 필요로합니다. 이것은 만약 소리가 처음에 크면, 에너지에서 큰 변동은 그렇게 많이 다르게 들리지 않는다는 것을 의미합니다.
이런 압축 기능은 우리의 feature를 인간이 실제 듣는것에 가깝게 매치시킵니다. 왜 세제곱이 아닌 로그일까요? 로그는 cepstral mean subtraction 이라는 채널 노멀라이제이션 기술을 쓸 수 있게 해주기 때문입니다.


마지막 단계는 log filterbank 에너지의 DCT를 계산하는 것입니다. 이 것을 하는 두가지 이유가 있습니다.
우리의 filterbank는 모두 겹치기 때문에, 필터뱅크의 에너지는 서로 꽤나 상관관계가 있습니다.
DCT 상관관계를 에너지들의 해제합니다. 이것은 diagonal covariance matrices(대각 공분산 행렬?)가 HMM 분류기처럼 feature를 만드는데 사용된다는 것을 의미합니다.
하지만 알아야 되는건 26개 중 12개의 DCT coefficients(계수) 만 유지됩니다.
DCT 계수가 높을수록 필터뱅크 에너지의 빠른 변화를 나타내고 이것은 ASR의 성능의 하락을 나타내기 때문입니다.
그래서 우리는 그것들을 떨어트려서 작은 개선을 얻는 것입니다.


### What is the Mel scale?

Mel scale은 깨끗한 톤의 주파수 또는 피치를 실제 측정된 주파수로 알아차리는 것과 관련이 있습니다.
인간은 높은 주파수에서 보다 낮은 주파수에서의 피치(소리의 높이)의 작은 변화를 더 잘 구별합니다. 이 스케일을 포함하는 것은 우리의 피쳐를 더 인간의 청각과 비슷하게 매치시켜 줍니다.

주파수에서 Mel scale로 변환하는 공식은 아래와 같습니다.


M(f) = 1125 ln(1 + f/700)


Mel scale에서 주파수로 바꾸는건 다음과 같습니다.


M^(-1)(m) = 700( exp(m/1125) - 1)



### Implementation steps

16kHz 의 샘플 음성 신호로 시작해봅시다.

1. 신호를 20-40ms 의 프레임으로 만듭니다. 25ms 가 기본입니다. 이것은 16kHz 신호의 프레임 길이는 0.025*16000 = 400 samples 라는 것을 의미합니다.
프레임 단계는 보통 프레임간 겹침을 허용하는 10ms (160 samples) 과 비슷합니다. 처음의 400 샘플 프레임은 샘플 0 에서 시작하고 다음 400 샘플은 샘플 160 등 에서 시작합니다. 음성 파일의 끝에 도착할때까지 반복됩니다.
만약 음성 파일이 짝수개의 프레임으로 나눠지지 않는다면 0 으로 된 pad를 넣어 나눠질 수 있게 합니다.

그 다음 단계는 매 한 개의 프레임에 적용됩니다. 12 MFCC 계수의 한 세트가 각 프레임별로 추출됩니다. 개념은 잠시 제쳐두고 : time domain signal 을 s(n) 이라고 합니다. 프레임이 만들어지면 si(n) 이라 하고
n은 1-400 의 범위를 갖습니다. ( 프레임이 400 샘플이라 가정 ) 그리고 i는 프레임수를 의미합니다.
DFT 복잡도를 계산할때 Si(k) 를 얻습니다. i 는 타임 도메인 프레임에 대응하는 프레임 번호입니다. pi(k)는 프레임 i의 파워 스펙트럼입니다.


2. 프레임의 Discrete Fourier Transform 을 얻기위해 아래의 공식을 적용한다:
