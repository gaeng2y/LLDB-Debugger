# LLDB-Debugger
iOS 개발을 잘하기 위해, 편하게 버그를 추적하기 위해 LLDB를 이용한 디버깅 방법 공부한 내용

- LLDB(Low-Level Debugger)는 **Apple**에서 진행한 Compiler에 필요한 Toolchain 개발 프로젝트다.
- 각 컴포넌트들의 재사용성을 중시해서, 모듈화가 잘 되어있다는 특징이 있다.
- 이렇게 모듈화 되어있는 컴포넌트들을 이용해 진행된 주요 서브 프로젝트들로는 LLVM Core, Clang, libc++, **LLDB** 등이 있다.
- LLDB는 LLVM의 Debugger Component를 개발하는 서브 프로젝트이다.
- LLVM 프로젝트를 통해 개발된 Clang Expression Parser, LLVM Disassembler 등 Low-Level 컨트롤이 가능한 모듈들로 이루어져 있어, 기계어에 가까운 영역까지 디버깅이 가능하다는 장점이 있다.

## LLDB 명령어 문법

`(lldb) command [subcommand] -option "this is argument`

Command, Subcommand, Option, Argument들로 이루어져 있고, 띄어쓰기로 구분한다.

- **Command와 Subcommand**는 LLDB 내 Object의 이름이다.(etc. *breakpoint*, *watchpoint*, *set*, *list* … ) 이들은 모두 계층화되어있어, Command에 따라 사용가능한 Subcommand 종류가 다르다.
- **Option**의 경우, Command 뒤 어느 곳에든 위치 가능하며, -(hyphen)으로 시작한다.
- **Argument**에 공백이 포함되는 경우도 있기 때문에, “”로 묶어줄 수 있다.

ex)

`(lldb) breakpoint set --file test.c --line 12`

breakpoint (Command)와 set (Subcommand)을 이용하며

—file option을 통해 test.c 파일 내

—line option을 통해 12번째 라인에

중단점을 set 해준다.

## Help & Apropos

LLDB에는 수많은 명령어와 해당하는 Subcommand, Option들이 존재한다.

- Help: 해당 문법으로 사용가능한 Subcommand, Option 리스트나 사용법을 보여주는 유용한 명령어

```bash
# LLDB에서 제공하는 Command가 궁금하다면,
(lldb) help

# 특정 Command의 Subcommand나, Option이 궁금하다면,
(lldb) help breakpoint
(lldb) help breakpoint set
```

- Apropos: 원하는 기능의 명령어가 있는지 관련 키워드를 통해 알아볼 수 있는 명령어

```bash
# referent count를 체크할 수 있는 명령어가 있을까? 궁금하다면,
(lldb) apropos "reference count"
# 결과
# The following commands may relate to 'reference count':
#     refcount -- Inspect the reference count data for a Swift object
```

## Breakpoint

- LLDB의 가장 중요한 Action 중 하나인 Stepping에 대해 알아보자
- Stepping은 프로세스를 단계별로 진행하면서 상태 변화를 관찰해 볼 수 있는 유용한 기능이다.

```bash
(lldb) breakpoint set [option] "arguments"
# 줄여서
(lldb) br s [option] "arguments"
```

> 대부분의 명령어와 옵션들은 command 맨 앞 1~2개 앞파벳으로 줄여서 사용할 수 있다.
> 

### Function Name

특정 이름을 가진 모든 함수에 -name (-n) option을 이용해 break를 걸 수 있다.

```bash
# 함수 이름 이용해 break
(lldb) breakpoint set --name viewDidLoad
(lldb) b -n viewDidLoad
```

또한, -func-regx (-r) option을 이용해 **정규표현식**을 활용할 수 있다.

```bash
(lldb) breakpoint set --func-regex '^hello'
(lldb) br s -r '^hello'
# 'breakpoint set --func-regex'는 줄여서 'rb'로 사용 가능
(lldb) rb '^hello'
```

### File

파일의 이름과 line 번호를 이용해 break를 걸기 위해서는, --file (-f), --line (-l) option을 이용할 수 있다.

```bash
# 특정 파일의 20번째 line에서 break
(lldb) breakpoint set --file ViewController.swift --line 20
(lldb) br s -f ViewController.swift -l 20
```

<aside>
💡 **Xcode를 이용해 Execute Point 이동하기**
bp의 우측에 녹색 햄버거 버튼을 위 아래로 드래그하면 다음 실행 지점을 변경할 수 있다.

</aside>

### Condition

-condition (-c) option을 이용하면, bp에 원하는 조건을 걸 수도 있다. -c option 뒤의 expression이 true인 경우에만 bp에서 멈춘다.

```bash
# viewWillAppear 호출 시, animated가 true인 경우에만 break
(lldb) breakpoint set --name "viewWillAppear" --condition animated
(lldb) br s -n "viewWillAppear" -c animated
```

> **Quiz**
(lldb) br s -f ViewController.swift -l 17 -c ‘helloWorld == “abc”’는 어떤 의미인가?
ViewController.swift 파일에서 17번째 라인의 helloWorld == “abc”가 true일 때 bp를 set
> 

