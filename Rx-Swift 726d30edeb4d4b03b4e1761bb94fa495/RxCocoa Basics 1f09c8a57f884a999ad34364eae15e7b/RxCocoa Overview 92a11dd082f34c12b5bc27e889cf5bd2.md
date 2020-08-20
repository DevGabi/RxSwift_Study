# RxCocoa Overview

- `Cocoa Framework` + ReactiveX
- pod '`RxCocoa`'를 통해서 개별로 추가.
- `extension Reactive` 를 통해서 확장가능
- `ReactiveCompatible` → 기존속성에 `rx` 속성을 추가. ( `namespace`와 동일한 개념 )
- `ControlEvent` - `RxCocoa`가 제공하는 `Traits`
- `Traits`는 UI처리에 특화된 기능(`Observable`)을 제공함 ( `driver`, `signal`, `ControlEvent` 등 )
- 대부분의 속성을 기존에 가지고있는 프로퍼티 이름과 동일함 ( `label.text` → `label.rx.text` )
- 필요한 매개변수(parameter)가 없을 경우 사용자가 직접 커스텀 가능

```swift
// Reactive.Swift

public struct Reactive<Base> {
    /// Base object to extend.
    public let base: Base

    /// Creates extensions with base object.
    ///
    /// - parameter base: Base object.
    public init(_ base: Base) {
        self.base = base
    }
}

/// A type that has reactive extensions.
public protocol ReactiveCompatible {
    /// Extended type
    associatedtype ReactiveBase

    @available(*, deprecated, message: "Use `ReactiveBase` instead.")
    typealias CompatibleType = ReactiveBase

    /// Reactive extensions.
    static var rx: Reactive<ReactiveBase>.Type { get set }

    /// Reactive extensions.
    var rx: Reactive<ReactiveBase> { get set }
}

extension ReactiveCompatible {
    /// Reactive extensions.
    public static var rx: Reactive<Self>.Type {
        get {
            return Reactive<Self>.self
        }
        set {
            // this enables using Reactive to "mutate" base type
        }
    }

    /// Reactive extensions.
    public var rx: Reactive<Self> {
        get {
            return Reactive(self)
        }
        set {
            // this enables using Reactive to "mutate" base object
        }
    }
}

import class Foundation.NSObject

/// Extend NSObject with `rx` proxy.
extension NSObject: ReactiveCompatible { }
```