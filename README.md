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

## [Breakpoint](https://github.com/gaeng2y/LLDB-Debugger/blob/main/Breakpoint/Breakpoint.md)
## [Stepping](https://github.com/gaeng2y/LLDB-Debugger/blob/main/Stepping/Stepping.md)
## [Expression](https://github.com/gaeng2y/LLDB-Debugger/blob/main/Expression/Expression.md)
## [Image](https://github.com/gaeng2y/LLDB-Debugger/blob/main/Image/Image.md)
## [Alias](https://github.com/gaeng2y/LLDB-Debugger/blob/main/Alias/Alias.md)
