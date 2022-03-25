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