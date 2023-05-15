---
layout: post
title:  "GNU sed와 toybox sed의 UTF-8 문자 지원 여부"
date:   2023-05-14 23:00:00 +0900
---

우분투의 `GNU sed`는 UTF-8 문자를 지원하지만, 안드로이드의 `toybox sed`는 UTF-8 문자를 지원하지 않는 것 같다.

유니코드 문자 `a`, `α`, `가`, `😀`는 UTF-8로 인코딩하면 각각 아래와 같이 1, 2, 3, 4바이트로 표현된다. 
- `a` = `U+0061` = `61`
- `α` = `U+03B1` = `CE B1`
- `가` = `U+AC00` = `EA B0 80`
- `😀` = `U+1F600` = `F0 9F 98 80`

일단, 이들 문자를 단순히 출력하는 것은 양쪽 모두 가능하다.
- 단, Windows 10의 `conhost.exe`는 이모지 출력을 지원하지 않으므로 이모지까지 출력되는 것을 확인하려면 Windows Terminal을 이용해야 한다.
- 또한 안드로이드 셸의 경우 `adb shell` 명령으로 interactive shell에 접속했을 때에는 한글 등의 유니코드 문자 입력시 명령 프롬프트 창이 죽어버리는 문제가 있다. (Windows Terminal에서도 마찬가지)\
그래서 안드로이드 셸은 `adb shell` 바로 뒤에 실행할 명령을 기술하여 실행하는 non-interactive 모드로 확인했다. 

하지만, `sed 's/./& /g'` 명령을 이용하여 각 문자 뒤에 스페이스를 하나씩 추가하는 것은, 우분투의 `GNU sed`에서만 기대한 대로 실행되며 안드로이드의 `toybox sed`에서는 기대한 대로 실행되지 않는다.

우분투 20.04 LTS on WSL1 on Windows 10 에서
```console
$ echo 'aα가😀'
aα가😀
$ echo 'aα가😀' | xxd -g 1
00000000: 61 ce b1 ea b0 80 f0 9f 98 80 0a                 a..........
$
$ echo 'aα가😀' | sed 's/./& /g'
a α 가 😀
$ echo 'aα가😀' | sed 's/./& /g' | xxd -g 1
00000000: 61 20 ce b1 20 ea b0 80 20 f0 9f 98 80 20 0a     a .. ... .... .
$
```

안드로이드 10 `QQ3A.200805.001` on Pixel 3 XL 에서
```console
E:\platform-tools>adb shell "echo 'aα가😀'"
aα가😀

E:\platform-tools>adb shell "echo 'aα가😀' | xxd -g 1"
00000000: 61 ce b1 ea b0 80 f0 9f 98 80 0a                 a..........

E:\platform-tools>
E:\platform-tools>adb shell "echo 'aα가😀' | sed 's/./& /g'"
a � � � � � � � � �

E:\platform-tools>adb shell "echo 'aα가😀' | sed 's/./& /g' | xxd -g 1"
00000000: 61 20 ce 20 b1 20 ea 20 b0 20 80 20 f0 20 9f 20  a . . . . . . .
00000010: 98 20 80 20 0a                                   . . .

E:\platform-tools>
```

우분투의 `GNU sed`에서는 정규식의 메타 문자 `.`이 실제로 모든 **문자**에 매치되지만, 안드로이드의 `toybox sed`에서는 이것이 모든 **바이트**에 매치되는 것을 확인할 수 있다.

`toybox sed`가 아예 멀티바이트 문자를 다루지 못하는 것인지, 아니면 `toybox sed`가 멀티바이트 문자를 다룰 수는 있지만 안드로이드 셸에 그 기능을 활성화하는 설정이 되어 있지 않아서 이렇게 작동하는 것인지는 아직 모르겠다. 좀 더 조사를 해 봐야겠다.
