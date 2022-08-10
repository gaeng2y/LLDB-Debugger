## Expression

## po

아마 실무에서 LLDB Command 중 가장 많이 사용되는 Command가 `(lldb) po` 일텐데, 객체에 대한 다양한 정보를 Console에 출력하여 확인할 수 있기 때문이다. `(lldb) help po` 를 통해 확인할 수 있듯, `po` 는 `(lldb) expression -O --` 의 Shorthand이다. 여기서 -O option 은 object의 description을 출력하겠다는 뜻이다.

**po**가 ****출력하는 description은 `NSObject` 의 `debugDescription` 이다.

따라서, 아래와 같이 `debugDescription` 를 override 한다면,

```swift
override var debugDescription: String {
    return "이 객체의 debugDescription은 \(super.debugDescription) 입니다."
  }
```

복잡한 내용의 Description을 보기 쉽게 출력할 때 사용 가능하다.

> Swift 또는 Obj-C 중 특정 언어 표현이 출력되 않나요?
> LLDB에는 Swift 사용하거나 / 사용하지 않는 두가지 Debugging Context가 존재한다. Obj-C Code 내에서 Break한 경우에는 Swift를 사용하지 않는 Context이고, Swift Code 내에서 Break한 경우에는 반대일 것이다.
 
## Variable 사용하기

`(lldb) expression` Command는 Runtime에 여러 정보를 출력할 수 있을 뿐아니라 값을 변경 해줄 수도 있다. LLDB는 내부적으로 값이 출력될때마다 local variable을 *$R~*의 형태로 만들어 저장한다. 이 값들은 해당 break context가 벗어나도 사용 가능한 값들이고, 심지어 수정해서 사용할 수도 있다. 

아래의 예시를 보며 확인해 보자!

- `(lldb) expression self.view`

self.view의 정보를 출력하는데, 출력된 정보에는 $R0 이라는 이름의 변수에 self.view가 저장된 것으로 보인다.

- `(lldb) expression $R0.backgroundColor = UIColor.blue`

위에서 나온 self.view가 저장된 $R0의 속성인 배경 색상을 변경한다.

- `(lldb) c`

마저 진행한다.

## Variable 선언하기

또, 아래 예시와 같이 `(lldb) expression` Command를 이용해서 변수를 직접 선언해서 사용할 수도 있다. 단, 사용하고자 하는 변수명 앞에 `$` 문자를 붙여줘야 한다.

```swift
(lldb) expr let $someNumber = 10
(lldb) expr var $someString = "some string"
```

> Multi-line Expression 입력하기
`(lldb) expression` 만 입력하면 가능
> 

### —ignore-breakpoints option 활용하기

`—ignore-breakpoints`는 유용한 option 중 하나로, expr 실행 중 만나는 br을 ignore할 지 여부를 선택할 수 있다. (default 값은 -ignore-breakpoint true)
