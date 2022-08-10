# Image

Image Command에 대해 알아보자

`(lldb) image`

Module 내에서 나타나는 Symbol에 대한 자세한 정보를 알아낼 수 있는 Command 이다.

image Command를 이용하면, Framework에 대한 Private한 정보들(private Class, private Method 등)을 Header File에 공개되어 있는 것 이상으로 알아볼 수 있다. Private API들에 접근 가능하다면, 객체에 대한 숨겨진 Description을 확인하거나, 더 세부적인 속성을 확인할 수 있을 거다. 

### Module이란?

Process에 Load되어 실행되는 Code를 의미한다. Swift에서 Module은 메인 실행 파일 뿐아니라, Framework나 Library, Plugin 등도 포함하는 개념이다. 예를 들면, 우리가 만든 Application 프로젝트를 포함해서, UIKit, AppKit 등의 Library 까지도 Module로 볼 수 있다.

### Symbol이란?

Symbol이란 Method, Variable, Class 등 말 그대로 기계가 아닌 사람의 눈으로 알아볼 수 있는 소스 코드를 이루는 작은 단위의 Symbol을 뜻한다. Symbol Table은 컴파일된 바이너리를 그에 맞는 메소드, 변수, 클래스 등으로 상호 매핑 해주는 역할을 한다. 또한 바이너리가 심볼로 번역되는 것을 Symbolicate라고 한다.

## Image List

`(lldb) image list` 는 현재 Process에 Load 되어있는 모든 Module 들의 정보를 출력한다. 새 Project를 만들고 실행 시킨 후에 명령어를 실행해보면 프로젝트 생성 이후 구현한 내용이 없는데도, 무려 350개의 모듈이 로드되어있음을 확인할 수 있다.

간단하게 요약해보면, 현재 로드되어있는 모듈들의 UUID, 실제 메모리 상 로드되어있는 주소값, 그리고 디스크 내에서 바이너리가 존재하는 path 정보를 알 수 있다.

## Image Dump

`(lldb) image dump` 커맨드를 이용하면 모듈의 세부적인 정보를 dump(기억 장치의 내용을 기록하는 것) 해볼 수 있다.

`(lldb) image dump symtab UIKitCore -s address` 명령어를 실행해보면, UIKitCore 라이브러리에 퍼함되어 있는 엄청나게 많은 Symbol table이 출력된다. 또한, -s address 옵션을 이용해, 주소값으로 정렬된 목록이 출력된 것을 볼 수 있다.(-s sort)

목록들 중에서 _가 Prefix로 붙어있는 Symbol들은 공식 문서에는 나와있지 않은 Private API입니다.

## Image Lookup

image dump 명령을 통해 나온 정보들을 필터링해서 볼 수 있는 명령어가 바로 `(lldb) image lookup` 이다.

```swift
# 함수 이름 (--function)
(lldb) image lookup -F "functionName"
# 주소값 (--address)
(lldb) image lookup -a "0x00address"
# 파일 이름 (--filename)
(lldb) image lookup -f "FileName.swift"
# 라인 번호 (--line)
(lldb) image lookup -f "FileName.swift" -l 15
# 정규식 이용 (--regex)
(lldb) image lookup -rn "regexExpression"
```

image lookup을 이용하면 크래시 로그 살펴볼 때도 굉장히 유용하다.

터미널에서도 LLDB Console이 가능하다.

- `(lldb) target create "dSYM 경로"` 를 통해 lldb를 dSYM 파일을 attach 할 수 있다.
