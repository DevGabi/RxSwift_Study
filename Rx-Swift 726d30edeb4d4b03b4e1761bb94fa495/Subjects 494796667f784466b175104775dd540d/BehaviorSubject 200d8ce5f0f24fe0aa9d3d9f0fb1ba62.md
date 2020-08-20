# BehaviorSubject

- 초기에 Buffer를 한개 저장함.
- Subscribe와 동시에 초기값을 전달해주기 때문에 Property를 세팅할때 가장 많이 사용.

```swift
let p = PublishSubject<Int>()
p.subscribe { print("PublishSubject >> ", $0) }
 .disposed(by: disposeBag)

let b = BehaviorSubject<Int>(value: 0)
b.subscribe { print("BehaviorSubject >> ", $0) }
 .disposed(by: disposeBag)
b.onNext(1)

b.subscribe { print("BehaviorSubject2 >> ", $0) }
 .disposed(by: disposeBag)

// print BehaviorSubject >> next(0)
// print BehaviorSubject >> next(1)
// print BehaviorSubject2 >> next(1)

b.onCompleted()
// print BehaviorSubject >> completed
// print BehaviorSubject2 >> completed

b.subscribe { print("BehaviorSubject3 >> ", $0) }
 .disposed(by: disposeBag)
// print BehaviorSubject3 >> completed
```