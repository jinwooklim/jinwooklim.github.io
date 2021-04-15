---
title:  "Camera Projection 1"
excerpt: ""
toc: true
toc_sticky: true
toc_label: "Table"
categories:
  - Development
tags:
  - Theory
comments : true
use_math: true

# layout: single
# classes: wide
---

# Camera Projection 1

조금 정리한 Camera Projection 내용입니다.

**Mathjax 버그로 인하여 가끔식 Notaion이 다를 수 있습니다... 따라서 참고용 figure를 추가했습니다.** 

&nbsp;

다음 레퍼런스에서 많은 도움을 받았습니다.

Reference :

1. [Penn state Computer Vision](http://www.cse.psu.edu/~rtc12/CSE486/)
2. [다크프로그래머님](https://darkpgmr.tistory.com/category/%EC%98%81%EC%83%81%EC%B2%98%EB%A6%AC)

&nbsp;

# World Coordintate System

먼저, World Coordinate System을 $(\textbf{U}, \textbf{V}, \textbf{W})$ 라고 합니다.

![001](/assets/images/posts/2021-04-15-Camera_Projection_1/001.PNG)

---

&nbsp;

# Camera Coordidate System

Camera Coordinate System을 $(\textbf{X}, \textbf{Y}, \textbf{Z})$ 라고 합니다.

1. $\textbf{Z}$는 optical axis 라고 하고, 한글로는 광축이라 표현합니다. 카메라의 전방을 표현합니다.
2. Image plane, 이미지 평면은 optical axis로 부터 $f$ 만큼 떨어진 곳에 위치합s니다.
3. $f$는 'focal length'라고 부릅니다. 한글로는 '초점 거리' 라고 표현합니다. 

![002](/assets/images/posts/2021-04-15-Camera_Projection_1/002.PNG)

---

&nbsp;

# World Point Projection

World coordinate의 점이 image plane에 투영(projection)되는 것을 표현하면 아래 그림과 같습니다.

$3\text{D} (\text{X},\text{Y},\text{Z})$ &rarr; $2\text{D} (x, y)$

![003](/assets/images/posts/2021-04-15-Camera_Projection_1/003.PNG)

---

&nbsp;

# Pixel Coordinates

그런데, 실제로 우리가 영상(image)를 얻기 위해서는 image plane위의 값을 digitize 하여 아래 그림과 같은 pixel coordinate $(u, v)$ 로 가져와야 합니다.

![004](/assets/images/posts/2021-04-15-Camera_Projection_1/004.PNG)

---

&nbsp;

# Forward Projection

지금까지 설명한 것을 모두 그리면 다음 그림과 같습니다.

이를 **"Forward Projection"** 이라 합니다.

![005](/assets/images/posts/2021-04-15-Camera_Projection_1/005.PNG)

---

&nbsp;

**"Forward Projection"**을 그림으로 정리하면 다음과 같습니다.

이 과정은 우리가 3D World points 들을 2D pixel coordinates로 표현할 때 사용됩니다.

그리고, 이 과정을 matrix equation으로 표현하는 것이 목적이 됩니다.

![006](/assets/images/posts/2021-04-15-Camera_Projection_1/006.PNG)

---

&nbsp;

# Backward Projection

당연히 **"Forward Projection"**의 역방향인 **"Backward Projection"**도 있습니다.

이 과정은 images (이미지들)로 3D scene structure를 복구하는 것이 목적입니다. (stereo 나 motion을 이용하여)

![007](/assets/images/posts/2021-04-15-Camera_Projection_1/007.PNG)

---

&nbsp;

# Camera Coordinates to Film Coordinates

설명의 용이함을 위해, 먼저 **Camera coordinates** 에서 **Film Coordinates**로 가는 것을 보겠습니다.

![008](/assets/images/posts/2021-04-15-Camera_Projection_1/008.PNG)

---

Camera Coordinates의 origin(원점)으로부터 $\text{Z}$축으로 $f$만큼 떨어져 있는 위치의 Image plane에 투영이 이루어진다고 합시다.

이때의, Perspective Projection (원근 투영)식은 $x=f{X \over Z}$, $y=f{Y \over Z}$입니다.

그리고, 다음 그림들을 통해 Scene Point to Image Point 를 삼각형의 닮음비를 이용해서 구할 수 있음을 보여주고 있습니다.

![009](/assets/images/posts/2021-04-15-Camera_Projection_1/009.PNG)

![010](/assets/images/posts/2021-04-15-Camera_Projection_1/010.PNG)

![011](/assets/images/posts/2021-04-15-Camera_Projection_1/011.PNG)

---

그런데, x, y 따로 계산하는 것 보다 계산의 효율성을 생각해서 matrix equation으로 이를 표현해 봅니다.

2D point $(x,y)$는 허구의 세번째 coordinate를 추가하여 3D point $(x',y',z')$로 표현할 수 있습니다. (Homogeneous coordinates)

만약, 3D point $(x',y',z')$가 주어지면, 우리는 아래 방정식을 이용해서 2D point $(x,y)$를 구할 수 있습니다.

$x={x' \over z'}$, $y={y' \over z'}$

**Note: (x,y) = (x,y,1) = (2x, 2y, 2) = (k x, ky, k) for any nonzero k (can be negative as well as positive)**

앞에서 이용한 방정식  $x=f{X \over Z}$, $y=f{Y \over Z}$ 을 이용하면 **Perspective Matrix Equation**을 구할 수 있습니다. (Remind : Camera Coordinates to Film Coordinates 간의 관계)

![012](/assets/images/posts/2021-04-15-Camera_Projection_1/012.PNG)

---

&nbsp;

# World Coordinates to Camera Coordinates

두번째로, World coordinates 에서 Camera coordinates로 이동하는 부분입니다.

이 부분은 world와 camera coordinate systems 간의 Rigid Transformation (rotation + translation) 입니다.

![013](/assets/images/posts/2021-04-15-Camera_Projection_1/013.PNG)

---

&nbsp;

# World to Camera Translation

**World to Camera translation** 을 그림으로 나타내면 다음과 같습니다.

World coordinates와 Camera coordinates 는 같은 점 $\text{P}_C$ $= \text{P}_W$ 을 각자 다른 형식으로 표현하고 있는 것이기에,

이를 맞춰주기 위해서 World coordinates 를 Camera coordinates로 Rigid Transformation 하는 것입니다. 

따라서 식으로 표현하자면 $\text{P}_C = \text{R}(\text{P}_C)$
로 표현할 수 있습니다. 
($\text{R}$ 은 축을 일치하기 위한 회전, $-\text{C}$ 는 origin 위치의 평행이동)

이를 Homogeneous coordinate를 이용해서, Matrix 형태로 표현하면 아래 식으로 표현할 수 있습니다.  

$\text{R} = \begin{bmatrix}
r_{11} & r_{12} & r_{13} & 0 \cr 
r_{21} & r_{22} & r_{23} & 0 \cr
r_{31} & r_{32} & r_{33} & 0 \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}$

$\text{P}_C - \text{C} = \begin{bmatrix}
1 & 0 & 0 & -c_x \cr
0 & 1 & 0 & -c_y \cr
0 & 0 & 1 & -c_z \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}$

$\text{P}_C = \text{R}(\text{P}_W - \text{C})$

$= \begin{bmatrix} \text{X} \cr \text{Y} \cr \text{Z} \cr 1 \end{bmatrix} = \begin{bmatrix}
r_{11} & r_{12} & r_{13} & 0 \cr
r_{21} & r_{22} & r_{23} & 0 \cr
r_{31} & r_{32} & r_{33} & 0 \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & -c_x \cr
0 & 1 & 0 & -c_y \cr
0 & 0 & 1 & -c_z \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}
\begin{bmatrix}
\text{U} \cr \text{V} \cr \text{W} \cr 1
\end{bmatrix}$

![014](/assets/images/posts/2021-04-15-Camera_Projection_1/014.PNG)

![015](/assets/images/posts/2021-04-15-Camera_Projection_1/015.PNG)

---

&nbsp;

# Stereo System Model

Stereo System을 Model을 가정한 예제를 통해 left camera와 right camera 간의 관계를 보겠습니다.

먼저, Left camera가 world origin (0, 0, 0)에 위치한다고 하고, ($(\text{U}, \text{V}, \text{W}) = (\text{X}, \text{Y}, \text{Z})$)

Right camera가 world coordinates 상에서 $\text{X}$축으로 $\text{T}_\text{x}$만큼 평행이동하여 align 되어 있다고 합시다.

![016](/assets/images/posts/2021-04-15-Camera_Projection_1/016.PNG)

---

&nbsp;

# Rigid Transformation : Left Camera

**Left camera**에 대한 **Rigid Transformation**은 다음과 같습니다.

World coordinates와 Left camera coordinates 가 일치하므로 translation 을 하지 않습니다(C=0).

$\text{P}_\text{W} - \text{C}=\begin{bmatrix}
1 & 0 & 0 & 0 \cr
0 & 1 & 0 & 0 \cr
0 & 0 & 1 & 0 \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}
\begin{bmatrix}
\text{U} \cr \text{V} \cr \text{W} \cr 1 \end{bmatrix}$

