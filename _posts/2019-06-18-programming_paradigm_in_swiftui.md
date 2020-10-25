---
layout: post
toc: true
title: "SwiftUI를 통해 보는 프로그래밍 패러다임"
categories: ios
tags: [ios, swift, ui, wwdc19]
---

이번 **WWDC19**에서 iOS의 `UIKit`과 MacOS의 `AppKit`을 대체할 새로운 UI 개발 프레임워크인 `SwiftUI`가 등장하였다. 애플은 7개나 되는 세션을 `SwiftUI`와 관련된 내용으로 채운 만큼 이번 **WWDC19**에서 가장 주요한 내용 중 하나로 포지셔닝하였다.

![SwiftUI](https://user-images.githubusercontent.com/7419790/95643315-74502800-0ae9-11eb-801a-c1bc555b9287.jpg)

가장 최신의 언어 중 하나인 `Swift`가 최신의 프로그래밍 패러다임을 잘 담고 있듯이 `SwiftUI` 또한 애플의 뛰어난 개발자들이 여러 의미 있는 소프트웨어적 개념들을 반영한 결과물이다. 그렇기 때문에 `SwiftUI`의 형태와 구성 하나하나를 음미해보는 것은 최신의 프로그래밍 패러다임의 흐름을 파악할 수 있는 좋은 방법이기도 하다.

이러한 의미에서 `SwiftUI`에 녹아있는 주요 프로그래밍 패러다임에 대해 정리해 보았다. 프로그래밍 패러다임은 특정 언어나 프레임워크에 국한된 개념이 아니라 컴퓨터 프로그래밍이라는 공통적으로 사고 방식이다. 따라서 단순히 `UIKit`을 대체하는 개념으로써의 `SwiftUI`에 익숙해지는 것보다 훨씬 더 `SwiftUI`에 대한 이해를 높일 수 있을 것이다. 그리고 `SwiftUI`기반의 새로운 iOS 프로그래밍 아키텍처를 논의하는데에도 탄탄한 바탕이 될 것이다.


## 1. 선언형 프로그래밍
`SwiftUI`의 가장 기본이 되는 패러다임은 선언형 프로그래밍(Declarative Programming)이다. 애플도 여러 세션에서 `SwiftUI`가 선언형임을 반복적으로 강조 하는만큼 이 개념에 대해서는 충분히 이해할 필요가 있다.

선언형 프로그래밍은 명령형 프로그래밍(Imperative Programming)과 대비되는 개념으로 프로그램이 수행해야 하는 동작을 순차적으로 나열하는 것이 아니라 결과적으로 구현하고자 하는 것을 명세하는 형태로 작성하는 프로그래밍 방법이다. 좀 더 직관적인 이해를 위해 아래 예제를 참고해보자.

> **명령형으로 아보카도 토스트 만들기**
> 1. 재료를 준비한다 : 아보카도, 빵, 아몬드 버터, 소금, 레드 페퍼
> 2. 도구를 준비한다 : 토스터, 접시, 버터 나이프
> 3. 빵을 살짝 토스터에 데운다
> 4. 접시에 빵을 올려놓는다
> 5. 아몬드 버터를 빵위에 얇게 펴바른다
> 6. 아보카도를 긴 방향으로 반으로 자른다
> 7. 아보카도 씨를 제거한다
> ...

> **선언형으로 아보카도 토스트 만들기**
> 1. 아몬드 버터, 소금, 레드 페퍼를 넣은 아보카도 토스트

이 때 선언형 프로그래밍은 여러 절차들을 단순히 하나로 묶어내 서브 루틴으로 만든 것과는 다르다는 점에 주의하여야 한다. 구현이 아닌 명세의 형태를 가져야 한다.

물론 튜링 머신의 구조를 갖는 컴퓨터 자체가 명령형으로 동작하기 때문에 모든 프로그램을 선언형으로만 작성하는 것은 불가하다. 따라서 선언형 프로그래밍을 위해서는 복잡한 명령형 코드가 추상화된 프레임워크/라이브러리가 필요하다.

`SwiftUI` 또한 Function Builder를 통해 간결한 선언형의 외형을 가지도록 추상화 되어 있으면서 실제 UI요소들을 그리는 동작은 프레임워크 내부의 명령형 코드에 의해 이루어진다. 이렇게 플랫폼마다 조금씩 달라질 수 있는 명령형 코드들이 선언형으로 추상화 된 덕분에  `iOS`, `macOS`, `watchOS`, `tvOS` 모두에서 사용가능한 코드 개발을 가능하게 해준다. 이는 플랫폼의 통합을 추구하는 애플이 `SwiftUI`를 선보인 중요한 하나의 목적이라고 판단된다.


## 2. 함수형 프로그래밍
`SwiftUI`가 외형적으로는 선언형의 특징을 가지고 있다면 내부적인 아키텍처는 함수형 프로그래밍(Functional Programming)의 원칙을 따른다. 함수형 프로그래밍을 설명하는데는 다양한 개념들이 따라오는데 그 중 가장 핵심이 되는 단 하나의 사고 방식을 이야기 하자면 함수를 순수 함수(Pure Function)라는 제한적인 개념으로만 사용한다는 것이다.

순수 함수는 동일한 Input에 대해서는 동일한 Output을 만들어 내며 외부로부터 영향을 주거나 받는 부수 효과(Side-Effet)가 없는 함수이다. 이것은 어떤 단일 기능 구현에 있어서는 상당히 큰 제약으로 작용하지만 전체 아키텍처 구성을 단순화 하는데 있어서는 강력한 이점을 가져다 준다.

``` swift
// SwiftUI Sample Code

struct OrderCell: View {
    var order: CompletedOrder
 
    var body: some View {
        HStack {
            VStack(alignment: .leading) {
                Text(order.summary)
                Text(order.purchaseDate)
                    .font(.subheadline)
                    .forgroundColor(.secondary)
            }
            Spacer()
            if order.includeSalt {
                SaltIcon()
            }
        }
    }
}
```

위의 코드 예제를 참고해보면 `SwiftUI`에서 뷰는 `View` 프로토콜을 따른다. `View` 프로토콜은 `body`라는 프로퍼티를 가지고 있어야 한다는 조건을 가지는데 `body`는 연산 프로퍼티, 즉 함수이다. 뿐만 아니라 뷰는 클래스가 아닌 구조체이기 때문에 내부적인 상태 변경이 불가한 불변적(Immutable) 특성을 가진다.

![diagram](https://user-images.githubusercontent.com/7419790/97068846-29b0ce80-1606-11eb-906f-4ab3247835b4.png)

이러한 구성은 뷰를 위 도식처럼 Input(`order` 프로퍼티)을 받아 `body`를 통해 Output으로 `View`를 출력하는 순수 함수의 형태로 만들어 준다. 그 결과 개발자는 뷰의 내외부 상태를 고려할 필요가 없으며 동일한 데이터에 대해서는 항상 동일한 뷰가 생성되는 것을 보장받는다. 다시 말해 데이터 변경이 발생했을 때 해당 데이터와 관련된 UI요소를 식별하여 업데이트 하는 것이 아니라 순수 함수를 통해 전체의 새로운 뷰를 생성하는 단일 흐름으로 단순화 된다.

이렇게 `SwiftUI`에서 뷰의 순수 함수적이고 불변적인 특성은 객체 지향적(Object-Oriented)인 `UIKit`과 결정적으로 차별화 되는 사항으로 `SwiftUI`를 `SwiftUI`답게 사용하기 위해 반드시 알고 있어야 하는 사항이다.


## 참고자료
- [WWDC19 - SwiftUI Essentials](https://wwdc.io/share/wwdc19/216)
- [WWDC19 - Introducing Combine](https://wwdc.io/share/wwdc19/722)
- [WWDC19 - Data Flow Through SwiftUI](https://wwdc.io/share/wwdc19/226)
- [Wikipedia - Declarative programming](https://en.wikipedia.org/wiki/Declarative_programming)
- [Swift by Sundell - The Swift 5.1 features that power SwiftUI’s API](https://www.swiftbysundell.com/articles/the-swift-51-features-that-power-swiftuis-api/)