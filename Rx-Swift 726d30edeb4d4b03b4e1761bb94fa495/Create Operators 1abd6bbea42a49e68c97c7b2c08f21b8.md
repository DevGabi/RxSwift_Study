# Create Operators

## Just, of, from

```swift
// #just - 한개의 이벤트를 발생하고 종료.
just(_ element: Self.Element) -> RxSwift.Observable<Self.Element>

Observable
	.just("Hello")
  .subscribe { print($0) }    
	.dispose(by: disposeBag)

// next("Hello")
// completed

Observable
	.just([1, 2, 3]
  .subscribe { print($0) }    
	.dispose(by: disposeBag)

// next([1, 2, 3])
// completed

```

```swift
// #of - 두개 이상의 요소를 방출할때 사용.
of(_ elements: Self.Element..., scheduler: RxSwift.ImmediateSchedulerType = CurrentThreadScheduler.instance) -> RxSwift.Observable<Self.Element>

Observable
	.of("Hello", "Rx", "Swift")
  .subscribe { print($0) }    
	.dispose(by: disposeBag)
//next("Hello")
//next("Rx")
//next("Swift")
//completed

Observable
	.of([1,2],[3,4],[5,6] )
  .subscribe { print($0) }    
	.dispose(by: disposeBag)

//next([1,2])
//next([3,4])
//next([5,6])
//completed
```

```swift
// #from - 배열의 각 요소를 방출시킨후 종료.
from(_ array: [Self.Element], scheduler: RxSwift.ImmediateSchedulerType = CurrentThreadScheduler.instance) -> RxSwift.Observable<Self.Element>
Observable
	.from([[1, 2, 3])
  .subscribe { print($0) }    
	.dispose(by: disposeBag)
//next(1)
//next(2)
//next(3)
//completed

```

## range

- 최초값에서 1씩 증가하는 정수값을 방출.
- 중간에 크기를 변경하거나 감소할 는 시퀀스는 불가능.

```swift
// #range - 시작값에서 1씩 증가하는 정수값을 방출.
Observable.range(start: 1.0, count: )
```

## generate

- 제약조건에 따라서 무한루프에 빠질 수 있음.

```swift
Observable<Int>
	.generate(initialState: 0, condition: { num -> Bool in
            num <= 10 // num.isMutiple(of: 2) <- 무한루프
		}, iterate: { num -> Int in
			num + 2
	})
  .subscribe { print($0) }    
	.dispose(by: disposeBag)	

let a = "a"
let b = "b"

Observable.generate(initialState: a, condition: { str -> Bool in
                    str.count <= 4
                }, iterate: { str -> String in
                    str.count.isMultiple(of: 2) ? str + a : str + b
            })
          .subscribe { print($0) }
          .disposed(by: disposeBag)

//next(a)
//next(ab)
//next(aba)
//next(abab)
//completed
```

## RepeatElement

- 들어온 값을 반복적으로 실행
- 제약조건이 없을 경우 무한반복

```swift
Observable.repeatElement("Hello")
	.take(2) // 제약조건 - 없을 경우 무한 반복
	.subscribe { print($0) }
	.disposed(by: disposeBag)

// next("Hello")
// next("Hello")
// Completed
```

## deferred

- 특정조건에 따라서 옵저버블 생성 가능.
- 동일한 형의 경우만 옵저버블 생성 가능.

```swift
let trueList = ["H","I"]
        let falseList = ["F","A"]
        var flag = true
        
        let ob = Observable<String>.deferred {
            flag.toggle()
            if flag {
                return Observable.from(trueList)
            }
            else {
                return Observable.from(falseList)
            }
        }
        ob
        .subscribe(onNext: { print("print", $0) })
        .disposed(by: disposeBag)
        
        ob
        .subscribe(onNext: { print("print", $0) })
        .disposed(by: disposeBag)

//print F
//print A
// ================================
//print H
//print I
```

## create

- 옵저버블의 동작을 직접 구현함

```swift
enum MyError: Error {
   case error
}

Observable<String>.create { (observer) -> Disposable in

    guard let url = URL(string: "https://apple.com") else {
        observer.onError(MyError.error)
        return Disposables.create()
    }
    
    guard let htmlString = try? String(contentsOf: url, encoding: .utf8) else {
        observer.onError(MyError.error)
        return Disposables.create()
    }
    observer.onNext(htmlString) // 요소를 방출하기 위해서는 onNext를 이용해서 방출.
    observer.onCompleted()
    
    observer.onNext("Hello") // 이벤트를 방출하지 않음.
    
    return Disposables.create()
}
.subscribe { print($0) }
.disposed(by: disposeBag)
```

## empty, error

- next 이벤트를 발생하지 않음.

```swift
//# empty
Observable<Void>.empty()
	.subscribe { print($0) }
	.disposed(by: disposeBag)

//print("completed")

//# error
enum MyError: Error {
   case error
}
Observable<Void>.error(MyError.error)
	.subscribe { print($0) }
	.disposed(by: disposeBag)
//print("error(error)")

```