그리고 Rotation의 경우에도 축들이 일치하므로 rotation을 하지 않습니다.

$\text{R} = \begin{bmatrix}
1 & 0 & 0 & 0 \cr
0 & 1 & 0 & 0 \cr
0 & 0 & 1 & 0 \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}$

이를 정리하면, $(U, V, W, 1)$부터 $(X, Y, Z, 1)$까지의 변환이 $\begin{bmatrix}
1 & 0 & 0 & 0 \cr
0 & 1 & 0 & 0 \cr
0 & 0 & 1 & 0 \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}$ 입니다.

따라서, Left camera 에 투영된 Image plane의 점은 다음과 같이 표현할 수 있습니다.

$\begin{bmatrix} x_l \cr y_l \cr  1 \end{bmatrix}
\sim\begin{bmatrix}
f & 0 & 0 & 0 \cr
0 & f & 0 & 0 \cr
0 & 0 & 1 & 0 \cr
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & 0 \cr
0 & 1 & 0 & 0 \cr
0 & 0 & 1 & 0 \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}
\begin{bmatrix}
X \cr Y \cr Z \cr 1
\end{bmatrix}$

$x_l = f{X \over Z}, y_l=f{Y \over Z}$ 입니다.

$\sim$ 는 up to scale 을 표현하기 위해 사용되었습니다.

