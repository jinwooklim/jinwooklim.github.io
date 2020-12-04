---
title:  "PyTorch-Flask with albumentations 정리"
excerpt: "PyTorch Inference, albumentations, Flask를 알아보자"
toc: true
toc_sticky: true
toc_label: "Table"
categories:
  - Development
tags:
  - PyTorch
  - Albumentations
  - Flask
comments : true
---

## 1. Intro
---
이번 포스트에서는 구글링을 통해서 쉽게 알수 있는 3가지 예제들을 섞어서 하나의 예제로 진행해보려 한다. 

* Pytorch 모델을 학습하여 저장하고 불러와서 사용하는 것
* albumentations을 사용하는 것
* Flask기반의  Server-Client간의 REST를 통해 Inference 하는 것

  

&nbsp;
## 2. Train & Load PyTorch Model
---
[코드 참조](https://github.com/pytorch/examples/tree/master/mnist)

코드는 간단하게 PyTorch에서 제공하는 mnist 예제를 이용하였다.



`main.py`를 실행시키면 train, test용 mnist dataset을 다운로드 받는다. 그리고 학습을 진행하고, 정해놓은 iteration이 끝나면 모델을  **'mnist_cnn.pt'** 파일로 저장하게 된다.

 * ```python
   train_loader = torch.utils.data.DataLoader(
       datasets.MNIST('../data', train=True, download=True,
                      transform=transforms.Compose([
                          transforms.ToTensor(),
                          transforms.Normalize((0.1307,), (0.3081,))
                      ])),
       batch_size=args.batch_size, shuffle=True, **kwargs)
   test_loader = torch.utils.data.DataLoader(
       datasets.MNIST('../data', train=False, transform=transforms.Compose([
           transforms.ToTensor(),
           transforms.Normalize((0.1307,), (0.3081,))
       ])),
       batch_size=args.test_batch_size, shuffle=True, **kwargs)
   ```

 * ```python
   model = Net().to(device)
   optimizer = optim.Adadelta(model.parameters(), lr=args.lr)
   
   scheduler = StepLR(optimizer, step_size=1, gamma=args.gamma)
   for epoch in range(1, args.epochs + 1):
       train(args, model, device, train_loader, optimizer, epoch)
       test(model, device, test_loader)
       scheduler.step()
   ```

 * ```python
   torch.save(model.state_dict(), "mnist_cnn.pt")
   ```
   



저장된 model을 불러오는 코드는 아래와 같다.

* ```python
  model = Net()
  model.load_state_dict(torch.load('mnist_cnn.pt'), strict=False)
  model.eval()
  ```

  

  

&nbsp;
## 3. Use albumentations
---

도대체 뜬금없게, PyTorch와 Flask를 사용하는 예제에서 **albumentations** 라는 것을 소개하는지 의문이들 수 있다. (~~두서없이 글을 쓰는 나 때문이다~~)

[albumentations](https://github.com/albumentations-team/albumentations) 은 Torchvision, imagaug, keras, augmentor 등 Image augmentation package들과 같은 augmentation을 제공하는 package이다. 하지만 이들 보다 훨씬 빠르다 ! [링크 논문의 Table 1 참조](https://arxiv.org/pdf/1809.06839.pdf)

간략하게 말하자면, albumentations는 torchvision에서 사용되는 transformer 코드 사용 형태를 그대로 유지하면서, 같은 결과물을 만드는 속도가 훨씬 빠르다.

따라서, 대부분의 vision task deep learning model의 input으로 들어가는 image들이 augmentation (preprocess)를 거쳐야 하는 상황이기에, model의 train과정에서 augmentation or preprocess speed를 x3 가까이 향상 시킬 수 있다. 그리고 inference의 경우에는 normalization, resize 같은 preprocess에서 처리 속도를 향상 시킬 수 있다.
**즉, Server-Client 구조에서 client가 보낸 image에 대하여 server가 preprocess를 하고, model에 input으로 넣는 과정을 빠르게 진행할 수 있게 된다.** (그렇다고 너무 맹신하지 말고, 효율적인 코드를 생각하자 !)



unsigned int8 image가 주어진다고 할 때의 PyTorch(Torchvision), albumentations 두 가지 경우에 대한 Flask function

* ```
  # Pytorch transformer
  normalize = transforms.Compose([
      transforms.Resize([28, 28]),
      transforms.ToTensor(),
      transforms.Normalize(mean=(0.1307,), std=(0.3081,)), # Just normalize
  ])
  
  
  # pytorch transformer
  @app.route('/inference/pytorch', methods=['POST'])
  def inference_pytorch():
      start = time.time()
      data = request.json # data is list
      data = np.array(data['images']) # data is list -> uint8 ndarray -> Image
      data = data / 255.
      data = data.astype(np.float32)  # Cast uint8 -> float32
      data = Image.fromarray(data) # torchvision resize needs Image object type
      
      data = normalize(data)
  
      result = model.forward(
          data.unsqueeze(0).float()
      )
      end = time.time()
      print(f'time elapsed: {end - start}')
      return str(result.argmax().item())
  ```

* ```
  # albumentations transformer
  normalize2 = albumentations.Compose([
      albumentations.Resize(height=28, width=28),
      albumentations.Normalize(mean=(0.1307,), std=(0.3081,)), # Divide pixel values by 255 = 2**8 - 1 and normalize
      albumentations.pytorch.ToTensorV2(),
  ])
  
  # albumentations transformer
  @app.route('/inference/albumentations', methods=['POST'])
  def inference_albumentations():
      start = time.time()
      data = request.json # data is list
      data = np.array(data['images']) # data is list -> uint8 ndarray
      data = data.astype(np.float32) # Cast uint8 -> float32
      data = np.expand_dims(data, 2) # albumentations needs (H, W, C) shape#
      
      data = normalize2(image=data)['image']
  
      result = model.forward(
          data.unsqueeze(0).float()
      )
      end = time.time()
      print(f'time elapsed: {end - start}')
      return str(result.argmax().item())
  ```

  

  

&nbsp;
## 4. Use Server-Client communication
---

세 번째 장에서는 Flask 기반의 Server와 python으로 간단하게 작성한 client의 REST 통신으로 첫 번째 장에서 보였던, mnist classification model에게 이미지를 보내고, 응답을 받는 예제를 소개하려 한다.

* ```python
  image = io.imread('./test.png')
  image = rgb2gray(image) # test.png is a color image, so we need to convert it.
  image = image * 255.
  image = image.astype(np.uint8)
  ```

* ```python
  headers = {'Content-Type': 'application/json'}
  address = "http://127.0.0.1:2431/inference/pytorch"
  data = {'images': image.tolist()}
  result = requests.post(address, data=json.dumps(data), headers=headers)
  print(str(result.content, encoding='utf-8'))
  
  address2 = "http://127.0.0.1:2431/inference/albumentations"
  data2 = {'images': image.tolist()}
  result2 = requests.post(address2, data=json.dumps(data2), headers=headers)
  print(str(result2.content, encoding='utf-8'))
  ```

* ```
  $ python client.py
  7
  7
  ```

  

