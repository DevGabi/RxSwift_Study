# Combining Operators

## startWith

- 기본값 or 시작값을 지정할 때 많이 사용.
- 연산자이기때문에 중복해서 사용 가능.
- 전달순서대로 앞에 추가.

```swift
// Last in First Out

let list = [1,2,3]
Observable.from(list)
    .startWith(0)
    .startWith(-1,-2  )
    .subscribe { print($0) }
    .disposed(by: disposeBag)

//next(-1)
//next(-2)
//next(0)
//next(1)
//next(2)
//next(3)
//completed
```

## concat

- 두개의 옵저버블을 연결할 때 사용
- 타입 메소드와 인스턴스 메소드 둘다 구현되어 있음.
- 선행된 옵저버블이 종료되어야 두번째 옵저버블이 방출시작.

```swift
// # Type Method
// 순서대로 연결된 하나의 옵저버블을 리턴

let list = Observable.from([1,2])
let list2 = Observable.from([3,4])

Observable.concat([list, list2])
    .subscribe { print($0) }
    .disposed(by: disposeBag)

// # Instance Method
// 옵저버블이 Completed되는 순간 연결된 요소를 방출.
// error일 경우는 방출되지 않음.

list.concat(list2)
    .subscribe { print($0) }
    .disposed(by: disposeBag)
```

## merge

- 두개이상의 옵저버블을 병합 후 하나의 옵저버블을 생성
- 병환된  옵저버블에서 방출하는 순서대로 요소를 방출
- 병합된 모든 옵저버블이 종료된 후 Completed 전달.
- 병합된 옵저버블 중 에러가 발생하면 즉시 error 전달 후 종료.
- maxConcurrent: 병합가능한 최대 옵저버블 수 ( 초과할 경우 큐에 저장 )

```swift
let oddNumbers = BehaviorSubject(value: 1)
let evenNumbers = BehaviorSubject(value: 2)
let negativeNumbers = BehaviorSubject(value: -1)

let source = Observable.of(oddNumbers, evenNumbers, negativeNumbers)

source
    .merge()
//    .merge(maxConcurrent: 2)
    .subscribe { print($0) }
    .disposed(by: disposeBag)

oddNumbers.onNext(3)
evenNumbers.onNext(4)
//oddNumbers.onError(MyError.error)
evenNumbers.onNext(6)
oddNumbers.onNext(5)
//oddNumbers.onCompleted()
evenNumbers.onNext(8)
//evenNumbers.onCompleted()
negativeNumbers.onNext(-4)
```

## combineLatest

- 소스 옵저버블을 결합하고, 클로져( 익명함수 )를 실행 한 후 결과값을 방출
- 모든 옵저버블이 completed가 되는 순간 전체 completed 방출.
- Error가 발생할 경우 그 즉시 종료

```swift
let greetings = PublishSubject<String>()
let languages = PublishSubject<String>()

Observable
    .combineLatest(greetings, languages) { "\($0), \($1)" }
    .subscribe { print($0) }
    .disposed(by: disposeBag)

greetings.onNext("Hello")
greetings.onError(MyError.error)
languages.onNext("swift")
languages.onNext("rxSwift")
greetings.onCompleted()
greetings.onNext("hi")
languages.onNext("objc")
languages.onCompleted()

//next(Hello, swift)
//next(Hello, rxSwift)
//next(Hello, objc)
//completed
```

## zip

- `Indexed Sequencing`을 구현
- 소스 옵저버블을 결합하고, 클로져( 익명함수 )를 실행 한 후 결과값을 방출
- 같은 인덱스와 짝을 이뤄서 방출 ( Swift: 튜플로 전달 )

```swift
let a = BehaviorSubject<Int>(value: 0)
let b = BehaviorSubject<String>(value: "RxSwift")

Observable.zip(a, b) {  }// Indexed Sequencing.
    .subscribe { print($0) }
    .disposed(by: disposeBag)

a.onNext(2)
a.onNext(5)
b.onNext("iOS")

//next((0, "RxSwift"))
//next((2, "iOS"))
```

## withLatestFrom ↔ sample

- `triggerObservable.withLatestFrom(dataObservable)`
- trigger Observable을 이용해서 data Observable이 가장 최근에 방출한 값을 방출.
- 동일한 요소일경우라도 동일하게 방출

```swift
let a = BehaviorSubject<Int>(value: 0)
// UI기준으로 버튼.
let trigger = PublishSubject<Void>() // 트리거 옵저버블
trigger.withLatestFrom(a)
    .subscribe { print($0) }
    .disposed(by: disposeBag)

a.onNext(2)
trigger.onNext(())
a.onNext(2)
trigger.onNext(())
trigger.onNext(())
a.onCompleted()
trigger.onNext(())

//next(2)
//next(2)
//next(2)
//next(2)
```

## sample

- `dataObservable.sample(trigger)`
- 이전에 방출한 요소와 동일한 요소일 경우는 무시

```swift
let a = BehaviorSubject<Int>(value: 0)
let trigger = PublishSubject<Void>() // 트리거 옵저버블
a.sample(trigger)
    .subscribe { print($0) }
    .disposed(by: disposeBag)

a.onNext(2)
trigger.onNext(())
trigger.onNext(())
a.onCompleted()
trigger.onNext(())

//next(2)
//completed
```

## switchLatest

- 가장 최근에 방출된 옵저버블을 구독하고, 이 옵저버블이 전달하는 이벤트를 구독자에게 전달
- 구독하는 옵저버블에 completed를 방출하면 구독자에게 전달
- Error이벤트의 경우는 최근 구독하는 옵저버블에서 방출되면 그 즉시 구독자에게 전달.

```swift
//switchLatest -> // 서버가 여러개일때, 굳이 사용처를 찾으면, 파일 업로드.
let a = BehaviorSubject<Int>(value: 0)
let b = BehaviorSubject<String>(value: "RxSwift")
let c = BehaviorSubject<Int>(value: 1000)
let o = PublishSubject<BehaviorSubject<Int>>() // Inner Observable

o.switchLatest()
    .subscribe { print($0) }
    .disposed(by: disposeBag)

o.onNext(a)
a.onNext(1)
c.onNext(2000)
o.onNext(c)
a.onNext(2)
c.onNext(3000)

//next(0)
//next(1)
//next(2000)
//next(3000)
```