![017](/assets/images/posts/2021-04-15-Camera_Projection_1/017.PNG)

![018](/assets/images/posts/2021-04-15-Camera_Projection_1/018.PNG)

---

&nbsp;

# Rigid Transformation : Right Camera

Right camera에 대한 Rigid Transformation은 다음과 같습니다.

World coordinates으로 부터 $X$축으로 $T_\text{x}$만큼 이동해 있으므로, 이를 반영해줍니다.

$\text{P}_\text{W} - \text{C}=
\begin{bmatrix}
1 & 0 & 0 & -\text{T}_x \cr
0 & 1 & 0 & 0 \cr
0 & 0 & 1 & 0 \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}
\begin{bmatrix}
\text{U} \cr \text{V} \cr \text{W} \cr 1 \end{bmatrix}$

그리고 Rotation의 경우에는 축들이 일치하므로 rotation을 하지 않습니다.

$\text{R} = \begin{bmatrix}
1 & 0 & 0 & 0 \cr
0 & 1 & 0 & 0 \cr
0 & 0 & 1 & 0 \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}$

이를 정리하면, $(U, V, W, 1)$부터 $(X, Y, Z, 1)$까지의 변환이 $\begin{bmatrix}
1 & 0 & 0 & -\text{T}_\text{x} \cr
0 & 1 & 0 & 0 \cr
0 & 0 & 1 & 0 \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}$ 입니다.

