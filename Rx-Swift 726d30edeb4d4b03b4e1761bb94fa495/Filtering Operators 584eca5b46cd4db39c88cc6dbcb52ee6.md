# Filtering Operators

## ignoreElements Operator

- Completed 와 Error에 대한 이벤트만 방출하고 Next 이벤트는 무시함.
- 옵저버블의 성공 or 실패에 대한 부분만 알고 싶을 때 사용.

```swift
Observable.from([1,2,3,4])
    .ignoreElements()
    .subscribe { print($0) }
    .disposed(by: disposeBag)

// print(completed)
```

## elementAt

- 특정 인덱스에 위치한 요소를 제한적으로 방출하는 연산자

```swift
public func elementAt(_ index: Int) -> RxSwift.Observable<Self.Element>

Observable
    .from([1,2,3,4])
    .elementAt(2)
    .subscribe { print($0) }
    .disposed(by: disposeBag)

// next(3)
// completed
```

## filter

- 특정 조건에 맞는 항목을 필터링

```swift
Observable
    .from(list)
    .filter { $0.isMultiple(of: 2)}
    .subscribe { print($0) }
    .disposed(by: disposeBag)

//next(2)
//next(4)
//next(6)
//next(8)
//next(0)
//completed
```

## Skip, SkipWhile, skipUntil

- 특정 요소를 무시.

```swift
//#Skip 
let list = [1,2,3,4,5,6,7,8,9,0]
Observable
    .from(list)
    .skip(8) // skip안의 값이 index가 아니라 count인 부분에 주의.
    .subscribe { print($0) }
    .disposed(by: disposeBag)

//#skipWhile
func skipWhile(_ predicate: @escaping (Self.Element) throws -> Bool) -> RxSwift.Observable<Self.Element>
Observable
    .from(list)
    .skipWhile { !$0.isMultiple(of: 8) } // true일때 무시, false가 시점부터 방출.
    .subscribe { print($0) }
    .disposed(by: disposeBag)

//#skipUntil
// Observable을 파라미터로 받음. 
// Trigger Observable이 요소를 방출한 이후부터 구독 시작.
func skipUntil<Source>(_ other: Source) -> RxSwift.Observable<Self.Element> where Source : RxSwift.ObservableType

let pSubject = PublishSubject<Int>()
let trigger = PublishSubject<Void>() 

pSubject
    .skipUntil(trigger) 
    .subscribe { print($0) }
    .disposed(by: disposeBag)

pSubject.onNext(1)

trigger.onNext(())
pSubject.onNext(3)

//print next(3)
```

## take, takeWhile, takeUntil, takeLast

- 처음 n개의 요소 또는 마지막 n개의 요소를 방출

```swift
let disposeBag = DisposeBag()
enum MyError: Error {
   case error
}
let list = [1,2,3,4,5,6,7,8,9,0]

//#take
Observable
    .from(list)
    .take(2) // take안의 값이 index가 아니라 count인 부분에 주의.
    .subscribe { print($0) }
    .disposed(by: disposeBag)

//print next(1)
//print next(2)

//#takeWhile
Observable
    .from(list)
    .takeWhile { !$0.isMultiple(of: 2) } // true가 반환되는 순간까지만 방출.
    .subscribe { print($0) }
    .disposed(by: disposeBag)

//next(1)
//completed

// takeUntil
let pSubject = PublishSubject<Int>()
let trigger = PublishSubject<Void>()

pSubject
    .takeUntil(trigger)
    .subscribe { print($0) }
    .disposed(by: disposeBag)

pSubject.onNext(1)
trigger.onNext(())
pSubject.onNext(3)

//next(1)
//completed

//takeLast
pSubject
    .takeLast(3) // 3개의 값을 observer가 종료될때 방출.
    .subscribe { print($0) }
    .disposed(by: disposeBag)

list.forEach { pSubject.onNext($0) }
pSubject.onNext(13)
pSubject.onNext(14)
pSubject.onNext(14)
pSubject.onNext(14)
pSubject.onNext(14)
pSubject.onNext(14)
pSubject.onNext(14)
pSubject.onNext(14)
pSubject.onNext(14)
pSubject.onNext(14)
pSubject.onNext(14)
pSubject.onCompleted()
//next(0)
//next(13)
//next(14)
//completed
pSubject.onError(MyError.error) // 에러 방출시 error만 방출
//error
```

