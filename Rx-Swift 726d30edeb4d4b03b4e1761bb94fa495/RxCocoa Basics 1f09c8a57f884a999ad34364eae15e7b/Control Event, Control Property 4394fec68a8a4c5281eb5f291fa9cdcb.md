# Control Event, Control Property

## Control Property

- ObservableType 과 ObserverType도 같이 상속

```swift
public protocol ControlPropertyType: ObservableType, ObserverType {
    ///...
}

public struct ControlProperty<PropertyType>: ControlPropertyType {
    ///...
}
```

- UI 바인딩에 사용하므로 에러이벤트를 전달받지도 하지도 않음
- Completed 는 Control이 제거되지 직전에 전달.

![Control%20Event,%20Control%20Property%204394fec68a8a4c5281eb5f291fa9cdcb/_2020-01-21__11.23.43.png](Control%20Event,%20Control%20Property%204394fec68a8a4c5281eb5f291fa9cdcb/_2020-01-21__11.23.43.png)

- Control Property는 시퀀스를 공유
- replay로 버퍼에 하나를 저장하고 있어서 구독함과 동시에 저장되어있는 값을 방출.

![Control%20Event,%20Control%20Property%204394fec68a8a4c5281eb5f291fa9cdcb/_2020-01-21__11.25.43.png](Control%20Event,%20Control%20Property%204394fec68a8a4c5281eb5f291fa9cdcb/_2020-01-21__11.25.43.png)

## Control Event

![Control%20Event,%20Control%20Property%204394fec68a8a4c5281eb5f291fa9cdcb/_2020-01-26__2.30.10.png](Control%20Event,%20Control%20Property%204394fec68a8a4c5281eb5f291fa9cdcb/_2020-01-26__2.30.10.png)

- ObservableType 만 상속

```swift
public protocol ControlPropertyType: ObservableType, ObserverType {
    ///...
}

public struct ControlProperty<PropertyType>: ControlPropertyType {
    ///...
}
```

![Control%20Event,%20Control%20Property%204394fec68a8a4c5281eb5f291fa9cdcb/_2020-01-26__2.32.12.png](Control%20Event,%20Control%20Property%204394fec68a8a4c5281eb5f291fa9cdcb/_2020-01-26__2.32.12.png)

- Error 이벤트는 전달하지 않음.
- Completed 이벤트는 컨트롤이 해지되기 직전에 전달