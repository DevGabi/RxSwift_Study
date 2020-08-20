# Disposables

```swift
//#1
let subscribe = Observable.from([1,2,3])
        .subscribe(onNext: { elem in
            print("Next", elem)
        }, onError: { (Error) in
            print(Error)
        }, onCompleted: {
            print("onCompleted")
        }, onDisposed: {
            print("onDisposed") // () -> void -> 모든 리소스반환이 완료된 후 호출.
        })
        subscribe.dispose() // 직접 해제. ( 직접 호출이 필요한 케이스 ( ex) 타이머 )
      
//#2  
var bag = DisposeBag() 
    Observable.from([1,2,3])
        .subscribe {
             print($0)
        }.disposed(by: bag) // DisposeBag이 해지되는 시점에 담겨있는 모든 리소스 해지
        
// 원하는 시점에 DisposeBag을 해지하고 싶을때 다시 생성 or nil
    bag = DisposeBag()
```

- 직접 해지할 경우 Completed 없이 바로 Disposed 실행되며 옵저버블 종료.
- 직접해지하는 방법보다는 연산자를 이용해서 다른방식으로 구현하는것을 권장함.