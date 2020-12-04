---
title:  "Tensorflow Extended (TFX) 정리"
excerpt: "Tensorflow Extended를 알아보자"
toc: true
toc_sticky: true
toc_label: "Table"
categories:
  - Development
tags:
  - Tensorflow 
comments : true
---

## 1. Intro
---
까마득한 지난 글에서 **Tensorflow Extended** (이하 **TFX**)에 대해서 언급하였다.  
아직 많은 리뷰가 없어서 Youtube의 TFX 관련 내용, [TFX Homepage](https://www.tensorflow.org/tfx) , 
[TFX Github Link](https://github.com/tensorflow/tfx) 를 기반으로 보고 기본 예제들과 회사에서 진행하는 과제로 얻은 것들을 정리하였다.  

TFX에대한 내용을 시작하기 앞서, ML production에 대한 말을 적으려한다.  
Machine Learning의 최신의 논문들로 부터 나오는 코드들은, 다양한 분야에 사용하기 위해서는 많은 작업들이 필요하다. 특히 기업의 Production 수준으로오면 더 그렇다.
따라서, Google에서는 발생할 문제들을 Modern Software 관점으로 해결할 생각을 하고 있었고, 이를 **NIPS 2015 : "Hidden Technical Debt in Machine Learning Systems"** 논문을 통해 알수 있다.

즉, Google은 이미 ML ecosystem에 대한 생각을 하고 있었고, Tensorflow와 같은 ML 관련 Software를 공개하며 준비를 진행해왔다. 
그리고 2019년 ML ecosystem에서 ML pipeline을 담당할 TFX를 오픈소스로 공개하면서, Google에서 생각하고 준비했던 **Modern Software Development** 기술과 **Machine Learning Development**의 결합을 보여주게 되었다.
즉, ML pipeline 이며 TFX의 목적은 아래 Machine Learning Development의 7가지 요소들과 Modern Software Development의 9가지 요소를 결합하는 것으로 볼수 있다.

### 1.1. Machine Learning Development
---
1. Labeled data
2. Feature space coverage
3. Minimal dimensionality
4. Maximum predictive data
5. Fairness
6. Rare conditions
7. Data lifecycle management

### 1.2. Modern Software Development
---
1. Scalability
2. Extensibility
3. Configuration
4. Consistency & reproducibility
5. Modularity
6. Best practices
7. Testability
8. Monitoring
9. Safety & security

&nbsp;
## 2. Tensorflow Extended (TFX)
---
TFX는 위에서 말한 ML ecosystem을 수행하기 위한 여러 구성요소들의 데이터 흐름을 정의하고 이를 DAG 형태의 파이프라인으로 표현한 것이다.
이러한 역할을 해주는 다른 프레임워크들도 있으므로, TFX는 Machine learning pipeline framework 중에 하나인 것으로 보면된다.

먼저, TFX는 **Artifact**라는 개념을 사용하고 있다. Artifact는 Model, data, evaluation result 등을 말한다. Artifact는 
Components들 간에 전달되는 데이터의 단위이고 파이프라인의 실행마다 RDBMS에 메타데이터로서 저장된다.
메타데이터로서 모든 것들을 저장하는 이유는 복잡한 구성을 가지고 긴 시간을 거치는 파이프라인에서 문제가 발생했을 경우, 정확하게 Debugging을 하기 위해서이다.

또한, TFX는 아래 8가지의 **Components**를 가지고 있고, 각각의 component는 ML production에서 필요한 기능들을 실행시켜주는 부분이다.
마찬가지로, Component에 관한 내용(Driver, Publisher)도 메타데이터에 저장되게 되어 파이프라인의 재실행시 빠른 Processing이 가능해진다.
Component는 {Driver, Executor, Publisher} 3가지 구성요소로 이루어져 있는데,
대부분의 TFX 사용자들은 현재 Executor 부분에 실행할 Code를 넣어 파이프라인을 수행시킨다고 보면 된다.  

![TFX pipeline_example_img](/assets/images/posts/2020-03-17-TF_2020_summit/001.png){: .align-center}  

### 2.1. Components
---
1. **ExampleGen** : 입력 데이터를 모으고 Train, Val, Test로 나누는 역할을 하는 파이프라인의 입력 부분을 구성함. 
(TFX 0.15.x 버전까지는 TFrecords만 입력으로 받고 있음.)  
2. **StatisticsGen** : 분할된 Train, Val, Test데이터에 대한 통계치를 계산해주는 부분 
(Label 편향 등 여러 요소를 볼 수 있어서 좋음, TFDV와 함께 쓰일 수 있음)  
3. **ExampleValidator** : 데이터셋에서 이상치나 결측값을 찾아내는 부분 (이상치의 경우 값의 수정, 결측값의 경우 제거할 수 있게 한다.)
4. **SchemaGen** : 통계치를 검사하고, 데이터 스키마를 작성하는 부분 
(예를 들면, 통계치에 따른 문제들을 보거나 입력 데이터가 INT 또는 Float으로 들어오는지 확인)
5. **Transform** : 데이터셋에 대한 Feature Engineering을 수행하는 부분. (이미지 데이터의 경우 전처리를 수행하는 부분)  
6. **Trainer** : 모델을 학습하는 부분 (TF 2.0의 경우 tf.Keras, Estimator를 사용함)
7. **Evaluator** : 훈련된 모델에 대한 분석을 하고, 모델의 유효성을 검사하는 부분
8. **Pusher** : REST API로 모델을 배포하고 싶으면 TF serving, 모바일이나 임베디드에 배포하고 싶으면 TF Lite를 사용하여 배포하는 부분.

### 2.2. Example code
---
1. [TFX MNIST Example Link](https://github.com/tensorflow/tfx/tree/master/tfx/examples/mnist)
2. 자체 코드 공개는 가능해지면 ...

&nbsp;
## 3. Extra
---
TFX youtube 영상을 보다가, Google에서 ML Business에대해 소개해주었다.  
흔히 대부분의 ML이 단순히 연구로만 접근되고 있는 경우들이 많이 있다. 따라서, 이는 대게 모델 성능을 분석하는 것에 치중되어 있는데,
현실적인 ML에서는 계속적인 ML 모델 및 시스템에 대한 심층적인 분석(**Accuracy, Data, Business**)이 지속적으로 필요하다고 한다.

즉, ML business의 문제는 크게 **3가지 {비지니스, 좋지 않은 데이터, 모델 자체의 문제}**로 이루어진다고 한다.
(특히, Business에 문제가 생기는 것은 사업을 할 때 깔아놓는 **전제**가 무너졌기 때문이라고 한다.)

따라서 ML business 과정에서 문제가 발생했을 때, 해결하기 위해 생각해야 하는 것들을 정리해주었다.

### 3.1 Solution
---
1. 문제가 생기면, 숨겨진 문제를 찾아야한다 **&rarr;** 항상 데이터가 먼저이다.
2. 비지니스, 삶. 모든 것은 변화한다 **&rarr;** 전제에 대한 추가/삭제는 필수이다.
3. 모델의 성능을 집중적으로 조사한다.
4. Data에 대한 이해는 필수이다.
5. 모델이 항상 정확하진 않다. **&rarr;** 비용을 고려해야한다.