## single

- 하나의 요소가 방출되고, 2개이상이 방출될 때는 1개의 요소를 방출하고 error를 방출.
- 반드시 한개의 요소만 방출하는걸 보장해야 될 때 사용.

```swift
let list = [1,2,3,4,5,6,7,8,9,0]
//#1
Observable
    .just(1)
//		.from(list)
    .single()
    .subscribe { print($0) }
    .disposed(by: disposeBag)
//next(1)
//completed
//next(1)
//error(Sequence contains more than one element.)

//#2
Observable
    .from(list)
    .single { $0 == 3}
    .subscribe { print($0) }
    .disposed(by: disposeBag)

//next(3)
//completed

let pSubject = PublishSubject<Int>()
//
pSubject
    .single()
    .subscribe (onNext: {}, onCompleted: {}, onError: {})
      //얼럿처리
	})
    .disposed(by: disposeBag)

pSubject.onNext(2)
//next(2)
//completed or Error
pSubject.onCompleted()
pSubject.onNext(3)
```

## distinctUntilChanged

- 동일한 요소가 연속적으로 방출되는 것을 막아주는 연산자

```swift
let list = [1,1,1,1,1,1,5,4,4,2,1]
Observable
    .from(list)
    .distinctUntilChanged()
    .subscribe { print($0) }
    .disposed(by: disposeBag)

//next(1)
//next(5)
//next(4)
//next(2)
//next(1)
//completed
```

## debounce, throttle

- 짧은 시간 반복적으로 전달되는 이벤트를 효율적으로 처리
- Debounce
    - 지정된시간동안 이벤트가 없으면 그때 최근 Next이벤트를 방출
    - 시간안에 이벤트가 발생하면 타이머를 초기화
- throttle
    - 지정된 주기마다 Next이벤트를 방출

```swift
// #code
let disposeBag = DisposeBag()

let buttonTap = Observable<String>.create { observer in
   DispatchQueue.global().async {
      for i in 1...10 {
         observer.onNext("Tap \(i)")
         Thread.sleep(forTimeInterval: 0.3)
      }
      Thread.sleep(forTimeInterval: 1)
      for i in 11...20 {
         observer.onNext("Tap \(i)")
         Thread.sleep(forTimeInterval: 0.5)
      }
      observer.onCompleted()
   }
   return Disposables.create {
   }
}

// #debounce
// 지정된시간동안 이벤트가 없으면 그때 최근 Next이벤트를 방출

func debounce(_ dueTime: RxSwift.RxTimeInterval, scheduler: RxSwift.SchedulerType) -> RxSwift.Observable<Self.Element>

buttonTap
    .debounce(.milliseconds(1000), scheduler: MainScheduler.instance)
    .subscribe { print($0) }
    .disposed(by: disposeBag)

//next(Tap 10)
//next(Tap 20)
//completed

//# throttle
func throttle(_ dueTime: TimeInterval, latest: Bool = true, scheduler: RxSwift.SchedulerType) -> RxSwift.Observable<Self.Element>

// 지정된시간마다 Next이벤트를 방출.
buttonTap
    .throttle(.milliseconds(1000), scheduler: MainScheduler.instance)
    .subscribe { print($0) }
    .disposed(by: disposeBag)

//next(Tap 1)
//next(Tap 4)
//next(Tap 7)
//next(Tap 10)
//next(Tap 11)
//next(Tap 12)
//next(Tap 14)
//next(Tap 16)
//next(Tap 18)
//next(Tap 20)
//completed

buttonTap
    .throttle(.milliseconds(1000), latest: false,  scheduler: MainScheduler.instance)
    .subscribe { print($0) }
    .disposed(by: disposeBag)

// throttle - latest값에 따른 이벤트 방출시점
// true일 경우는, 시간에 맞춰서 방출.
// false일 경우는 시간이 경과한 이후 발생하는 최근 Next이벤트를 방출.

Observable<Int>.interval(.milliseconds(1000), scheduler: MainScheduler.instance)
    .take(10)
    .throttle(.milliseconds(1000), latest: false, scheduler: MainScheduler.instance)
    .subscribe { print($0) }
    .disposed(by: disposeBag)

//next(0)
//next(2)
//next(4)
//next(5)
//next(6)
//next(8)
//completed
```