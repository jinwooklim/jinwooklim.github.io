---
title:  "Accelerated Training with ML Compute on M1-Powerd Macs"
excerpt: ""
toc: true
toc_sticky: true
toc_label: "Table"
categories:
  - Conference
tags:
  - Tensorflow
  - Mac
  - M1
comments : true
---

# Accelerated Training with ML Compute on M1-Powerd Macs



이번에 작성할 내용은 Apple ML Compute에 관한 내용입니다. NIPS 2020에 참여하면서 듣게된 내용과 구글링한 내용으로 간략하게 소개하려고 합니다.

&nbsp;

## ML Compute

Apple에서 소개하는 [ML Compute](https://developer.apple.com/documentation/mlcompute)는 Apple의  Deep learning accelerating framework 입니다.

기존의 [BNNS](https://developer.apple.com/documentation/accelerate/bnns), [Metal Performance Shaders](https://developer.apple.com/documentation/metalperformanceshaders) 를 이용하여 가속화 하는 방식으로 동작합니다. 즉, ML Compute의 구성은 상부는 Compute Engine Interface, 하부는 CPU 파트(Backend, BNNS), GPU 파트(Multi-Core Backend, MPS)로 이루어져 있습니다. 발표자에 따르면 [Apple M1 chip](https://www.apple.com/kr/newsroom/2020/11/apple-unleashes-m1/)을 사용하게 되면서, CPU와 GPU를 Unified memroy에서 함께 동작시키기 때문에 성능의 향상(Vetor processing, GPU) 및 저전력 효율성(Energy efficient, CPU)을 더욱 가질 수 있다고 합니다.

한편, ML Compute는 Swift, Objectvie-C API로 제공되기 때문에, 쉽게 Application level에서 모델의 학습을 시도할 수도 있고, CPU/GPU에 의도에 맞는 작업을 할당할 수도 있습니다.

&nbsp;

## TensorFlow on Mac

Apple은 ML Compute를 활용하여 Tensorflow를 가속화했음을 발표했습니다 ([mac optimized fork of tensorflow 2.4](https://github.com/apple/tensorflow_macos)).

**CPU** : ML Compute (BNNS) + Fallback to default Tensorflow's dispatch for unsupported ops.

**GPU** : ML Compute (MPS) + Fallback to default Tensorflow's dispatch for unsupported ops.

따라서, 앞에서 설명한 ML Compute으로 Tensorflow operation의 각 suboperation을 대체하는 방식으로 구현이 된 것 같습니다.

아래는 지원이 확인된 Tensorflow operation입니다.

**Layers** : tf.keras, tf.nn, tf.math,

**Addons** : GroupNorm , Multi-Head Attention, gelu, hardshrink, softshrink, tanhshrink

&nbsp;

## M1 + Mac-optimized Tensorflow 2.4 (GPU) vs Intel Based Tensorflow 2.3 (CPU)

발표에 사용된 그림을 인용하면 좋겠으나, 그러지 못하는 관계로 공개된 그림을 첨부합니다.

![Image1](/assets/images/posts/2020-12-15-ML_compute_for_mac/macbook-pro-tensorflow-graph.png){: .align-center}

그림에서의 회색-주황색이 해당하는 아래 표의 값입니다.

| Model / Unit : (Seconds/batch) | 13-inch Macbook Pro (times) | Mac Mini (times) |
| ------------------------------ | --------------------------- | ---------------- |
| Style Transfer                 | 5.1x                        | 4.5x             |
| Densenet 121                   | 7.0x                        | 6.7x             |
| CycleGAN                       | 5.6x                        | 5.3x             |
| ResNet50-V2                    | 6.2x                        | 5.1x             |
| MobileNet V3                   | 3.8x                        | 3.3x             |
| ResNet50-V2 FineTuning         | 4.3x                        | 3.4x             |

&nbsp;

발표에서 공개된 내용입니다.

| Model / Unit : Watts   | M1 + Mac-optimized Tensorflow 2.4 (GPU) (times) |
| ---------------------- | ----------------------------------------------- |
| *Style Transfer*       | *~~8.6x or 3.7x~~*                              |
| Densenet 121           | 3.5x                                            |
| CycleGAN               | 8.1x                                            |
| *ResNet50-V2*          | *~~8.6x or 3.7x~~*                              |
| MobileNet V3           | 4.0x                                            |
| ResNet50-V2 FineTuning | 7.4x                                            |

*Style Transfer와 ResNet50이 오기입 되어 있어 보이는 Apple의 발표 자료 문제로, 해당 부분은 공란으로 두었습니다.*
*두 값은 8.6x 또는 3.7x 입니다.*

표를 확인하면, GPU을 사용했기 때문에 비교로 사용되는 모델의 구조상 당연히 속도가 빠를 수 있다고 생각되지만, 전력 소모량을 고려해보면 CPU보다도 적은 전력으로 높은 성능을 내는 것을 확인 할 수 있습니다.

&nbsp;

## Mac-optimized Tensorflow 2.4 (GPU) vs Intel Based Tensorflow 2.3 (CPU)

![Image1](/assets/images/posts/2020-12-15-ML_compute_for_mac/mac-pro-tensorflow-graph.png){: .align-center}

Intel based Mac에서도 성능 향상이 있다고 하지만, 확인이 어렵습니다 (CPU vs GPU 이기 때문입니다).

&nbsp;

## Optimize Tensorflow using ML Compute

ML Compute에서 Tensorflow를 Optimize하는 방법은 아래와 같습니다.

**Graph Mode Tensorflow**에서는 전체 Graph를 그리고, Graph에서 대체 가능한 Operation을 Replace 하는 방식을 사용했습니다. 그리고 대체된 여러개의 op들을 하나의 sub op로 압축하는 방식을 이용했습니다. (MLC Operations **→** Single MLCSubgraphOp)

**Eager Mode Tensorflow**에서는 각 layer가 매순간 생성되는 방식이기 때문에, Corresponding layer를 생성해주고 바로 대체하는 방식을 사용했습니다. (MLCConv2D **→** MLCConv2DEagerOp **→** MLCConvolutionLayer)



