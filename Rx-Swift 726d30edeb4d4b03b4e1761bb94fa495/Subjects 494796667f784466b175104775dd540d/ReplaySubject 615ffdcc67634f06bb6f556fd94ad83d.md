# ReplaySubject

- 지정된 버퍼크기만큼 이벤트를 저장.
- Subscribe와 동시에 버퍼에 저장된 이벤트를 방출.
- Create Method를 통해서 생성.
- 지정된 수만큼 사용하기 때문에 메모리 크기에 주의.

```swift
let rs = ReplaySubject<Int>.create(bufferSize: 3) 

(1...10).forEach { rs.onNext($0) }

rs.subscribe { print("Observer 1 >>", $0 }
  .disposed(by: disposeBag)

rs.onNext(11)

rs.subscribe { print("Observer 2 >>", $0 }
  .disposed(by: disposeBag)

```