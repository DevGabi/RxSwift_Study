# Time-based Operators

## interval

- 지정된 주기마다 정수를 방출
- 종료시점이 없기때문에 사용자가 dispose() 호출하기 전까지 방출
- 구독이 시작되는 시점에 내부에 새로운 타이머가 실행

```swift
let i = Observable<Int>
    .interval(.seconds(1), scheduler: MainScheduler.instance)
    .subscribe { print($0) }

DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
    i.dispose()
}
```

## timer

- 지연 시간과 반복 주기를 지정해서 정수를 방출
- period ( 반복주기 )에 따라서 동작방식이 다름
    - 지연시간 이후 반복주기에 따라 한번씩 이벤트를 방출

```swift
let i = Observable<Int>
    .timer(.seconds(1), period: .seconds(2), scheduler: MainScheduler.instance)
    .subscribe { print($0) }

DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
    i.dispose()
}
```

## timeout

- 지정된 시간 이내에 요소를 방출하지 않으면 에러 이벤트를 전달
- error 대신 다른 옵저버블로 변경도 가능.

```swift
let subject = PublishSubject<Int>()

subject.timeout(.seconds(3), other: Observable.just(100), scheduler: MainScheduler.instance)
    .subscribe { print($0) }
    .disposed(by: disposeBag)

Observable<Int>
    .timer(.seconds(2), period: .seconds(5), scheduler: MainScheduler.instance)
    .subscribe (onNext: { subject.onNext($0) })
    .disposed(by: disposeBag)
```

## delay

- Next 이벤트가 전달되는 시점과 구독이 시작되는 시점을 지연
- delay - 이벤트 방출 후 구독되는 시점 사이를 지연
- delaySubscription - 이벤트 방출을 지연

```swift
// #delay
Observable<Int>
    .interval(.seconds(1), scheduler: MainScheduler.instance)
    .take(3)
    .delay(.seconds(3), scheduler: MainScheduler.instance)
    .subscribe { print($0) }
    .disposed(by: disposeBag)

// #delaySubscription
Observable<Int>
    .interval(.seconds(1), scheduler: MainScheduler.instance)
    .take(3)
		.delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
    .subscribe { print($0) }
    .disposed(by: disposeBag)
```