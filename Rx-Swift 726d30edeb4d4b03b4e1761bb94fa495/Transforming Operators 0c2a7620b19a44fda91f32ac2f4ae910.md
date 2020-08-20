# Transforming Operators

## toArray

- 옵저버블이 방출하는 모든 요소를 하나의 배열로 방출

```swift
func toArray() -> Single<[Self.Element]>

let pSubject = PublishSubject<Int>()

subject
	.toArray()
	.subscribe { print($0) }
	.disposed(by: disposeBag)

subject.onNext(1)
subject.onNext(2)
subject.onCompleted

//print(success([1,2])
```

## map

- 옵저버블이 방출하는 요소를 대상으로 함수를 실행하고 결과를 새로운 옵저버블로 리턴
- 클로저를 실행해서 새로운값을 리턴할때 형식에 대한 제약은 없음.

```swift
let strArray = ["Rx", "Swift", "Objc"]
Observable
    .from(strArray)
    .map {"Hello \($0)"} //sr
    .map { $0.count} // str -> int
    .subscribe { print($0) }
    .disposed(by: disposeBag)
```

## flatMap

- 원본 옵저버블의 방출값을 확인하고 새로운 옵저버블을 방출
- 하나의 옵저버블 → 다수의 옵저버블 → 하나의 옵저버블

```swift
let a = BehaviorSubject(value: 1)
let b = BehaviorSubject(value: 2)

let subject = PublishSubject<BehaviorSubject<Int>>()

subject
    .flatMap { $0.asObservable() }
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

subject.onNext(a)
subject.onNext(b)
a.onNext(10)
b.onNext(20)

//next(1)
//next(2)
//next(10)
//next(20)
```

## flatMapFirst, flatMapLatest

```swift
// #flatMapFirst
// 첫번째로 변환된 옵저버블만 방출하고 두번째 옵저버블은 무시.

let a = BehaviorSubject(value: 1)
let b = BehaviorSubject(value: 2)
let subject = PublishSubject<BehaviorSubject<Int>>()

subject
    .flatMapFirst { $0.asObservable() }
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

subject.onNext(a)
subject.onNext(b)
a.onNext(10)
b.onNext(20)
//next(1)
//next(10)

// #flatMapLatest
// 최근 변환된 옵저버블의 이벤트만 구독.

let a = BehaviorSubject(value: 1)
let b = BehaviorSubject(value: 2)
let subject = PublishSubject<BehaviorSubject<Int>>()

subject
    .flatMapLatest { $0.asObservable() }
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

subject.onNext(a)
a.onNext(10)
b.onNext(50)
subject.onNext(b)
b.onNext(5)
a.onNext(20)

//next(1)
//next(10)
//next(50)
//next(5)
```

## scan, reduce

- accumulator 클로저가 실행된 결과값을 방출
- 중간 처리값이 전부 필요할때는 scan 연산자
- 최종 결과값만 필요할 경우는 reduce 연산자

```swift
// #scan
Observable.range(start: 1, count: 3)
    .scan(0, accumulator: + )
    .subscribe { print($0) }
    .disposed(by: disposeBag)
//next(1)
//next(3)
//next(6)
//completed

// #reduce
Observable.range(start: 1, count: 3)
    .reduce(0, accumulator: { $0 + $1})
    .subscribe { print($0) }
    .disposed(by: disposeBag)

//next(6)
//completed

Observable.range(start: 1, count: 3)
    .reduce(0, accumulator: { $0 + $1}, mapResult: { "count \($0)"})
    .subscribe { print($0) }
    .disposed(by: disposeBag)

//next(count 6)
//completed
```

## buffer

- 특정주기동안 항목을 수집하고 하나의 배열로 방출 (Controlled  Buffering)

```swift
Observable<Int>
    .interval(.milliseconds(1000), scheduler: MainScheduler.instance)
    .buffer(timeSpan: .seconds(3), count: 3, scheduler: MainScheduler.instance)
    .take(5)
    .subscribe { print($0) }
    .disposed(by: disposeBag)

//next([0, 1])
//next([2, 3, 4])
//next([5, 6, 7])
//completed
```

## window

- 원본 옵저버블을 작은단위의 옵저버블로 리턴.
- 방출시점과 완료시점의 이해가 중요함.
- inner Observable → 옵저버블이 방출하는 옵저버블.

```swift
Observable<Int>
    .interval(.seconds(1), scheduler: MainScheduler.instance)
    .window(timeSpan: .seconds(5), count: 3, scheduler: MainScheduler.instance)
    .take(3)
    .subscribe {
        if let observable = $0.element {
            observable
                .subscribe { print($0) }
                .disposed(by: disposeBag)
        }
    }
    .disposed(by: disposeBag)

//next(0)
//next(1)
//next(2)
//completed
//next(3)
//next(4)
//next(5)
//completed
//next(6)
//next(7)
//next(8)
//completed
```

## groupBy

- 방출되는 요소를 조건에 따라서 그룹핑.

```swift
let words = ["Apple", "Banana", "Orange", "Book"]

// 글자수에 따른 그룹핑
Observable
    .from(words)
    .groupBy { $0.count}
    .subscribe(onNext: { groupObserver in
        print("== \(groupObserver.key)")
        groupObserver
            .subscribe(onNext: { print($0) })
            .disposed(by: disposeBag)
    })
    .disposed(by: disposeBag)

// 앞글자를 기준으로 그룹핑
Observable
    .from(words)
    .groupBy { $0.first ?? Character(" ")}
    .flatMap{ $0.toArray() }
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)

// 짝수 / 홀수를 나누는 그룹
Observable<Int>.range(start: 1, count: 10)
    .groupBy { $0.isMultiple(of: 2) }
    .flatMap { $0.toArray() }
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)
```