---
layout: post
toc: true
title: "단방향 데이터 흐름의 ViewModel"
categories: ios
tags: [ios, swift, ui]
---

MVVM`(Model-View-ViewModel)`은 2005년 Microsoft WPF 프레임워크에서 제안된 아키텍처로 GUI 기반 어플리케이션의 Business Logic과 View의 관계를 정의한다. 최근에는 안드로이드 및 iOS와 같은 모바일 앱 개발 진영에서도 First Party에서 공식적으로 권장하는 MVC`(Model-View-Constroller)`아키텍처의 단점을 개선하기 위해 MVVM이 빠르게 도입되고 있다.

이렇게 MVVM에 대한 관심이 많아지면서 MVVM이 가진 문제점들 또한 하나 둘 제기되고 있는데 대표적으로는 ViewModel의 비대화, ViewModel의 역할 정의 불명확 등이 있다. 이는 대부분 MVVM이라는 아키텍처에 대해 개발자들간의 개념 정의와 이해도가 일치하지 않아서 발생하는 문제라고 생각한다.

따라서 본 포스팅에서는 이 문제에 대한 하나의 솔루션으로 MVVM의 핵심이 되는 ViewModel을 단방향 데이터 흐름을 갖도록 엄격하게 정의하는 방법을 제안하고자 한다. 이를 이용한다면 전체 코드에 걸쳐 일관적인 MVVM 아키텍처를 구성하는데 효과적일 것으로 기대한다.


## iOS에서의 MVVM
![1*iwgAHz3uZGqyk3OhOOjgyg](https://user-images.githubusercontent.com/7419790/94890022-0be5c300-04b9-11eb-8b01-8e1dd8c32bd4.jpeg)

iOS 에서의 MVVM은 기본적으로 `View`와 `Controller`가 결합된 `ViewController`를 `View`로써 역할을 제한하고 Business Logic 부분을 `ViewModel`에 따로 떼어낸 형태를 가진다.

부가적으로 데이터 바인딩의 개념이 적용되어 있으나 기본적으로 MVC 구조에서 크게 벗어나지 않았기 때문에 단일 책임 원칙과 테스트 용이성 등의 이점을 가져가면서도 기존 코드에서의 마이그레이션도 어렵지 않다.


## 표준 MVVM 아키텍처의 부재
MVVM을 구현할 때 가장 `ViewModel`이다. `ViewModel`은 Model에서 데이터를 받아 View에서 View는 바인딩 하여 

- Delegation
- Property Observer
- Closure
- Reactive Programming (RxSwift)

최근에는 RxSwift로 대표되는 Reactive 프로그래밍의 개념도 도입되면서 
Subject는 많은 로직들을 
이렇게 하는 경우 


## 단일 데이터 흐름 ViewModel
특히 RxSwift를 사용하는 경우에 대한 ViewModel 구성을 제안하고자 한다.

먼저 `Input`과 `Output` 타입을 가지는 `ViewModelType` 프로토콜을 정의한다. 그리고 `transform` 함수를 통해 `View`로부터 `Input`을 받아 변환하여 다시 `View`로 `Output`을 제공하는 단일 데이터 흐름을 강제한다.

``` swift
protocol ViewModelType {
    
    associatedtype Input
    associatedtype Output
    
    func transform(from input: Input) -> Output
}
```

모든 `ViewModel`은 `ViewModelType`을 따라야 하는데 이 때 `Input`과 `Output` 타입 내의 프로퍼티는 `var` 대신 `let`을, `Subject`(또는 `Relay`) 대신 `Observable`만 사용하도록 한정한다.

이를 통해 `ViewModel`에서 의도한 `Input`과 `Output`의 데이터 방향성을 `View`에서 오용하는 것을 방지할 수 있다.

``` swift
class ViewModel: ViewModelType {
    
    struct Input {
        let searchText: Observable<String>
        let selectItem: Observable<Int>
    }
    
    struct Output {
        let status: Observable<String>
        let itemList: Observable<[String]>
    }
    
    func transform(from input: Input) -> Output {
        
        let status =
            input.selectItem
                .map { ... }
                ...

        let itemList =
            input.searchText
                .map{ ... }
                ...

        return Output(
            status: status
            itemList: itemList
        )
    }
    
}
```


## ViewController 예제
`ViewController`에서는 `View`가 로드 되는 시점에 UI요소에서 `ViewModel`로 전달할 이벤트를 `Input`으로 생성하고 `transform` 수행 후 만들어진 `Output`을 UI요소에 바인딩 한다.

이로써 `ViewModel`과 `ViewController`간의 단일 데이터 흐름이 완성된다. 한편으로는 모든 데이터 바인딩 관련 코드가 `bind()` 함수 한 곳에서 수행되기 때문에 응집도 향상이라는 부수적인 효과도 가진다.

``` swift
class ViewController: UIViewController {

    var viewModel: ViewModel!

    ...


    override func viewDidLoad() {
        super.viewDidLoad()

        bind()
    }

    func bind() {

        let input = ViewModel.Input(
            searchText: textView.rx.text,
            selectItem: tableView.rx.modelSelected(String.self)
        )

        let output = viewModel.transform(from: input)

        output.status
            .bind( ... )
            .disposed(by: disposeBag)

        output.itemList
            .bind( ... )
            .disposed(by: disposeBag)
        
    }

}
```


## 결론
본 포스팅에서 제시한 MVVM의 패턴은 `View` 초기화 시에 모든 데이터 바인딩을 수행해야 하므로 실무에서는 꽤나 큰 제약으로 느껴질 수도 있다. 하지만 구현 과정에서 희생된 약간의 자유도는 낮은 모듈간 결합도와 테스트를 용이성이라는 이점으로 크게 보상된다. 

앞으로 여러 프로젝트를 진행하면서 이러한 단일 데이터 흐름의 MVVM 패턴이 어느 정도까지 활용 가능할지 계속 검증해 나갈 예정이다.


## 참고자료
- [애플 개발자 문서 - Model-View-Controller](https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/Model-View-Controller/Model-View-Controller.html)
- [Wikipedia - Model–view–viewmodel](https://en.wikipedia.org/wiki/Model–view–viewmodel)
- [MSDN - Introduction to Model/View/ViewModel pattern for building WPF apps](https://docs.microsoft.com/ko-kr/archive/blogs/johngossman/introduction-to-modelviewviewmodel-pattern-for-building-wpf-apps)
- [MSDN - Model-View-ViewModel 디자인 패턴을 사용한 WPF 응용 프로그램](https://docs.microsoft.com/ko-kr/archive/msdn-magazine/2009/february/patterns-wpf-apps-with-the-model-view-viewmodel-design-pattern)