# Conditional Operators

## amb

- 여러 옵저버블 중에서 가장 먼저 이벤트를 방출하는 옵저버블을 선택

```swift
//Type Method, Instance Method
let a = BehaviorSubject<Int>(value: 0)
let b = BehaviorSubject<Int>(value: 1000)

//AMB - Inner 가 아니어도 됌. -> 제약이 있음.
Observable.amb([b, a])
//.a.amb(b)
.subscribe { print($0) }
    .disposed(by: disposeBag)

b.onNext(3000)
a.onNext(1)
b.onNext(3000)
a.onNext(2)

//next(1000)
//next(3000)
//next(3000)

```