---
layout: post
toc: true
title: "iOS의 코드 기반 Autolayout"
categories: ios
tags: [ios, swift, autolayout, storyboard, ui]
---

## Interface Builder vs 코드 기반 Autolayout
iOS 개발에 입문할 때 보통 Xcode의 Interface Builder 상에서 Autolayout을 이용한 UI 요소의 배치 방법을 배우게 될 것이다.

간단한 프로젝트에서는 이 방법으로 특별히 문제가 없겠지만 규모가 크고 여러 개발자가 협업하는 프로젝트에서는 어떤 방식으로 UI를 구현할 것인지는 중요한 의사결정 사항 중 하나이다.

이러한 이슈에 대한 이해를 돕기 위해 Interface Builder 기반 Autolayout의 장점과 단점을 정리하여 보았다.

> 장점
> - 직관적으로 Constraint를 배치 가능
> - 컴파일 없이 결과물이 곧바로 시각화 됨 (디자이너와 소통 용이)
> - 레이아웃 관련 코드 생략되어 코드가 간결해 짐

> 단점
> - 형상관리에서 diff를 통한 변경사항 파악 어려움
> - .xib 또는 .storyboard 파일을 포함하여 각 UI 모듈마다 복수개의 파일을 관리해야 함

현재는 각 개발자와 조직별로 위 사항을 고려하여 주어진 개발 여건에 맞게 Interface Builder 또는 코드 기반의 Autolayout을 적용하는 것으로 양분되어 있지만

조만간 SwiftUI를 이용한 UI개발이 보편화 되면 더 이상 이러한 고민이 필요해지지 않을 것으로 예상된다.



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

UIView에는 6가지(top, bottom, leading, trailing, width, height) Anchor가 존재하고 각 Anchor에 상수 또는 다른 Anchor와의 관계를 constrain 함수로 지정한다.

만들어진 Constraint는 isActive 속성을 true로 바꾸어 활성화 시켜주어야만 런타임에서 반영된다. Constraint를 활성화하기 이전에는 반드시 addSubView를 통해 부모 뷰에 등록 되어야 하는데 그렇지 않은 경우 런타임 에러가 발생하게 되므로 주의한다.

``` swift 
// 방법1. 하나의 Constraint 활성화
subView.topAnchor.constraint(equalTo: parentView.topAnchor).isActive = true
```

``` swift 
// 방법2. 동시에 여러 Constraint 활성화
NSLayoutConstraint.activate([
    subView.topAnchor.constraint(equalTo: parentView.topAnchor),
    subView.bottomAnchor.constraint(equalTo: parentView.bottomAnchor),
    subView.leadingAnchor.constraint(equalTo: parentView.leadingAnchor),
    subView.trailingAnchor.constraint(equalTo: parentView.trailingAnchor)
])
```

### 3. Priority 지정
Constraint에 Priority를 지정하는 경우 메소드 체이닝을 이용하여 생성과 동시에 지정하는 것은 기본적으로 불가하며 Constraint를 변수에 저장 후 priority 속성을 변경하여야 한다.

만일 Priority 지정이 잦은 경우 위와 같은 방법으로는 불필요한 변수 선언이 코드의 가독성으 해칠 수 있으므로 extension을 활용하에 메소드 체이닝 형태로 지정할 수 있도록 한다.

``` swift
// 방법1. 변수 대입 후 속성 변경
let constraint = subView.widthAnchor.constraint(equalToConstant: 300)
constraint.priority = UILayoutPriority(1000)
constraint.isActive = true
```

``` swift
// 방법2. 메소드 체이닝을 위한 extension 활용
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
지금까지 Apple에서 제공하는 SDK 환경에서의 코드 기반 Autolayout 방법을 알아보았다.

만일 코드가 다소 길게 느껴진다면 좀 더 간결하게 작성하기 위해 [SnapKit](https://github.com/SnapKit/SnapKit), [Cartography](https://github.com/robb/Cartography)와 같은 Wrapper 라이브러리를 활용하는 방안도 생각해 볼 수 있다.


## 참고자료
- [애플 개발자 문서 - translatesAutoresizingMaskIntoConstraints](https://developer.apple.com/documentation/uikit/uiview/1622572-translatesautoresizingmaskintoco)
- [애플 개발자 문서 - Programmatically Creating Constraints](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/ProgrammaticallyCreatingConstraints.html)