따라서, Right camera 에 투영된 Image plane의 점은 다음과 같이 표현할 수 있습니다.

$\begin{bmatrix} x_r \cr y_r \cr  1 \end{bmatrix}
\sim\begin{bmatrix}
f & 0 & 0 & 0 \cr
0 & f & 0 & 0 \cr
0 & 0 & 1 & 0 \cr
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & -\text{T}_\text{x} \cr
0 & 1 & 0 & 0 \cr
0 & 0 & 1 & 0 \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}
\begin{bmatrix}
X \cr Y \cr Z \cr 1
\end{bmatrix}$

$x_r = f{X--\text{T}_\text{x} \over Z}, y_r=f{Y \over Z}$ 입니다.

$\sim$ 는 up to scale 을 표현하기 위해 사용되었습니다.

![019](/assets/images/posts/2021-04-15-Camera_Projection_1/019.PNG)

![020](/assets/images/posts/2021-04-15-Camera_Projection_1/020.PNG)

---

&nbsp;

# Stereo Camera Projection Equations

앞에서 정리했던 Stereo (Left camera, Right camera) projection equations를 다시 정리하면 다음과 같습니다.

![021](/assets/images/posts/2021-04-15-Camera_Projection_1/021.PNG)

---

&nbsp;

# Bob’s sure-fire way(s) to figure out the rotation

Rotations 에 대해서 좀 더 알아보겠습니다.

$\begin{bmatrix} \text{X} \cr \text{Y} \cr \text{Z} \cr 1 \end{bmatrix}
=\begin{bmatrix}
r_{11} & r_{12} & r_{13} & 0 \cr
r_{21} & r_{22} & r_{23} & 0 \cr
r_{31} & r_{32} & r_{33} & 0 \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & -c_x \cr
0 & 1 & 0 & -c_y \cr
0 & 0 & 1 & -c_z \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}
\begin{bmatrix}
\text{U} \cr \text{V} \cr \text{W} \cr 1
\end{bmatrix}$ 에서 rotations의 영향을 알아보는 동안에는 우항의 중앙 행렬은 무시하면 됩니다.

$\text{P}_C = \text{R}\text{P}_W$는 world coordinates system 에서의 vectors 들이 camera coordinates system에 변환되는 식입니다.

![022](/assets/images/posts/2021-04-15-Camera_Projection_1/022.PNG)

---

$\text{P}_C = \text{R}\text{P}_W$를 조금 더 자세히 보겠습니다.

world $\text{X}$ axis (1, 0, 0) 가 camera axis (a,b,c)로 이동된다면 rotations matrix에서 어떤 부분이 영향을 주는 것일까요 ?

$r_{11}, r_{21}, r_{31}$ 만이 영향을 주는 것을 볼 수 있습니다.

즉, world 에서 camera 로는
$\begin{bmatrix}
r_{11} & r_{12} & r_{13} & 0 \cr
r_{21} & r_{22} & r_{23} & 0 \cr
r_{31} & r_{32} & r_{33} & 0 \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}$ 에서

$(r_{11}, r_{21}, r_{31})$ 가 X axis 에 영향을 주고

$(r_{12}, r_{22}, r_{32})$ 가 Y axis 에 영향을 주고

$(r_{13}, r_{23}, r_{33})$ 가 Z axis 에 영향을 줍니다.

![023](/assets/images/posts/2021-04-15-Camera_Projection_1/023.PNG)

![024](/assets/images/posts/2021-04-15-Camera_Projection_1/024.PNG)

---

반대로, camera X, Y, Z axis가 world 에서 어디에 해당하는지 알수 있습니다.

$\text{P}_C = \text{R}\text{P}_W$ 
&rarr; 
$\text{R}^{-1}\text{P}_C = \text{P}_W$ 
&rarr; 
$\text{R}^{\text{T}}\text{P}_C = \text{P}_W$

