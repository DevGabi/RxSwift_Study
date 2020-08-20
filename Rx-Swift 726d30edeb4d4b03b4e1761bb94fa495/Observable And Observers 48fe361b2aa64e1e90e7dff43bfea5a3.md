# Observable And Observers

- Observable 생성 ( 2가지 방법 )

```swift
// Observable<Element> : ObservableType 
// 생성시 제네릭으로 해당 형을 지정해줘야함.

// Create를 이용한 직접 생성
Observable.create(subscribe: (AnyObserver<_>) -> Disposable)

let observable = Observable.create() { (observer) -> Disposable in
   // code 
   // 반환값은 Disposable // 메모리 해제를 위한 객체
}

// 연산자를 사용해서 생성
Observable.from([0,1]) // 배열안의 값을 순차적으로 방출.

// Observable은 Subscribe 되는 시점에 코드가 실행.
```

- Observable → (Event Emission) → Observer
    - Event 종류
        - onNext: 이벤트 발생 ( Emission )
        - onError: 에러 발생 후 Observable 종료. ( Notification )
        - onCompleted: 완료 발생 후 해당 Observable 종료.  ( Notification )

```swift
let observable = Observable<Int>.create() { (observer) -> Disposable in 
    observer.on(.next(0)) X
		observer.onNext(1)		
		onCompleted()
    //onError()

		return Disposables.create() // Disposable 리소스 해제.
}

Observable.just(1)
```

- Observable ← (Subscribe) ← Observer
    - 옵저버는 동시에 두가지 이상의 이벤트를 처리하지는 않음.
    - 하나의 이벤트 처리 이후 다음 이벤트 처리

```swift
// #1 -> subscribe로 사용 해당 값에 접근시 옵셔널 해지 후 사용.
ob.subscribe {
    if let elem = $0.element {
       // 해당 전달받은 엘리먼트 사용
		}
}

// #2 
observable
		.subscribe(onNext: { elem in 
				// elem <- 바로 사용가능.
     } 
     onCompleted: { print("completed")}
oneError:) // print 'completed' 종료.
```