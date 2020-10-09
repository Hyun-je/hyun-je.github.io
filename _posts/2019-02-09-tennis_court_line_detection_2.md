---
layout: post
toc: true
title: "OpenCV 테니스 코트 인식 2 (카메라 캘리브레이션)"
categories: vision
tags: [opencv, python, vision]
---

[첫번째 포스팅](https://hyun-je.github.io/vision/2019/02/07/tennis_court_line_detection_1.html)에서는 간단한 영상처리와 직선 검출을 통해 테니스 코트 위 4개의 기준 포인트를 구하였다. 이 기준 포인트를 이용하여 카메라 캘리브레이션(Camera Calibration)을 수행하는 코드를 작성할 것이다.

카메라 캘리브레이션은 2D 상의 이미지 좌표 $$q$$와 실제 3D 공간상의 좌표 $$Q$$를 수학적으로 상호 변환가능하게 해주는 카메라 모델 식의 파라미터를 찾는 것을 의미하며 그 식은 다음과 같은 형태를 갖는다.

$$
q=\begin{bmatrix}
u \\ v \\ 1\\
\end{bmatrix}, \ \ \ 
Q=\begin{bmatrix}
x \\ y \\ z \\ 1
\end{bmatrix}
$$

$$
c \cdot q=M \begin{bmatrix}R|T\end{bmatrix} X
$$

이 때 $$M$$은 `Intrinsic Matrix`를 의미하며 $$\begin{bmatrix}R|T\end{bmatrix}$$는 `Extrinsic Matrix`를 의미하는데 각 Matrix를 구하는 과정은 아래에 자세하게 정리하였다.

$$
M=\begin{bmatrix}
f_x & 0 & c_x \\
0 & f_y & c_y \\
0 & 0 & 1 \\
\end{bmatrix}, \ \ \ 
R=\begin{bmatrix}
r_{11} & r_{12} & r_{13} \\
r_{21} & r_{22} & r_{23} \\
r_{31} & r_{32} & r_{33} \\
\end{bmatrix}, \ \ \ 
T=\begin{bmatrix}
t_1 \\
t_2 \\
t_3 \\
\end{bmatrix}
$$


## 1. 좌표계 설정
가장 먼저 필요한 작업은 3D 월드 좌표계 및 2D 이미지 좌표계를 설정하는 것이다. 좌표계 설정에 따라서 이후의 연산 과정이나 최종 연산 결과를 렌더링하는 등의 작업 편의성이 크게 달라진다. 따라서 좌표계를 설정하기전 내가 사용하는 라이브러리 또는 프레임워크에서 어떤 좌표계를 사용하는지 충분히 숙지한 뒤 적절한 좌표계를 설정하는 것이 좋다.

여기서는 테니스 코트 중심을 원점으로 하는 미터 단위의 월드 좌표계를 설정하였고, OpenCV에서 사용하는 이미지 우측 상단을 원점으로 하는 픽셀 단위의 이미지 좌표계를 설정하였다.

![coordinate system](https://user-images.githubusercontent.com/7419790/95461047-b7f34680-09b0-11eb-8f1d-f41fe36ec093.jpg)


## 2. Intrinsic Matrix 계산
`Intrinsic Matrix`는 카메라의 이미지 센서와 렌즈의 하드웨어적인 스펙에 의해 정해지는 파라미터로 구성된다.

예제 사진은 iPhone X의 후면 카메라를 사용하여 촬영하였는데 애플은 모든 아이폰 카메라의 파라미터를 [개발자 문서](https://developer.apple.com/library/archive/documentation/DeviceInformation/Reference/iOSDeviceCompatibility/Cameras/Cameras.html)를 통해 제공하므로 해당 값을 그대로 이용할 수 있다. 만일 카메라의 파라미터를 구할 수 없는 경우 OpenCV에서 제공하는 카메라 캘리브레이션 기능을 활용하여 추정할 수 있다.

``` python
# 카메라 파라미터 정의
imageWidth = 1280
imageHeight = 720
cameraFOV = 58.903 # iPhone 6 Plus Rear Camera (x-axis)

# x, y축 FOV 계산 (단위: radians)
fov_x = math.radians(cameraFOV)
fov_y = 2 * math.atan(imageHeight/imageWidth * math.tan(fov_x/2))

# Intrinsic Matrix 구함
fx = (imageWidth/2) / math.tan(fov_x/2)
fy = (imageHeight/2) / math.tan(fov_y/2)
cx = imageWidth/2
cy = imageHeight/2

cameraMatrix = np.array([
    [fx,  0, cx],
    [ 0, fy, cy],
    [ 0,  0,  1]
])

# cameraMatrix = [
#   [1133.4      0.0   640.0]
#   [   0.0   1133.4   360.0]
#   [   0.0      0.0     1.0]
# ]
```

## 3. Extrinsic Matrix 추정
`Extrinsic Matrix`는 3D 공간상의 Camera Pose를 의미하는데 Rotational Vector와 Translational Vector가 합쳐진 형태를 가진다. 대부분의 문제에서 `Extrinsic Matrix`는 미지의 값이기 때문에 이를 해석적 방법 또는 반복적 방법으로 해를 구해야 한다. 이것을 비전 분야에서는 Perspective-n-Point Problem으로 정의하여 이미 많은 연구가 이뤄진 매우 고전적인 문제이다.

OpenCV에서는 `solvePnP`, `solvePnpRansac`등 이미 검증된 Solver 알고리즘을 제공하므로 직접 구현할 필요없이 적절히 사용해주기만 하면 된다. 이번에는 그 중 가장 기본이 되는 `solvePnP`함수를 사용할 것이다. 이 함수는 3개의 좌표 세트만 주어지면 Extrinsic Matrix 추정할 수 있고 초기값을 정답과 대략적으로 유사하게 제공해주면 훨씬 더 좋은 결과물을 얻을 수 있다.

`solvePnP`의 입력으로 필요한 2D 이미지 좌표(`imagePoints`)는 [첫번째 포스팅](https://hyun-je.github.io/vision/2019/02/07/tennis_court_line_detection_1.html)에서 구한 4개의 기준 포인트를 사용할 것이며 각 2D 이미지 포인트에 매칭되는 3D 월드 좌표(`objectPoints`)는 테니스 코트 규격을 참조하여 입력하였다.

그리고 이미지 센서의 Skew나 렌즈에 의한 Distortion은 없다고 가정하였는데 초점거리가 짧아 렌즈에 의한 왜곡이 큰 카메라로 촬영한 영상을 사용하는 경우 Distortion에 대한 고려도 반드시 필요하다.

``` python
objectPoints = np.array([
    [11.89, -4.115, 0.0],
    [11.89,  4.115, 0.0],
    [ 6.4,  -4.115, 0.0],
    [ 6.4,   4.115, 0.0]
])

imagePoints = np.array([
    [-318.74, 502.66],
    [1663.06, 524.96],
    [ 216.80, 308.27],
    [1093.20, 322.04]   
])

distCoeffs = np.zeros((4, 1), dtype='float32') # No Distortion

# Estimate Extrinsic Matrix
retval, rvec, tvec = cv2.solvePnP(objectPoints, imagePoints, cameraMatrix, distCoeffs)

# Rodrigues → Rotational Matrix
rmat = np.zeros(shape=(3, 3))
cv2.Rodrigues(rvec, rmat)

# rmat = [
#   [-0.00205 -0.99991  0.01311]
#   [-0.18644 -0.01250 -0.98238]
#   [ 0.98246 -0.00446 -0.18640]
# ]
```


## 4. 테니스 코트 렌더링
이제 `Intrinsic Matrix`, `Extrinsic Matrix`를 모두 알고 있으므로 주어진 $$x, y, z$$ 값을 갖는 월드 좌표에 대해 $$u, v$$ 값을 갖는 이미지 좌표를 구할 수 있다. 즉, 기준 좌표로 사용한 4개의 포인트 이외에도 테니스 코트 위의 특정 위치에 점을 찍는 등의 렌더링이 가능하고 더 나아가서 3D 오브젝트를 렌더링하는 AR 구현도 가능한 것이다.

아래 코드에서는 간단하게 테니스 코트의 외곽 라인을 렌더링해보았다. 단지 테니스 코트 상의 4개의 기준점만 알아냈을 뿐인데 이미지 상의 코트 전체의 라인의 위치를 추정할 수 있다는 점이 매우 흥미로웠다.

``` python
# 코트 외곽 라인의 월드 좌표 리스트
outerLineWorld = [
    [ 11.89,  5.485, 0.0],
    [ 11.89, -5.485, 0.0],
    [-11.89, -5.485, 0.0],
    [-11.89,  5.485, 0.0],
    [ 11.89,  5.485, 0.0]
]

# 코트 외곽 라인의 이미지 좌표 연산
outerLineImage = []
for point in outerLineWorld:
    x, y, z = point
    q = np.matmul(cameraMatrix, np.matmul(rmat, [[x], [y], [z]]) + tvec)
    c = q[2]
    q = q / c
    outerLineImage.append(q)

# 코트 외곽 라인 렌더링
for i in range(len(outerLineImage)-1):
    x1, y1, _ = outerLineImage[i]
    x2, y2, _ = outerLineImage[i+1]
    cv2.line(srcImg,(x1, y1), (x2, y2), (255,0,255), 2)
```
![result](https://user-images.githubusercontent.com/7419790/95569980-41625180-0a61-11eb-872b-4ffcd197c1fc.jpg)


한편 앞의 경우와 반대 연산인 이미지 좌표에서 월드 좌표를 구하는 것도 가능하다. 단, 이 경우에는 카메라 모델에서 $$c$$가 미지의 값이므로 점이 아닌 선의 형태로 구해진다. 따라서 월드 좌표의 $$x, y, z$$ 값 중 하나는 알고 있어야 유일한 하나의 점을 구할 수 있다.

예를 들어 테니스 코트 바닥면에 놓인 테니스 공을 인식하여 이미지 좌표를 구했다면 바닥면의 $$z$$ 값이 0인 것을 이용하여 나머지 $$x, y$$ 좌표를 얻을 수 있다. 이런 방법을 응용한다면 비전 인식을 통해 실제 공간상 물체의 3D 좌표를 알 수 있는 것이다.


## 결론
카메라 캘리브레이션은 비전, AR, 컴퓨터 그래픽스 등 다양한 분야에서 사용되는 개념으로 카메라 모델의 수학적인 배경 지식이 필요하므로 완전히 이해하는데는 시간이 좀 필요하였다. 그러나 비전 기술에 대한 사고의 폭을 이미지 평면에만 머무르지 않고 현실 공간으로 확장시켜 주는 매우 흥미로운 기술이라는 생각이 들었다.

이번 프로젝트에서는 최종적으로 테니스 코트 라인을 렌더링하는 간단한 결과물만 얻었지만 좀 더 발전시켜서 테니스 선수나 공을 추적하여 실제 위치나 속도를 추적하는 등의 복잡한 문제에도 도전해 볼 계획이다.


## 참고자료
- [OpenCV Documentation - Camera Calibration and 3D Reconstruction](https://docs.opencv.org/4.0.0/d9/d0c/group__calib3d.html)