### Command 실행 & AutoContinue

—command (-C) option을 이용하면 break 시 원하는 lldb command를 실행할 수 있다.

```bash
(lldb) breakpoint set -n "viewDidLoad" --command "po $arg1" -G1
(lldb) br s -n "viewDidLoad" -C "po $arg1" -G1
```

—auto-continue (-G) option의 기능은 auto continue로, command 실행 후 break에 걸린채로 있지 않고 프로그램을 자동 진행하게 해준다.

### ? b (_regexp-break) Command

(lldb) regexp-break는 간단하게 Breakpoint 생성을 할 수 있도록 도와주는 Shorthand Command 입니다.

(lldb) b로 줄여서 사용할 수 있다.

- 사용 방법

```bash
# 특정 이름을 가진 function에서 break
(lldb) b viewDidLoad
# 현재 파일 20번째 line에서 break
(lldb) b 20
# 특정 파일 20번째 line에서 break
(lldb) b ViewController.swift:12
# 현재 파일 내 특정 text를 포함한 line에서 break
(lldb) b /stop here/
# 특정 주소값에서 break
(lldb) b 0x1234000
```

## Breakpoint 리스트 확인하기

(lldb) breakpoint list command를 통해 현재 프로그램에 생성되어있는 Breakpoint의 목록을 확인할 수 있다. 

또한, 이 목록 정보에는 Breakpoint의 id와 이름, hit-count 정보, enable 정보, source 상의 위치, 주소값 등등 유용한 정보가 포함되어 있다.

### hit-count 란?

<aside>
💡 프로그램 실행 중 활성 상태인 Breakpoint 지점이 실행되면, Debugger는 hit count를 1씩 늘려가며 기록한다.
하지만 Breakpoint가 걸려있다 하더라도, disable 상태이면 count되지 않는다.

</aside>

Breakpoint id를 통해 원하는 내용만 출력하거나, —brief (-b) option을 통해 간단한 내용을 확인해 볼 수도 있다.

```bash
# bp 목록 전체 출력
(lldb) breakpoint list
(lldb) br list
# bp 목록 간단하게 출력
(lldb) br list -b
# 특정 id를 가진 bp의 정보만 출력
(lldb) br list 1
```

## Breakpoint 삭제 or 비활성화 시키기

delete, disable Subcommand를 이용해 Breakpoint를 삭제하거나, 비활성화 할 수 있다.

```bash
# bp 전체 삭제
(lldb) breakpoint delete
(lldb) br de
# 특정 bp 삭제
(lldb) br de 1
# bp 전체 비활성화
(lldb) breakpoint disable
(lldb) br di
# 특정 bp 비활성화
(lldb) br di 1.1
```

## Stepping

LLDB의 가장 중요한 Action 중 하나인 **Stepping**에 대해 알아보자. Stepping은 프로세스를 단계별로 진행하면서 상태 변화를 관찰해 볼 수 있는 유용한 기능이다.

### Stepping Over
---

`(lldb) next` Command를 이용하면, 현재 Break 걸려있는 지점에서 바로 다음 Statement로 **Step Over** 할 수 있다. 줄여서 `(lldb) n`

### Stepping In
---

**Stepping in**은 다음 Statement가 Function Call 인 경우 Debugger를 해당 함수 내부에 위치한 시작 기점으로 이동하게 해준다. `(lldb) step` Command를 이용해 실행할 수 있다. 줄여서 `(lldb) s` 으로 사용 가능하다.


> 💡 Steeping In 동작이 이상해요!?

> Stepping In은 주로 바로 위에서 소개된 Stepping Over와 비슷하게 동작하는 경우가 많다.
LLDB의 경우에는 기본적으로 Debug Symbol이 없는 함수에 대해서는 Stepping In을 무시하고, 프로그램을 진행하기 때문이다.

### Stepping Out
---

현재 진행 중인 function이 return 될 때까지 프로그램을 진행한 후 프로그램 Break 걸어주는 Stepping Action을 **Stepping Out**이라고 한다. `(lldb) finish` Command로 실행해 볼 수 있다. Stack Memory 관점에서 Stepping Out은 Stack Frame을 Pop하는 것과 동일하다.

`(lldb) next` 명령을 통해 한 줄씩 진행하고, `(lldb) step` 명령을 통해 UIKit의 ViewDidLoad 함수 내부 Debug Symbol로 이동한 후, `(lldb) finish` 명령을 통해 함수 return 이후 지점까지 진행하는 모습을 볼 수 있다. 

더해서, 왼쪽 Debug Navigator를 잘 살펴보면 Stepping-In과 Stepping-Out 시 StackFrame이 어떻게 달라지는 지 확인할 수 있다.

> `(lldb) run` 명령은 말 그대로 현재 프로그램을 중단하고, 새로운 Build/Run을 진행한다.
> 또한, `(lldb) continue` 명령은 다음 Breakpoint가 나타날 때까지 프로그램을 진행한다.

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