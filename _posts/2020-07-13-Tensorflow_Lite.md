---
title:  "Tensorflow Lite"
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
**Tensorflow Lite (TF Lite)** 

Tensorflow Lite (이하 tflite)는 휴대용 기기 및 IoT 기기에서 **Tensorflow(TF)** 모델을 실행할 수 있도록하는 library 입니다.

tflite는 위의 기능을 제공하기 위해서 크게  두 가지 요소로 구성되어있습니다.

* **Tensorflow Lite Interpreter** 
* **Tensorflow Lite Converter**

이 글에서는  Tensorflow Lite Interpreter, Tensorflow Lite Converter 및 Python, Android 에서의 빠른 tflite 시작 예제에 대해서 알아보겠습니다.

>  현재, tflite는 Tensorflow의 1.x에서의 2.x 버전 변경으로 인하여 각 API들이 추가 및 변경되고 있는 상황입니다. 사용시 버전에 대한 확인이 필요합니다.

&nbsp;

## 2. Tensorflow Lite Interpreter 
---

[Reference link](https://www.tensorflow.org/lite/guide/inference?hl=ko)

위 링크에서도 볼 수 있듯이, TF Lite에서의 Inference는 TF Lite Interpreter를 통해서 이루어집니다.

> "*Interpreter*는 빠른 속도를 위해 Static graph와  (less-dynamic) Memory allocator를 이용하며, 따라서 load, initialization에서 execution latency를 최소화 할 수 있다고 합니다."

### **2.1. Concepts**
---
TF Lite의 Inference의 과정은 아래와 같습니다. (상기 링크 참조)

> TensorFlow Lite inference typically follows the following steps:
>
> 1. **Loading a model**
>
>    You must load the `.tflite` model into memory, which contains the model's execution graph.
>
>    * tflite 모델 file을 memory로 올립니다.
>
> 2. **Transforming data**
>
>    Raw input data for the model generally does not match the input data format expected by the model. For example, you might need to resize an image or change the image format to be compatible with the model.
>
>    * 흔히, 전처리라고 불리는 부분입니다. Computer vision에서의 Object Detection을 예로 들자면,  일반적인 경우, 카메라로부터 영상을 바로 입력 받아서 동작하지 못합니다 (이미지의 크기, 형식 등이 정해진 모델의 입력의 형식과 다르기 때문). 따라서 이러한 문제를 해결하는 부분입니다.
>
> 3. **Running inference**
>
>    This step involves using the TensorFlow Lite API to execute the model. It involves a few steps such as building the interpreter, and allocating tensors, as described in the following sections.
>
>    * interpreter build를 수행합니다.
>    * 전처리된 영상을 Tensor Object로 변환 및 할당하여 tflite 모델에 입력으로 주고 실행합니다.
>
> 4. **Interpreting output**
>
>    When you receive results from the model inference, you must interpret the tensors in a meaningful way that's useful in your application.
>
>    For example, a model might return only a list of probabilities. It's up to you to map the probabilities to relevant categories and present it to your end-user.
>
>    * 후처리라고 볼 수 있는 부분입니다. interpreter로 부터 얻어진 tensor를 우리의 목적에 맞게 (ex: Object detection) 해석하고, 이용해야합니다.

### **2.2. Supported operations**
---
[Operation set link](https://www.tensorflow.org/mlir/tfl_ops?hl=ko)

[TF lite Roadmap](https://www.tensorflow.org/lite/guide/roadmap?hl=ko)

Tensorflow와 달리 Tensorflow Lite는 아직 대부분의 연산 및 기능이 지원되지 않고, 기초적인 부분들 위주로 지원되고 있습니다. Tensorflow에서는 2.x 버전에 들어서 TF lite에 대해서 많은 기능 지원이 이루어질 것이라고 했습니다.

### **2.3. Supported platforms**
---
TF lite는 Android, iOS, Linux, micro-controller 등을 지원합니다. (C++, Java, Python)

예제

- [TF lite Java example](https://www.tensorflow.org/lite/guide/inference?hl=ko#load_and_run_a_model_in_java)
- [TF lite Swift example](https://www.tensorflow.org/lite/guide/inference?hl=ko#load_and_run_a_model_in_swift)
- [TF lite Objective-C example](https://www.tensorflow.org/lite/guide/inference?hl=ko#load_and_run_a_model_in_objective-c)
- [TF lite C++ example](https://www.tensorflow.org/lite/guide/inference?hl=ko#load_and_run_a_model_in_c)
- [TF lite Python example](https://www.tensorflow.org/lite/guide/inference?hl=ko#load_and_run_a_model_in_python)
- [TF lite Custom operator](https://www.tensorflow.org/lite/guide/inference?hl=ko#write_a_custom_operator)

공식 예제가 주어지지만, 그렇게 친절하지 않고 자료가 많지 않으므로 자신에게 익숙한 플랫폼, 언어로부터 시작하는게 검증 및 구현에 편합니다.

&nbsp;

## 3. Tensorflow Lite Converter
---
[Reference link](https://www.tensorflow.org/lite/convert/index)

Tensorflow Lite Converter는 Tensorflow Model을 **FlatBuffer** file (.tflite)로 변환하는 것을 돕습니다. 공식 문서에서는 이 부분에 들어서 Tensorflow 2.x와 Tensorflow 1.x에 대른 부분에 대해서 확인하라고 강조합니다. 왜냐하면, TF 2.x 버전에 들어서 SavedModel 부분에 변경점이 생겼고 (for keras, TF 2 구조의 변경), TF 2.x의 concrete function 때문에 그렇습니다.

> **FlatBuffer** is an efficient open-source cross-platform serialization library. It is similar to [protocol buffers](https://developers.google.com/protocol-buffers), with the distinction that FlatBuffers do not need a parsing/unpacking step to a secondary representation before data can be accessed, avoiding per-object memory allocation. The code footprint of FlatBuffers is an order of magnitude smaller than protocol buffers.

### **3.1. Tensorflow Lite Converter V1**
---
[Reference link](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/lite/g3doc/r1/convert/index.md)

Converter V2와 비교시 그림을 통해 비교하면 좋습니다. Converter V1의 기능은 모두 V2에서 지원됩니다.

### **3.2. Tensorflow Lite Converter V2**
---
[Reference link](https://www.tensorflow.org/lite/convert/index)

TF 2.2 버전에 이르러서, 새로운 Converter (V2)가 default converter로 제공됩니다.

이로 인해, 몇가지 장점이 생겼습니다.

* Mask R-CNN, BERT 등의 지원
* TF 2.x에서의 functional control flow 지원
* 오류 발생시, 변환중에 오류 node 노출 (기존에는 그냥 오류가 발생하고 끝)
* ML를 위한 새로운 컴파일러 기술인 [MLIR](https://mlir.llvm.org/)을 제공한다고 합니다. ([TF MLIR](https://www.tensorflow.org/mlir?hl=ko))
* Unknown dimension가 입력인 모델에 대한 support
* 기존의 모든 converter 기능 지원 (모두 호환 !)

&nbsp;