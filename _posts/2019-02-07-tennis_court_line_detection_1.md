---
layout: post
toc: true
title: "OpenCV 테니스 코트 인식 1 (라인 검출)"
categories: vision
tags: [opencv, python, vision]
---


Python 환경에서 OpenCV를 이용해 이미지에서 테니스 코트 라인 추출을 하는 과정을 설명합니다.

사용할 라이브러리와 원본 이미지는 아래와 같습니다. 이번 포스팅에서는 아래 원본 이미지 처럼 테니스 코트의 중앙에서 전체 코트가 포함되도록 수평으로 촬영된 이미지를 기준으로 코드를 작성하였습니다. 만일 이러한 구속조건이 없는 경우 좀 더 복잡한 알고리즘을 구상해야 합니다.

``` python
import math
import cv2
import numpy as np

img = cv2.imread("court.jpg") # 원본 이미지 불러오기
```
![original](https://user-images.githubusercontent.com/7419790/95008179-80a03500-0652-11eb-9e57-630777873222.jpg)
![court](https://user-images.githubusercontent.com/7419790/95008759-bf84b980-0657-11eb-9606-f97636f6c73a.jpg)

이미지 처리 및 테니스 코트 라인 검출은 아래과 같은 순서로 이루어집니다. 최종적으로 테니스 코트 상의 4개의 포인트 좌표가 검출 되며 이 좌표는 이후의 **Camera Pose** 추정을 위한 기준점으로 활용됩니다.


## 1. 이미지 전처리

주변 환경과 카메라의 특성에 의한 영향을 줄이기 위해 전처리 과정을 적용합니다. 여기서는 어떤 정답이 있는게 아니라 경험적으로 판단한 적정 알고리즘과 파라미터를 찾아야 합니다.

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
이미지에 포함된 직선들을 검출합니다. 먼저 Morphology 연산과 Canny Edge 알고리즘을 적용하여 윤곽이 강조되도록 한 뒤 Hough 변환을 이용한 직선 검출 알고리즘을 사용합니다. 검출된 직선들은 각각 `rho`와 `theta` 두개의 파라미터로 표현됩니다.
``` python
def lineDetection(img):

    # Morphology 적용으로 흰색 선 강조
    kernel = np.ones((5,5), np.uint8)
    img = cv2.morphologyEx(img, cv2.MORPH_CLOSE, kernel)

    # Edge 검출로 윤곽만 추출
    img = cv2.Canny(img, 50, 200)

    # 직선 검출
    lines = cv2.HoughLines(img, 1.0, 3.141592/180/5, 300)

    return lines
```
![06_line](https://user-images.githubusercontent.com/7419790/95008221-dbd22780-0652-11eb-9cf5-c7a43bd84c44.jpg)


## 3. 라인 클러스터링
Hough 변환에 의한 직선 검출은 위 이미지 처럼 하나의 코트 라인에 직선 여러개가 검출됩니다. 따라서 유사한 직선 끼리 클러스터링한 뒤 각 클러스터를 대표하는 값으로 평균 `rho`와 `theta`를 저장합니다.

이번 예제에서는 단순히 **Euclidean Distance**를 이용하여 클러스터링을 수행하였으며 좀 더 정확한 클러스터링을 위해 다른 알고리즘을 활용할 수도 있습니다.

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



## 5. 라인의 교점 연산



