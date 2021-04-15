---
title:  "Tensorflow Lite (2)"
excerpt: ""
toc: true
toc_sticky: true
toc_label: "Table"
categories:
  - Development
tags:
  - Tensorflow
  - Tensorflow Lite
comments : true
---

## 1. Intro
---
이전의 글 ([Tensorflow Lite](https://jinwooklim.github.io/development/Tensorflow-Lite/))에서는 기본적인 내용들과 예제들을 정리했습니다.

이번 글에서는 Tensorflow Lite의 조금 더 심화되는 내용들을 다루려합니다.

&nbsp;

## 2. Flat Buffer
---
Tensorflow 에서는 학습한 모델을 binary로 저장하기 위해서 **Protocol Buffer**를 사용합니다. 반면, Tensorflow Lite에서는 Tensorflow Lite Converter를 이용해서 모델을 저장하기 위해서 **Flat Buffer**를 사용합니다

**Protocol Buffer**는 이미 구글에서 진행중인 많은 프로젝트에서 사용되고 있는데, 왜 Google은 Tensorflow Lite에서는 **Flat Buffer**를 사용했을까요 ?

이는 알아보기 위해 찾아보았더니, **Flat Buffer**가  object를 사용하기 위해서 전체 데이터를 deserialize (parsing / unpacking) 할 필요가 없고, Flexible한 Schema를 가지고 있으며, 좀 더 적은 메모리를 사용하기 때문이라고 설명되어 있었습니다.

하지만, **Protocol Buffer** 대비 단점으로는 **Flat Buffer**를 사용하기 위해서 사용되는 코드의 양이 **Protocol Buffer**에 비해 많고, **JSON**과 같이 자주 사용되는 형태로 변환이 지원되지 않는 문제가 있었습니다.

따라서, 학습된 모델에 대해서는 모델의 값이 다른 형태로 변경될 이유가 없고, Mobile 환경으로 빠르게 변환되고 동작하면 되므로 Tensorflow Lite에서는 **Flat Buffer**를 사용한 것으로 보입니다.

&nbsp;

## 3. Quantization
---
Tensorflow Lite에서는 Edge device에서의 동작을 위한 모델 경량화 및 latency 단축이 필요해서 **Quantization**에 관한 이야기가 많습니다.

아래는 Model의 최적화 (Quantization 과정)을 위한 Flowchart 입니다.

![optimization](/assets/images/posts/2020-08-10-Tensorflow_Lite_2/optimization.jpg){: .align-center}

흔히, GPU를 이용하여 Model을 가속할 경우에는 float16 quantization을 많이 사용합니다 (GPU가 floating point를 사용하기 때문). 이는 Tensorflow로 부터  생성된 모델의 parameter 값은 float16으로 바꾸고, Model은 그대로 float32를 사용하는 것입니다.

> Have RepresentativeDataset? &rarr; Quantization이 잘 수행되었는지 확인할 용도의 Dataset
>
> Limit to Int8 ops ? 

위 처럼 라고 쓰여진 부분에서 flow  분기가 발생하는 것을 볼 수 있습니다. 이 부분들은 Parameter 또는 Model ops의 **32 bit floating point number**를 가장 가까운 **8 bit fixed point number**로 근사시키는 것입니다. 

이로 인해서 성능에는 손해를 보겠지만, Model이 작아지고 추론 속도가 증가하게 됩니다 (Edge TPU는 Interger를 사용하기 때문).  이 외에도 **Dynamic range quantization** 같은 다른 방법들이 있지만, HW에 의한 제한 같은 이유로 사용이 더 어렵습니다.

![quant_image](/assets/images/posts/2020-08-10-Tensorflow_Lite_2/quant_image.png){: .align-center}

float32 values &rarr; int8 values

&nbsp;

지금까지는 **Post-training quantization** 을 정리해 봤지만, 가지고 있는 모델이 크지 않은 경우이거나, 완전히 Edge device에 최적화된 성능의 모델을 올리기 위해서는 **Quantization-aware training (QAT)**을 사용해야합니다. 또한 Pruning, Distillation과 같은 방법들을 함께 사용하기도 합니다.

**QAT**를 수행할 때 실제로는 tranining graph 자체는 floating-point (ex: float32)에서 동작하지만, fixed-point의 결과를 내보내야 하기 때문에, 특수한 operation ( [tensorflow::ops::FakeQuantWithMinMaxVars](https://www.tensorflow.org/api_docs/cc/class/tensorflow/ops/fake-quant-with-min-max-vars))을 사용합니다. 이로 인해서, Quantization로 인한 손실이 학습에 포함되게 되는 것입니다.

![quant_train](/assets/images/posts/2020-08-10-Tensorflow_Lite_2/quant_train.png){: width="328" height="240"}{: .align-center}

&nbsp;

## Reference
---
1. [https://google.github.io/flatbuffers/](https://google.github.io/flatbuffers/)

2. [https://codeburst.io/json-vs-protocol-buffers-vs-flatbuffers-a4247f8bda6f](https://codeburst.io/json-vs-protocol-buffers-vs-flatbuffers-a4247f8bda6f)

3. [https://capnproto.org/news/2014-06-17-capnproto-flatbuffers-sbe.html](https://capnproto.org/news/2014-06-17-capnproto-flatbuffers-sbe.html)

4. [https://gompangs.tistory.com/entry/Flatbuffers-%ED%94%8C%EB%9E%AB%EB%B2%84%ED%8D%BC%EB%9E%80](https://gompangs.tistory.com/entry/Flatbuffers-플랫버퍼란)

5. [https://www.tensorflow.org/lite](https://www.tensorflow.org/lite)

6. [https://www.tensorflow.org/lite/performance/quantization_spec](https://www.tensorflow.org/lite/performance/quantization_spec)

7. [https://arxiv.org/pdf/1712.05877.pdf](https://arxiv.org/pdf/1712.05877.pdf)

8. [https://blog.tensorflow.org/2020/04/quantization-aware-training-with-tensorflow-model-optimization-toolkit.html](https://blog.tensorflow.org/2020/04/quantization-aware-training-with-tensorflow-model-optimization-toolkit.html)

9. [https://arxiv.org/pdf/1710.09282.pdf](https://arxiv.org/pdf/1710.09282.pdf)

   