즉, camera 에서 world 로는
$\begin{bmatrix}
r_{11} & r_{21} & r_{31} & 0 \cr
r_{12} & r_{22} & r_{32} & 0 \cr
r_{13} & r_{23} & r_{33} & 0 \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}$ 에서

$(r_{11}, r_{21}, r_{31})$ 가 X axis 에 영향을 주고

$(r_{12}, r_{22}, r_{32})$ 가 Y axis 에 영향을 주고

$(r_{13}, r_{23}, r_{33})$ 가 Z axis 에 영향을 줍니다.

![025](/assets/images/posts/2021-04-15-Camera_Projection_1/025.PNG)

![026](/assets/images/posts/2021-04-15-Camera_Projection_1/026.PNG)

![027](/assets/images/posts/2021-04-15-Camera_Projection_1/027.PNG)

---

이해한 것을 예제에 적용하면,

World &rarr; Train 은 다음과 같습니다.

$P_{train} = R_{train}P_{world}$

$\begin{bmatrix} x \text{ in train camera coordinates} \cr
y \text{ in train camera coordinates} \cr 
z \text{ in train camera coordinates}
\end{bmatrix}
=\begin{bmatrix}
0 & 0 & 1 \cr
0 & -1 & 0 \cr
1 & 0 & 0 \cr
\end{bmatrix}
\begin{bmatrix}
X \text{ in world coordinates} \cr
Y \text{ in world coordinates} \cr 
Z \text{ in world coordinates}
\end{bmatrix}$

World &rarr; Fly 은 다음과 같습니다.

$P_{fly} = R_{fly}P_{world}$

$\begin{bmatrix} x \text{ in fly camera coordinates} \cr
y \text{ in fly camera coordinates} \cr 
z \text{ in fly camera coordinates}
\end{bmatrix}
=\begin{bmatrix}
0 & -1 & 0 \cr
0 & 0 & 1 \cr
-1 & 0 & 0 \cr
\end{bmatrix}
\begin{bmatrix}
X \text{ in world coordinates} \cr
Y \text{ in world coordinates} \cr 
Z \text{ in world coordinates}
\end{bmatrix}$

![028](/assets/images/posts/2021-04-15-Camera_Projection_1/028.PNG)

---

&nbsp;

# Note: External Parameters also often written as R,T

다시 한번, World coordinates to Camera coordinates 식을 한번 보겠습니다.

$\begin{bmatrix} \text{X} \cr \text{Y} \cr \text{Z} \cr 1 \end{bmatrix}
=\begin{bmatrix}
r_{11} & r_{12} & r_{13} & 0 \cr
r_{21} & r_{22} & r_{23} & 0 \cr
r_{31} & r_{32} & r_{33} & 0 \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & -c_x \cr
0 & 1 & 0 & -c_y \cr
0 & 0 & 1 & -c_z \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}
\begin{bmatrix}
\text{U} \cr \text{V} \cr \text{W} \cr 1
\end{bmatrix}$

지금까지 설명한 rotation matrix, translation matrix는 하나로 합쳐서 설명할 수 있습니다.

rotation matrix : $\begin{bmatrix}
r_{11} & r_{12} & r_{13} & 0 \cr
r_{21} & r_{22} & r_{23} & 0 \cr
r_{31} & r_{32} & r_{33} & 0 \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}$

translation matrix : $\begin{bmatrix}
1 & 0 & 0 & -c_x \cr
0 & 1 & 0 & -c_y \cr
0 & 0 & 1 & -c_z \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}$

$\text{R}(\text{P}_W-\text{C})
=\text{R}\text{P}_W-\text{R}\text{C}
=\text{R}\text{P}_W+\text{T}$

$=\begin{bmatrix}
r_{11} & r_{12} & r_{13} & t_x \cr
r_{21} & r_{22} & r_{23} & t_y \cr
r_{31} & r_{32} & r_{33} & t_z \cr
0 & 0 & 0 & 1 \cr
\end{bmatrix}$

![029](/assets/images/posts/2021-04-15-Camera_Projection_1/029.PNG)



