---
layout: post
toc: true
title: "Swift 및 iOS Framework 기반 Face Alignment 구현"
categories: vision
tags: [swift, vision, openface]
---

Face Alignment는 얼굴 관련 알고리즘 및 머신러닝에서 필요로 하는 전처리 방법으로 원본 이미지에 포함된 얼굴의 각도, 크기, 비율 등에 관계 없이 표준화된 크기 및 눈, 코, 입 위치를 갖는 얼굴 이미지가 되도록 이미지를 Affine Transfrom 및 Crop 하는 것을 말한다.

Face Alignment는 매우 널리 사용되는 전처리 방법이므로 Python과 OpenCV를 이용한 코드는 GitHub 등에서 쉽게 찾을 수 있다. 그러나 iOS 환경에서 C++로 작성된 OpenCV를 구동하기 위해서는 다소 복잡한 Bridging 과정이 필요하며 또한 CPU만 이용하여 연산되므로 성능면에서도 불리하다는 문제가 있다.

따라서 얼굴 인식 관련 iOS 앱 개발 시 유용하게 사용할 수 있는 Swift 및 Apple 디바이스에 최적화된 Face Alignment 코드를 만들어보고자 한다.


### 1. Face Landmark 추출
Apple에서 제공하는 Vision 프레임워크의 Face Landmark 추출 기능을 활용한다. `VNDetectFaceLandmarksRequest`를 생성 후 실행하면 이미지에 포함된 얼굴 개수 만큼의 `VNFaceObservation`의 배열이 만들어진다.

``` swift
let request = VNDetectFaceLandmarksRequest()
try? VNSequenceRequestHandler().perform([request], on: originalImage)

guard let landmarksResults = faceLandmarks.results as? [VNFaceObservation] else { return nil }
```

각 `VNFaceObservation`의 `landmarks` 프로퍼티는 `VNFaceLankmarks2D` 타입을 가지며 아래와 같은 프로퍼티를 통해 전체 포인트 또는 특정 얼굴 요소의 포인트를 얻을 수 있다.

- 모든 포인트 : `allPoints`
- 얼굴 중심축 : `medianLine`
- 얼굴 윤곽 : `faceContour` 
- 눈 : `leftEye` `rightEye`
- 눈동자 : `leftPupil` `rightPupul`
- 눈썹 : `leftEyebrow` `rightEyebrow`
- 코 : `nose` `noseCrest`
- 입술 : `OuterLips` `innerLips` 

![face_landmarks](https://user-images.githubusercontent.com/7419790/94780635-bd2a2180-0403-11eb-9ddf-ef3c78f406ae.jpg)


### 2. 기준 Lankmark 선택
Face Landmark 포인트 중에서 `OUTER EYES AND NOSE` 기준의 Face Alignment를 수행하기 위해 필요한 좌표값은 `leftEye[0]`, `rightEye[4]`, `nose[4]` 이다.

이 때 `Vision`과 `CoreImage` 프레임워크는 y축의 방향이 서로 반대인 좌표 시스템을 사용한다는 점에 유의하여야 한다. 여기서는 이후에 수행 할 `CoreImage` 기반의 연산을 편리하게 하기 위해 미리 좌표 변환하여 저장하였다.

![coordinate_system](https://user-images.githubusercontent.com/7419790/94823105-6ee64400-043e-11eb-84a3-201ed887cdb5.jpg)

![face_landmarks_selection](https://user-images.githubusercontent.com/7419790/94782434-2a3eb680-0406-11eb-94be-29a207558b4f.jpg)
``` swift
// Left Outer Eye
let x1 = Double(leftEye[0].x)
let y1 = Double(originalImageSize.height - leftEye[0].y)

// Right Outer Eye
let x2 = Double(rightEye[4].x)
let y2 = Double(originalImageSize.height - rightEye[4].y)

// Nose
let x3 = Double(nose[4].x)
let y3 = Double(originalImageSize.height - nose[4].y)
```

### 3. Transform Matrix 얻기
Transform Matrix를 구하기 위해 행렬의 각 요소를 미지수로 하는 6원 연립방정식을 정의한다.

연립방정식은 다음과 같이 행렬로 나타낸 뒤 역행렬을 이용하여 해를 구한다. 이 때 역행렬 연산 및 행렬 곱 연산은 `Accelerate` 프레임워크의 `LinearAlgebra` 관련 함수를 이용하여 성능을 최적화 한다.

``` swift
// 역행렬 연산 함수
func invert(_ matrix : [Double]) -> [Double] {

	var inMatrix = matrix
	var N = __CLPK_integer(sqrt(Double(matrix.count)))
	var pivots = [__CLPK_integer](repeating: 0, count: Int(N))
	var workspace = [Double](repeating: 0.0, count: Int(N))
	var error : __CLPK_integer = 0

	withUnsafeMutablePointer(to: &N) {
		dgetrf_($0, $0, &inMatrix, $0, &pivots, &error)
		dgetri_($0, &inMatrix, $0, &pivots, &workspace, $0, &error)
	}

	return inMatrix
}

// 행렬 곱 연산 함수
func multiply(_ matrixA: [Double], _ matrixB: [Double], _m: Int, _k: Int, _n:Int) -> [Double] {

	var matrixC = [Double](repeating: [Double](repeating: 0, count: n), count: m)
	cblas_dgemm(CblasRowMajor, CblasNoTrans, CblasNoTrans,
				m, n, k,
				1.0, matrixA, k,
				matrixB, n,
				0.0, &matrixC, matrixC.count)
	
	return matrixC
}
```

``` swift
let m = 6
let k = 6
let n = 1

// m x k
let matrix = [	x1, y1, 1,  0,  0, 0,
				 0,  0, 0, x1, y1, 1,
				x2, y2, 1,  0,  0, 0,
				 0,  0, 0, x2, y2, 1,
				x3, y3, 1,  0,  0, 0,
				 0,  0, 0, x3, y3, 1	]

// k * n			 
let vector = [	18.63907, 16.24962,
				75.73048, 15.18443,
				47.51528, 49.38637	]

let answerMatrix = multiply(invert(matrix), vector, m, k, n)

var alignMatrix = CGAffineTransform()
alignMatrix.a 	= CGFloat(answerMatrix[0])
alignMatrix.b 	= CGFloat(answerMatrix[3])
alignMatrix.c 	= CGFloat(answerMatrix[1])
alignMatrix.d 	= CGFloat(answerMatrix[4])
alignMatrix.tx 	= CGFloat(answerMatrix[2])
alignMatrix.ty 	= CGFloat(answerMatrix[5])
```

### 4. Affine Transform 수행
앞에서 연산된 변환 행렬을 이용하여 CoreImage에서 제공하는 Affine Transform 및 Crop을 수행하면 최종 Face Alignment 결과물을 얻을 수 있다.

``` swift
let alignedImage = originalImage
					.transformed(by: alignMatrix)
					.cropped(to: CGRect(x: 0, y: 0, width: 96, height: 96))
```
![affine_transform](https://user-images.githubusercontent.com/7419790/94819991-e74b0600-043a-11eb-997f-ab040ecf3ca5.jpg)


## 참고자료
- [애플 개발자 문서 - VNDetectFaceLandmarksRequest](https://developer.apple.com/documentation/vision/vndetectfacelandmarksrequest)
- [OpenFace 프로젝트 페이지](https://cmusatyalab.github.io/openface/)
