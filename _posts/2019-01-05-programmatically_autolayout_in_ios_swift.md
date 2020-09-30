---
layout: post
toc: true
title: "iOS의 코드 기반 Autolayout"
categories: ios
tags: [ios, swift, autolayout, storyboard, ui]
---

## 스토리보드 vs 코드 기반 Autolayout
iOS 개발에 입문할 때 보통 Xcode의 스토리보드 상에서 Autolayout을 이용한 UI 요소의 배치 방법을 배우게 될 것이다.
간단한 프로젝트에서는 스토리보드 사용에 따른 문제가 특별히 없겠지만 규모가 크고 여러 개발자가 협업하는 프로젝트에서는
어떤 방식으로 UI를 구현할 것인지는 중요한 의사결정 사항 중 하나이다.
이해를 돕기 위해 스토리보드 기반 Autolayout의 장점과 단점을 정리하여 보았다.

> 장점
> - 직관적으로 Constraint를 배치 가능
> - 컴파일 없이 결과물이 곧바로 시각화 됨 (디자이너와 소통 용이)
> - 레이아웃 관련 코드 생략되어 코드가 간결해 짐

> 단점
> - 형상관리에서 diff를 통한 변경사항 파악 어려움
> - .xib 또는 .storyboard 파일을 포함하여 각 UI 모듈마다 복수개의 파일을 관리해야 함

현재는 각 개발자와 조직별로 주어진 개발 여건에 맞게 스토리보드 또는 코드 기반의 Autolayout을 적용하는 것으로 양분되어 있지만
조만간 SwiftUI를 이용한 개발이 보편화 되면 더 이상 이러한 의사결정이 필요해지지 않을 것으로 예상된다.




## 코드 기반 Autolayout 방법


### 1. UIView 속성 설정
코드 기반의 Autolayout을 하고자 하는 UIView(또는 UIView를 상속한 클래스)에 대해서는 translatesAutoresizingMaskIntoConstraints 속성을 false로 지정할 필요가 있다.

이 속성은 뷰의 크기가 부모 뷰의 크기 변경에 따라 Flexible하게 자동 조정되는 기능을 사용할지 여부를 결정하는데 기본값은 true를 가진다.
이 값이 true인 경우 명시적으로 지정한 모든 Constraint가 무시되므로 반드시 false로 설정해야 하는 것이다.

``` swift
var view = UIView()
view.translatesAutoresizingMaskIntoConstraints = false
```

### 2. Constraint 지정
iOS에서 UIView의 Constraint 지정하는 것은 여러 방법이 있으나 iOS 9.0에 추가된 가장 최신의 방법 기준으로 설명하겠다.

UIView에는 6가지(top, bottom, leading, trailing, width, height) Anchor가 존재하고 각 Anchor에 상수 또는 다른 Anchor와의 관계를 지정하는 방법으로 
Constraint의 종류는 함수의 파라미터명만 보아도 각각을 이해하고 활용하는데 어려움은 없을 것이다.

Constraint를 지정하기 전 주의해야 할 한가지 사항은 Constraint를 활성화하기 이전에 반드시 addSubView를 통해 부모 뷰에 등록 되어 있어야 한다는 점이다. 그렇지 않은 경우 런타임 에러가 발생하게 된다.

``` swift 
// 하나의 Constraint 활성화
subView.topAnchor.constraint(equalTo: parentView.topAnchor).isActive = true
```

``` swift 
// 동시에 여러 Constraint 활성화
NSLayoutConstraint.activate([
    subView.topAnchor.constraint(equalTo: parentView.topAnchor),
    subView.bottomAnchor.constraint(equalTo: parentView.bottomAnchor),
    subView.leadingAnchor.constraint(equalTo: parentView.leadingAnchor),
    subView.trailingAnchor.constraint(equalTo: parentView.trailingAnchor)
])
```

### 3. Priority 지정
``` swift
extension NSLayoutConstraint {
    
    func with(_ priority: UILayoutPriority) -> NSLayoutConstraint {
        self.priority = priority
        return self
    }
    
}

NSLayoutConstraint.activate([
    subView.widthAnchor.constraint(equalToConstant: 300).with(UILayoutPriority(1000)),
    subView.trailingAnchor.constraint(equalTo: parentView.trailingAnchor).with(UILayoutPriority(700)),
])
```

## 결론


## 참고자료
[애플 개발자 문서 - translatesAutoresizingMaskIntoConstraints](https://developer.apple.com/documentation/uikit/uiview/1622572-translatesautoresizingmaskintoco)
