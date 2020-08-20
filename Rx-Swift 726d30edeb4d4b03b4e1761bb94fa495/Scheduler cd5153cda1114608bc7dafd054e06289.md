# Scheduler

- iOS: GCD
- RxSwift: Scheduler
- 스레드 지정이 없을 경우 currentThread 사용
- observeOn: runOn
- subscribeOn: startOn

```swift
// 쓰레드, 큐, 스케쥴러 // 레퍼런스 사이클 걱정 없음. 동기화 걱정 없음.
        let globalQueue = DispatchQueue.global()
        let globalScheduler = ConcurrentDispatchQueueScheduler(queue: globalQueue)

        //중요
        // #1 생성
        // #2 순서
        Observable.just(1) // StartOn, emitOn, generateOn
            .subscribeOn(globalScheduler) 
            .map { (num: Int) -> String in
                if Thread.isMainThread {
                    print("Main Thread")
                } else {
                    print("Background Thread")
                }
                return "\(num)"
        }
            .observeOn(MainScheduler.instance) // runOn -> 실제는 BindTo
            .subscribe {

                if Thread.isMainThread {
                    print("Subscribe Main Thread")
                } else {
                    print("Subscribe Background Thread")
                }
                print($0)
        }
            .disposed(by: disposeBag)
```