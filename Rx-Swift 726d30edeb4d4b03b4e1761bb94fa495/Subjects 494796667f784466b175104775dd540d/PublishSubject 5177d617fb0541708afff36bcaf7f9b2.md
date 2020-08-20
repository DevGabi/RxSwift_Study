# PublishSubject

- Subscribe 이전의 Event는 무시.
- Subscribe 이후의 Event만 구독.

```swift
let subject = PublishSubject<String>()
subject.onNext("Hello") // 
let o1 = subject.subscribe { print(">> 1", $0 }
                .disposed(by: disposeBag)

// --------

subject.onNext("RxSwift") 
// >> 1 next(RxSwift)

let o2 = subject.subscribe { print(">> 2", $0 }
                .disposed(by: disposeBag)

subject.onNext("Subject")
// >> 1 next(RxSwift)
// >> 2 next(RxSwift)

subject.onCompleted()
// >> 1 completed
// >> 2 completed

subject.onError(MyError.error)

let o3 = subject.subscribe { print(">> 3", $0 }
                .disposed(by: disposeBag)

// >> 3 completed

```