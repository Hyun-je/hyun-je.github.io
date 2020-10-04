---
layout: post
toc: true
title: "OpenCV 테니스 코트 인식 1 (라인 검출)"
categories: vision
tags: [opencv, python, vision]
---


Python 환경에서 OpenCV를 이용해 이미지에서 테니스 코트 라인 추출을 하는 과정을 정리하고자 한다.

사용할 라이브러리와 원본 이미지는 아래와 같다. 이번 포스팅에서는 아래 원본 이미지 처럼 테니스 코트의 중앙에서 전체 코트가 포함되도록 수평으로 촬영된 이미지를 기준으로 코드를 작성하였다. 만일 이러한 구속조건이 없는 경우 좀 더 복잡한 알고리즘을 구상해야 한다.

``` python
import math
import cv2
import numpy as np

img = cv2.imread("court.jpg") # 원본 이미지 불러오기
```
![original](https://user-images.githubusercontent.com/7419790/95008179-80a03500-0652-11eb-9e57-630777873222.jpg)

![court](https://user-images.githubusercontent.com/7419790/95008759-bf84b980-0657-11eb-9606-f97636f6c73a.jpg)

아래에서 설명할 이미지 처리 및 테니스 코트 라인 검출 과정을 통해 최종적으로 **X**로 표시된 4개 포인트(서비스 라인, 베이스 라인, 단식 사이드 라인의 교점)의 이미지 평면상의 좌표가 검출 된다.

위 4개의 포인트를 검출하는 이유는 코트 한쪽 사이드의 사람 키 높이에서에서 촬영을 한다고 가정했을 때 이미지내에서 가장 정확하게 검출 가능한 라인들의 교점이기 때문이다.


## 1. 이미지 전처리
주변 환경과 카메라의 특성에 의한 영향을 줄이기 위해 전처리 과정을 적용한다. 여기서는 어떤 정답이 있는게 아니라 경험적으로 판단한 적정 알고리즘과 파라미터를 찾아야 한다. 가장 간단해 보이면서도 인식률과 환경 변화에 대한 강건성에 큰 영향을 주는 부분이다.

- Color Space 변경 (RGB → 그레이스케일) : 불필요한 컬러 정보 제거
- Histogram 이퀄라이징 : 이미지의 밝기, 대비 등의 영향성을 낮춤
- Gaussian Blur : 노이즈의 영향성 낮춤

``` python
def preprocessImage(img):

    # 이미지 컬러스페이스 변환
    img = cv2.cvtColor(img, cv2.COLOR_RGB2GRAY)

    # 히스토그램 이퀄라이징
    clahe = cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8,8))
    img = clahe.apply(img)

    # 가우시안 블러 적용
    img = cv2.GaussianBlur(img, (5,5), 0)

    return img
```
![03_blur](https://user-images.githubusercontent.com/7419790/95008200-b0e7d380-0652-11eb-8444-66514f1b459a.jpg)



## 2. 직선 검출
이미지에 포함된 직선들을 Hough 변환 알고리즘을 사용하여 검출한다. 검출된 직선들은 각각 `rho`와 `theta` 두개의 파라미터로 표현된다.

- Morphology 연산 : 흰색 선을 강조
- Canny Edge 연산 : 윤곽 성분만 Binary 이미지로 검출
- Hough 변환 : Binary 이미지에 포함된 직선들을 검출

``` python
def lineDetection(img):

    # Morphology 적용
    kernel = np.ones((5,5), np.uint8)
    img = cv2.morphologyEx(img, cv2.MORPH_CLOSE, kernel)

    # Edge 검출
    img = cv2.Canny(img, 50, 200)

    # 직선 검출
    lines = cv2.HoughLines(img, 1.0, 3.141592/180/5, 300)

    return lines
```
![06_line](https://user-images.githubusercontent.com/7419790/95008221-dbd22780-0652-11eb-9cf5-c7a43bd84c44.jpg)


## 3. 라인 클러스터링
Hough 변환에 의한 직선 검출은 위 이미지 처럼 하나의 코트 라인에 직선 여러개가 검출된다. 따라서 유사한 직선 끼리 클러스터링한 뒤 각 클러스터를 대표하는 값으로 `rho`와 `theta`의 평균값을 저장한다.

이번 예제에서는 단순히 **Euclidean Distance**를 이용하여 클러스터링을 수행하였으며 좀 더 정확한 클러스터링을 위해 다른 알고리즘을 활용할 수도 있다.

``` python
def lineClustering(lines, threshold = 20.0):

    line_clusters = []

    for line in lines:
        
        rho0, theta0 = line[0]
        is_added_cluster = False

        for (index, line_cluster) in enumerate(line_clusters):

            rho1, theta1, cluster_size = line_cluster

            x1 = rho0 * math.cos(theta0)
            y1 = rho0 * math.sin(theta0)
            x2 = rho1 * math.cos(theta1)
            y2 = rho1 * math.sin(theta1)
            
            distance = math.sqrt(math.pow(x1 - x2, 2) + math.pow(y1 - y2, 2))

            if distance <= threshold:

                rho_new =   (rho1   * cluster_size + rho0)      / (cluster_size + 1)
                theta_new = (theta1 * cluster_size + theta0)    / (cluster_size + 1)

                line_clusters[index] = (rho_new, theta_new, cluster_size + 1)

                is_added_cluster = True
                break

        if is_added_cluster == False:
            line_clusters.append((rho0, theta0, 1))

    return line_clusters
```
![07_cluster](https://user-images.githubusercontent.com/7419790/95008232-fc9a7d00-0652-11eb-83cc-ce87021a9435.jpg)


## 4. 테니스 코트 라인 분류
클러스터링 된 직선 중 서비스 라인, 베이스 라인, 단식 사이드 라인에 해당하는 4개의 직선을 분류한다. 아래 코드의 알고리즘은 포스팅 서론에서 제시된 카메라 포즈에서만 유효하므로 좀 더 다양한 화각의 촬영조건에 대응하기 위해서는 고도의 알고리즘이 필요하다.

- 수평선 중 이미지 아래 쪽 2개의 라인 선택 → **서비스 라인, 베이스 라인**
- 좌우 대칭이 되는 라인 중 기울기가 가장 큰 2개의 라인 선택 → **단식 사이드 라인**

``` python
def horizontalLineDetection(lines):

    horizontalLines = list(filter(lambda line: abs(line[1] - math.pi/2.0) * 180.0/math.pi < 5.0, lines))
    horizontalLines.sort(reverse=True)

    baseLine = horizontalLines[0]
    serviceLine = horizontalLines[1]

    return (baseLine, serviceLine)


def sideLineDetection(lines):

    sideLines = []

    # 기울기 크기 순으로 정렬
    lines.sort(key=(lambda x: abs(x[1] - np.pi/2)), reverse=True)
    lines = list(filter(lambda line: abs(line[1] - math.pi/2.0) * 180.0/math.pi > 5.0, lines))

    # 대칭인 두개의 라인 찾기 (최대 기울기 1쌍만 선택)
    for i in range(len(lines) - 1):
        angle0 = lines[i][1]*180.0/math.pi - 90.0
        angle1 = lines[i+1][1]*180.0/math.pi - 90.0

        if abs(angle0 + angle1) < 2.0:
            if angle0 < 0:
                return (lines[0], lines[1])
            else:
                return (lines[1], lines[0])
```
![line_detected](https://user-images.githubusercontent.com/7419790/95012530-46478f80-0674-11eb-8bc8-c179d22c9fe1.jpg)


## 5. 라인의 교점 연산
최종적으로 분류된 4개의 테니스 코트 라인이 이루는 4개의 교점 좌표를 구한다. 이미 각 직선의 파라미터를 알고 있으므로 간단한 연립방정식 풀이만으로 교점의 x, y 좌표를 구할 수 있다.

아래 스크린샷에서 처럼 경우에 따라서는 교점이 이미지 경계를 벗어나고 심지어는 음수의 좌표가 구해질 수도 있는데 다음 포스팅에서 이어질 연산을 수행하는데 수학적으로 아무런 문제는 없다.

``` python
def intersectionPoint(line1, line2):

    rho1, theta1 = line1
    rho2, theta2 = line2

    m11 = math.cos(theta1)
    m12 = math.sin(theta1)
    m21 = math.cos(theta2)
    m22 = math.sin(theta2)

    x = rho1
    y = rho2
    
    d = m11*m22 - m21*m12

    x0 = ( m22 * x - m12 * y) / d
    y0 = (-m21 * x + m11 * y) / d

    return (x0, y0)
```
![points](https://user-images.githubusercontent.com/7419790/95012767-303ace80-0676-11eb-99b2-646c7076b953.jpg)


## 결론
이상으로 테니스 코트 인식을 OpenCV에서 제공하는 고전적인 Vision 기법으로 수행해 보았다. 촬영 위치에 대한 구속 조건이 있었으므로 꽤 단순한 과정만으로 처리가 가능하였지만 이 알고리즘을 베이스로 하여 더 고도화된 알고리즘 개발에도 도전해 보면 좋을 것이다.

한편, 전체 코트 라인 또는 Bounding Box를 검출하지 않고 단지 코트 상의 4개의 좌표만을 검출하였을때 이것을 어떻게 활용가능할지는 다음 포스팅을 보면 이해할 수 있을 것이다.


## 참고자료
- [OpenCV Documentation](https://docs.opencv.org/4.0.0/)
