# Binding

![Binding%20631cba3f6b4b4a4eab02a1ebea2c41f4/_2020-01-21__11.01.39.png](Binding%20631cba3f6b4b4a4eab02a1ebea2c41f4/_2020-01-21__11.01.39.png)

- 데이터 생산자 ( Observable ) 와 데이터 소비자가 있음 ( UI Component)
- 데이터 생산자
    - Observable을 채용한 모든 요소
- 데이터 소비자 ( UI Component )
- 데이터 소비자가 데이터 생산자에게 데이터를 전달하는 경우는 없음 ( 단방향 )

### Binder는 Observer로 이벤트를 전달받을 수는 있지만 구독을 추가할 수는 없음

![Binding%20631cba3f6b4b4a4eab02a1ebea2c41f4/_2020-01-21__11.07.19.png](Binding%20631cba3f6b4b4a4eab02a1ebea2c41f4/_2020-01-21__11.07.19.png)

### Error 이벤트를 받지 않음.

### Bind는 Binder가 MainThread에서 동작하는  것을 보장함.

![Binding%20631cba3f6b4b4a4eab02a1ebea2c41f4/_2020-01-21__11.22.57.png](Binding%20631cba3f6b4b4a4eab02a1ebea2c41f4/_2020-01-21__11.22.57.png)