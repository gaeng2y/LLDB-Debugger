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

