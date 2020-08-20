# RxCocoa Traits

- UI에 특화된 Observable이며, Main Thread에서 실행
- UI에서 데이터 생산자에 속함
- Error를 방출하지 않기 때문에 UI가 정상적으로 동작 하는 것을 보장

    ![RxCocoa%20Traits%209bebb182fb504d73a30e077115b8988d/_2020-01-21__7.34.08.png](RxCocoa%20Traits%209bebb182fb504d73a30e077115b8988d/_2020-01-21__7.34.08.png)

- Observable이 구독될때는 항상 새로운 시퀀스로 실행되지만, Traits의 경우는 동일한 시퀀스를 공유함

    ![RxCocoa%20Traits%209bebb182fb504d73a30e077115b8988d/_2020-01-21__7.35.51.png](RxCocoa%20Traits%209bebb182fb504d73a30e077115b8988d/_2020-01-21__7.35.51.png)

- RxCocoa를 사용할 때, Traits를 반드시 사용할 필요는 없음. ( Subscribe로 구현은 가능 )
- But 적극적으로 사용하는게 간결한 코드를 작성하는데 도움이 됌.

```swift
// # Swift Code
class BindingCocoaTouchViewController: UIViewController {
     
   @IBOutlet weak var valueLabel: UILabel!
   
   @IBOutlet weak var valueField: UITextField!
   
   override func viewDidLoad() {
      super.viewDidLoad()
      
      valueLabel.text = ""
      valueField.delegate = self
      valueField.becomeFirstResponder()
   }
   
   override func viewWillDisappear(_ animated: Bool) {
      super.viewWillDisappear(animated)
      
      valueField.resignFirstResponder()
   }
}

extension BindingCocoaTouchViewController: UITextFieldDelegate {
   func textField(_ textField: UITextField, shouldChangeCharactersIn range: NSRange, replacementString string: String) -> Bool {
      
      guard let currentText = textField.text else {
         return true
      }
      
      let finalText = (currentText as NSString).replacingCharacters(in: range, with: string)
      valueLabel.text = finalText
      
      return true
   }
}
```

```swift
// # RxSwift

// #1
valueField.rx.text
            .subscribe(onNext: { [weak self] str in
                // Swift MainThread
                DispatchQueue.main.async {
                    self?.valueLabel.text = str
                }
            })
            .disposed(by: disposeBag)

// #2        
valueField.rx.text
            .observeOn(MainScheduler.instance)
            .subscribe(onNext: { [weak self] str in
                self?.valueLabel.text = str
            })
            .disposed(by: disposeBag)

// #3        
valueField.rx.text
            .bind(to: valueLabel.rx.text)
            .disposed(by: disposeBag)
```