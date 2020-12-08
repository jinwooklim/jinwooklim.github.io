---
title:  "Horovod 설치하기"
excerpt: "Tensorflow에 Horovod 사용하기"
toc: true
toc_sticky: true
toc_label: "Table"
categories:
  - Development
tags:
  - Tensorflow 
  - Horovod
comments : true
---

## 1. Intro
---
Distributed deep learning에 사용되는 ["Horovod"](https://github.com/horovod/horovod)에 대해 소개하고, 설치, 사용법을 공유하려고 합니다.  
Horovod는 아직 잘 지원되지 않는 Distributed computation을 간단한 코드 변경으로 잘 지원해주고 있습니다.
위 링크를 따라가서, 성능 향상을 보면 일반적으로 좋은편인 것을 알 수 있습니다.   
설치는 OpenMPI - (TF,Horovod 사전준비) - NCCL설치 - Horovod설치 순으로 진행하기 바랍니다.  

&nbsp;
## 2. OpenMPI설치
---
1. `https://www.open-mpi.org/software/ompi/v4.0/`   
2. Download and Extract 'openmpi-4.0.2.tar.gz'  # Use Latest
3. 
    ```bash
    $ cd ./openmpi-4.0.2.tar.gz   
    ```
4. ```bash
    $ ./configure --prefix=/home/$USER/.openmpi  
   ```
    It is necessary to add on the prefix the installation directory we want to use for OpenMPI.  
    The normal thing to do would be to select the next directory “/home/'user'/.openmpi”.  
5. Install (For parallel compile)  
    ```bash
    $ NPROCS='grep -c processor /proc/cpuinfo';  
    $ make -j $NPROCS all  
    $ make install  
    ```

6. Environment setting  
    ```bash
    $ echo 'export PATH=$PATH:/home/$USER/.openmpi/bin' >> /home/$USER/.bashrc  
    $ echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/$USER/.openmpi/lib/' >> /home/$USER/.bashrc  
    ```

&nbsp;
## 3. OpenMPI 설치완료 확인
---
```bash
$ mpirun
--------------------------------------------------------------------------
mpirun could not find anything to do.

It is possible that you forgot to specify how many processes to run
via the "-np" argument.
--------------------------------------------------------------------------
```

&nbsp;
## 4. Tensorflow - Horovod 연동전 사전작업
---
```bash
(optional) $ conda install gxx_linux-64 # conda 환경에서는 필수  
$ gcc -v # Check gcc version > 4.9  
$ pip install tensorflow # 1.13 버전으로 가정  
```

&nbsp;
## 5. NCCL 설치
---
Note: If you are using the network repository, the following command will upgrade CUDA to the latest version.  
```bash 
$ sudo apt install libnccl2 libnccl-dev  
```

OR, If you prefer to keep an older version of CUDA, specify a specific version, for example:  
```bash
$ sudo apt install libnccl2=2.5.6-1+cuda10.0 libnccl-dev=2.5.6-1+cuda10.0 # CUDA 10.0으로 설정  
$ echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/x86_64-linux-gnu' >> ~/.bashrc  
$ source ~/.bashrc  
```

&nbsp;
## 6. Horovod 설치
---
1. NCCL 설치 확인  
```bash
$ sudo dpkg-query -L libnccl-dev # 위치 확인  
/usr/lib/x86_64-linux-gnu/
/usr/include/
```
2. 설치  
```bash
$ HOROVOD_NCCL_HOME=/usr/lib/x86_64-linux-gnu HOROVOD_GPU_ALLREDUCE=NCCL HOROVOD_WITH_TENSORFLOW=1 HOROVOD_WITHOUT_PYTORCH=1 HOROVOD_WITHOUT_MXNET=1 pip install --no-cache-dir horovod  
```
OR (추천 방법, NCCL 위치 설정 제거)
```bash   
$ HOROVOD_NCCL_HOME=/usr/lib/x86_64-linux-gnu HOROVOD_WITH_TENSORFLOW=1 HOROVOD_WITHOUT_PYTORCH=1 HOROVOD_WITHOUT_MXNET=1 pip install --no-cache-dir horovod  
```

&nbsp;
## 7. Run Tensorflow with Horovod  
---
1. [Example code](https://github.com/horovod/horovod/blob/master/examples/tensorflow_mnist.py)
2. 실행
```bash
$ horovodrun -np 4 -H localhost:4 python tensorflow_mnist.py  
```

&nbsp;
## 8. Reference link
---
<http://solarisailab.com/archives/2627>  
<https://github.com/horovod/horovod>  
<https://samishappy.tistory.com/25>  
<http://lsi.ugr.es/jmantas/pdp/ayuda/datos/instalaciones/Install_OpenMPI_en.pdf>  
<https://lambdalabs.com/blog/horovod-keras-for-multi-gpu-training/>  
<https://github.com/horovod/horovod/blob/master/examples/tensorflow_mnist.py>  



