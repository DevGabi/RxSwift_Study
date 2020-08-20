# AsyncSubject

- Subscribe와 동시에 이벤트를 전달하지 않음.
- onCompleted Event가 전달될 때, 그 시점의 최근 Next Event 전달.
- Event가 없을 경우는 Completed만 실행하고 종료.

```swift
let subejct = AsyncSubject<Int>()

subject.subscribe { print($0) }
       .disposed(by: bag)

subject.onNext(1)
subject.onNext(2)
subject.onNext(3)

=============

subject.onCompleted()

//next(3)
//Completed
```