# Sharing Subscription

## multicast

- 유니캐스트 방식을 멀티캐스트 방식으로 변경해줌
    - 유니캐스트: 1:1로 데이터를 전달하는 통신방식
    - 멀티캐스트: 특정 그룹에게 데이터를 전달하는 방식
- ConnectableObservable → 구독자가 Connect를 실행하는 순간 시작, 원본옵저버블과 연결된 옵저버블을 하나로 연결해줌.
- 모든 등록자가 구독된 이후에 하나의 시퀀스로 실행하는게 가능.
- Connect → Disposable을 반환

```swift
/*:
 # multicast
 */

let bag = DisposeBag()
let subject = PublishSubject<Int>()

let source = Observable<Int>
    .interval(.seconds(1), scheduler: MainScheduler.instance)
    .take(5)
    .multicast()

source
   .subscribe { print("🔵", $0) }
   .disposed(by: bag)

source
   .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
   .subscribe { print("🔴", $0) }
   .disposed(by: bag)

source.connect().disposed(by: bag)
```

## publish, replay, replayAll

- 연산자 내부에서 subject 생성 후 mutilcast를 사용
- 버퍼 제한이 없는 replayAll 연산자는 되도록 사용하지 않도록 함.

```swift
let bag = DisposeBag()
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
														.take(5)
														//.publish()  
														//.replay(5)  // replaySubject

source
   .subscribe { print("🔵", $0) }
   .disposed(by: bag)

source
   .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
   .subscribe { print("🔴", $0) }
   .disposed(by: bag)

source.connect()
```

## refCount, refCountObservable

- refCount를 통해서 생성하는 옵저버블을 refCountObservable이라고 함.
- rxdocument에서는 connect / disconnect라고 표현.
- refCount연산자를 활용하면 사용자가 직접 connect / dispose를 해줄 필요 없음.

```swift
let bag = DisposeBag()
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).debug().publish().refCount()

let observer1 = source
   .subscribe { print("🔵", $0) }

DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
   observer1.dispose()
}

DispatchQueue.main.asyncAfter(deadline: .now() + 7) {
   let observer2 = source.subscribe { print("🔴", $0) }

   DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
      observer2.dispose()
   }
}
```

## Share

- Share 두개의 파라메터를 받음
    - replay: 해당 값을 토대로 멀티캐스트를 생성
    - scope: 생명주기를 나타냄
        - connectWhile - 모든 커넥션마다 새로운 서브젝트 생성, 커넥션은 다른 커넥션과 격리.
        - forever - 모든 커넥션이 하나의 서브젝트를 공유

```swift
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
                            .debug().share(replay: 5, scope: .forever) // refCountObservable

let observer1 = source
   .subscribe { print("🔵", $0) }

let observer2 = source
    .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
    .subscribe { print("🔴", $0) }

DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
    observer1.dispose()
    observer2.dispose()
}

DispatchQueue.main.asyncAfter(deadline: .now() + 7) {
   let observer3 = source.subscribe { print("🔵🔴", $0) }
   DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
      observer3.dispose()
   }
}
```

```swift
let bag = DisposeBag()

let source = Observable<String>.create { observer in
   let url = URL(string: "https://kxcoding-study.azurewebsites.net/api/string")!
   let task = URLSession.shared.dataTask(with: url) { (data, response, error) in
      if let data = data, let html = String(data: data, encoding: .utf8) {
         observer.onNext(html)
      }
      
      observer.onCompleted()
   }
   task.resume()
   
   return Disposables.create {
      task.cancel()
   }
}
.debug()
.share()

```