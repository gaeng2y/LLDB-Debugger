# Alias

사실 개발 할 때 가장 싫은건 반복에서 같은 내용을 일일히 타이핑 해야할 때이다. 

🤢반복해서 사용하고 싶은 명령어가 너무 복잡하거나 긴 경우에는 🤯, *(특히 명령에 복잡한 정규식 있을 때)*

`(lldb) command alias 별명 "줄이고 싶은 Command"` 명령어를 이용하면 간단한 별명으로 줄여서 이용할 수 있습니다. 일반적으로 프로그래밍 언어에서 쓰이는 `typedef`, `typealias`와 비슷한 느낌으로 사용하시면 됩니다! 🤓

예를 들면, Swift Code 내에서 Objective-C expression을 출력하기 위해 `(lldb) expression -l objc -O --` Command를 사용하는 경우가 종종 있습니다.

`(lldb) command alias pojc expression -l objc -O --`

위 명령어를 실행해준 이후에는, `(lldb) pojc "[Objective-C Expression]"` 형태로 명령어를 사용할 수 있습니다! 🥳

### Alias를 등록해도 자꾸 사라지면..?

Command Alias로 등록해 두어도 별칭은 해당 빌드 시점이 지나면, LLDB Session이 끝남과 동시에 사라진다.

Xcode를 실행 시 LLDB 초기화를 위해 사용되는 ~/.lldbinit 파일에 원하는 명령어 Alias를 추가해두면, 별칭을 매번 정해주지 않고 계속 사용 가능하다.
