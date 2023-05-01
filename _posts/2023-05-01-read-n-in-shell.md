---
layout: post
title:  "셸에서 `read -n` 명령으로 일정한 글자수마다 줄바꿈하여 출력하기"
date:   2023-05-01 20:55:00 +0900
---

지난 금요일에 다른 것 찾다가 우연히 발견한 스택오버플로우 질문.

- [Breaking down a long string and then converting hex to decimal](https://stackoverflow.com/questions/67007960/breaking-down-a-long-string-and-then-converting-hex-to-decimal) (Stack Overflow)

지난번에 작성한 [셸에서 랜덤한 N바이트를 16진수로 출력하기](https://nooriro.github.io/230414/random-n-bytes-as-hex-string-in-shell)의 뒷부분에서, ‘셸에서 임의 길이의 문자열을 특정 길이마다 줄바꿈하여 출력하려면 어떻게 해야 하는가?’와 비슷한 질문을 던졌었는데, 그것과 관련된 내용의 질문이라서 눈에 확 들어왔다.

위 스택오버플로우 질문에 달린 몇 가지 답변들 중 가장 놀라웠던 것은, [`read` 명령의 `-n` 옵션으로 읽을 바이트 수를 지정할 수 있다](https://stackoverflow.com/questions/67007960/breaking-down-a-long-string-and-then-converting-hex-to-decimal/67009278#67009278)는 것이었다. 이게 된다면, 굳이 어려운(?) 명령을 동원하지 않고도 지난번에 제기했던 문제를 쉽게(?) 해결할 수 있을 것이다.

하지만 그 얘기는 잠시 후에 하기로 하고, 우선 지난 글에서 언급했던 내용을 다시 끄집어내는 것으로 이야기를 시작해보고자 한다. 혹시, 결론만 필요하다면 이 글 맨 끝에 있는 ‘결론’과 ‘일반화’만 보면 된다.

## `xxd -p` 명령의 결과를 줄바꿈 없이 출력하기

`xxd -l 100 -p /dev/urandom` 명령을 실행하면 랜덤한 100바이트의 데이터가 plain hexdump style로 출력된다. 이때 **1바이트**는 **십육진수 두 자리**로 표현되므로, 결국 랜덤한 십육진수 문자 200개가 한번에 출력된다.\
(지난번 글에서는 200바이트, 그러니까 십육진수 400자리를 한번에 출력했는데, 여기서도 그렇게 하면 이 글이 너무 길어지기 때문에 개수를 절반으로 줄여 보았다.)

Ubuntu 20.04 on WSL1에서 직접 실행해본 결과는 아래와 같다. (이하 모두 특별한 언급이 없다면 Ubuntu 20.04 on WSL1에서 실행해본 결과이다.)

```console
$ xxd -l 100 -p /dev/urandom
0dbd28965f5d3b690442f05716e35b83dbf37f585f165be7191118b41ed9
7c2a4a28b0274299864d32b2c061b9b4bd204c2b011f97b26b63b531bcae
7001bebd5e2ea4e1f78fb667868ecb85f4bde30a5ad636fd69c4cd0c6660
147e7b7f7fe74b7bccf1
$
```

그런데 위 출력 결과를 보면, 별도의 옵션을 지정하지 않았는데도 일정한 길이(60개 글자)마다 줄바꿈되어 출력된 것을 확인할 수 있다. 이는 `-c` 옵션(이것의 역할은 [지난 글](https://nooriro.github.io/230414/random-n-bytes-as-hex-string-in-shell)을 참고)의 **디폴트 값**(기본 값)이 있기 때문이다. 출력 스타일에 따라서 이 디폴트 값은 다른데, 여기서처럼 `-p` 옵션을 주어 plain hexdump style로 출력할 때에는 `-c` 옵션의 디폴트 값이 **30**이다. `xxd --help` 명령으로 이를 확인할 수 있다.

```console
$ xxd --help
Usage:
       xxd [options] [infile [outfile]]
    or
       xxd -r [-s [-]offset] [-c cols] [-ps] [infile [outfile]]
Options:
                     ⋮
-c cols     format <cols> octets per line. Default 16 (-i: 12, -ps: 30).
                     ⋮
$
```

그러므로 `-c` 옵션의 값을 **`-l` 옵션으로 지정한 값**(여기서는 100)**보다 크거나 같게** 지정하면 맨 끝에 딱 한 번만 줄바꿈되어 출력된다.

```console
$ xxd -l 100 -p -c 100 /dev/urandom
c7fe47823b5d859523d4d6b6ad770c8a307fd2b270430e4f1ce2394988e16a35b49d4a1a2aa3ca6c3ca33652c17ee041aced8e35be59723da24923e1b1e6a96b43765c738db23a28bad50e2f12dcd53213b18914c6f28b2328c754f05536c3a642479ac3
$ xxd -l 100 -p -c 1000 /dev/urandom
c6baacf17ff46bece47e9dc11a1c827548adbf10e4d312d4a14144953952ee64746bea49510030a97bd3e8c4f95e8b855b7c220b50e77b904cb2a1cf3bbbf8fc0153fe9a59558ac2ffd1731c54f786973e1c26ce49c5ebf3955a59179a89120aaff49988
$
```

한편, 다른 방법도 있다. 아래와 같이 xxd 명령의 출력을 파이프(`|`)로 `tr -d '\n'` 명령의 입력으로 보내는 방법이다.
```
xxd -l 100 -p /dev/urandom | tr -d '\n'
```
이렇게 하면 xxd의 출력에서 모든 개행 문자(`\n`, ‘줄바꿈 문자’ 혹은 ‘새줄 문자’라고도 함. 영어로는 newline character)가 제거되어 출력된다.
```console
$ xxd -l 100 -p /dev/urandom
7968638316f547d2ad0964b0a505fd53d6535530691576660433f65ddfe9
8b7b2f8c4ad8fc85d9c49b4450baf1b7c2c079500f0ffe27306a40907633
c1ba5a0ce6b92a59f46be62fed06b0782412e7ca9af59b52499f9c5162ec
008fd6d2f24b98bfda7d
$ xxd -l 100 -p /dev/urandom | tr -d '\n'
251bdd5c7eee3b1447155dd241b605321ca6bb93875541665b5694a4a2e80fb5d8af8b3e61f61ac26058b0ac47ac555af2c0b6deba3ffe7d8c9f327411e73b57c04df361a699e1630814282066ddad380d87fc8e13a58cb21ba4dd2e0464cc75b44c5892$
```
xxd에 `-c` 옵션을 이용했을 때와 다른 점은, 위 명령은 xxd의 출력에서 모든 개행 문자 `\n`를 제거하기 때문에 맨 끝에 있는 개행문자도 제거된다는 것이다. 둘의 차이를 확인해 보자. 출력이 길면 확인하기 불편하므로 이번에는 십육진수 문자를 각각 40개씩만 출력해 보자.

```console
$ xxd -l 20 -p -c 20 /dev/urandom
f53290f0bf669a50ca0727d4d7693c79bc55a993
$ xxd -l 20 -p /dev/urandom | tr -d '\n'
3312e91e09e114dd6dbff9ee143c553f26c1b43e$
```

첫 번째는 xxd가 맨 끝에 덧붙인 개행문자로 인해 줄바꿈이 되어 **그 다음 줄에** 셸 프롬프트(`$`)가 출력되었다. 반면에 두 번째는 xxd가 출력한 모든 개행 문자가 `tr -d '\n'` 명령에 의해 제거되어 십육진수 40자리가 출력되고 **바로 이어서 같은 줄에** 셸 프롬프트(`$`)가 출력되었다.

참고로, 만일 `xxd -l 20 -p -c 20 /dev/urandom | tr -d '\n'`와 같이 xxd의 `-c` 옵션과 `tr -d '\n'` 명령을 둘 다 이용하면, 그 경우에도 출력은 위의 두 번째와 동일하다. 이 경우, xxd의 출력에는 개행 문자가 맨 끝에 1개만 있으므로 tr 명령이 하는 일은 그 맨 끝에 붙은 개행 문자를 제거하는 일 뿐이다. 이렇게 마지막이 개행 문자로 끝난다는 것이 확실하다면, 사실 마지막 1개 문자를 ‘무조건’ 제거해도 결과는 동일하다. 여러 가지 방법으로 할 수 있지만, 이를테면 `xxd -l 20 -p -c 20 /dev/urandom | head -c -1`을 실행해도 위의 두 번째와 동일한 결과를 얻는다. 

## 한 줄 출력의 끝에 개행 문자를 붙여야 할까?

출력이 여러 줄일 경우 마지막 줄의 끝에도 개행 문자가 들어가는 것이 확실히 자연스러워 보인다. 하지만 위와 같이 출력이 1줄 뿐일 경우에도 끝에 개행문자를 붙여서 줄바꿈되도록 해야 할까? 이 질문은 생각해 볼 만한 가치가 있는 질문이다. 내 생각은 다음과 같다.

- 일단, 위와 같이 터미널에서 직접 명령을 실행하여 결과를 그대로 화면에 출력하는 것이 목적이라면, 출력이 1줄 뿐이더라도 마지막에 줄바꿈이 들어가는 것이 적합하다. 그렇지 않으면 위와 같이 셸 프롬프트(`$`)가 다음 줄로 넘어가지 않고 출력된 내용 바로 뒤에 붙어서 출력되기 때문이다. 이렇게 되면 이전 명령의 출력과 프롬프트, 유저가 입력하는 명령이 전부 같은 줄에 있게 되므로 가독성이 상당히 나빠진다.
- 하지만 출력 결과를 파이프(`|`)로 다른 명령에 넘겨서 처리하고자 할 경우에는 파이프 오른쪽의 명령이 데이터를 어떻게 취급하는지를 따져보아야 한다. 만일 파이프 오른쪽의 명령이 단순히 줄 단위로 데이터를 다룬다면, 이 경우에도 끝에 개행 문자가 붙는 것이 적합하다. 하지만 파이프 오른쪽의 명령이 단순히 줄 단위로만 데이터를 다루지 않는다면, 끝에 개행 문자를 붙이는 것과 붙이지 않는 것 중에서 무엇이 더 적합할지 생각을 좀 해 봐야 한다.
- 단, 위 두 명령이 출력한 결과를 `$(...)` 문법을 이용하여 셸 변수에 저장하려고 한다면, 두 가지 방법 중 아무거나 써도 된다. 표준 출력을 셸 변수에 담는 과정에서 맨 끝에 붙은 개행 문자(trailing newline)는 자동으로 제거되기 때문이다.

이 글 전체를 통해, 한 줄 출력의 끝에 개행 문자가 붙은 것과 붙지 않은 것의 차이가 이후 처리에 무슨 영향을 주는지 탐구해 볼 생각이다.

그런데, 매번 위 두 명령(`xxd -l 100 -p -c 100 /dev/urandom`과 `xxd -l 100 -p /dev/urandom | tr -d '\n'`)을 직접 실행해서 그 데이터를 처리하는 코드를 보이면, 우선 코드에서 강조할 부분이 한눈에 안 들어와서 집중이 잘 되지 않는다. 또한 둘의 차이(개행 문자의 유무)가 직관적으로 보이지 않아서 계속 둘의 차이를 환기해야 하는데, 그걸 매번 말로 이야기하면 설명이 너무 장황해진다. 그래서 일단 아래 명령을 실행해서 설명에 필요한 데이터를 파일로 저장해 두고 그걸 이용해서 설명하도록 하겠다.

```console
$ xxd -l 100 -p -c 100 /dev/urandom > w_newline
$ cat w_newline | tr -d '\n' > wo_newline
```

위 명령들을 실행하면 `w_newline`과 `wo_newline`이라는 이름의 파일 두 개가 생성된다.\
두 파일 모두 동일한 데이터(랜덤한 200자리의 십육진수 문자)를 담고 있지만 **`w_newline`은 끝에 개행 문자(`\n`)가 붙어 있고 `wo_newline`은 그렇지 않다.** 이들 파일의 내용을 보려면 아래와 같이 `cat` 명령을 실행하면 된다.

```console
$ cat w_newline
5ac375dfd15414bb865f10455de23bacac58c1d146908103f0ad79f520cf00881ad4505e5fa8de056e9a00f32335f2e53d7b9c34382e958370d0dad492dfb87e55beee009a8d932ada058b9fc28a9f256157ecfc9a383b18c6736a4d4ccb061d6dc1960c
$ cat wo_newline
5ac375dfd15414bb865f10455de23bacac58c1d146908103f0ad79f520cf00881ad4505e5fa8de056e9a00f32335f2e53d7b9c34382e958370d0dad492dfb87e55beee009a8d932ada058b9fc28a9f256157ecfc9a383b18c6736a4d4ccb061d6dc1960c$
```

## `read -n` 명령을 일단 사용해 보자

자, 이제 준비가 되었으니 원래 하려던 이야기로 돌아가자. 기본적으로 `read` 명령은 표준 입력으로부터 **줄 단위**로 입력을 받는다. 하지만 `read` 명령의 `-n` 옵션을 이용하면 (한 줄의 내용 중에서) **최대로 읽을 바이트 수를 지정**할 수 있다. 이를 이용하여 특정 길이만큼 읽어서 출력하는 작업을 반복하면 원하는 결과를 얻을 수 있다.

앞서 만든 `w_newline` 파일과 `wo_newline` 파일의 내용을 각각, `read -n` 명령을 이용하여 40개 문자마다 줄바꿈해서 출력해 보자.

```console
$ cat w_newline | while read -n 40 line; do echo $line; done
5ac375dfd15414bb865f10455de23bacac58c1d1
46908103f0ad79f520cf00881ad4505e5fa8de05
6e9a00f32335f2e53d7b9c34382e958370d0dad4
92dfb87e55beee009a8d932ada058b9fc28a9f25
6157ecfc9a383b18c6736a4d4ccb061d6dc1960c

$ cat wo_newline | while read -n 40 line; do echo $line; done
5ac375dfd15414bb865f10455de23bacac58c1d1
46908103f0ad79f520cf00881ad4505e5fa8de05
6e9a00f32335f2e53d7b9c34382e958370d0dad4
92dfb87e55beee009a8d932ada058b9fc28a9f25
6157ecfc9a383b18c6736a4d4ccb061d6dc1960c
$
```

`cat` 명령만 단독으로 실행하면 지정한 파일의 내용이 화면에 출력된다. 하지만 위와 같이 실행하면 `cat` 명령의 출력을 파이프(`|`)로 연결된 while 루프가 입력으로 받아서 처리한다. `read -n 40 line` 명령은 표준 입력으로부터 **개행 문자가 나오기 전까지 최대 40바이트까지만 읽어서 그 값을 `line`이라는 이름의 변수에 저장**하라는 뜻이다. 이렇게 읽어들인 `$line` 변수의 값은 `echo $line` 명령에 의해 출력된다. 읽고 남은 값은 버려지는 게 아니라 다음 번 `read` 명령 실행시에 읽혀진다. 표준 입력의 모든 데이터를 다 처리할 때까지 이 과정이 반복된다.

결과를 보면 양쪽 모두 글자 40개마다 줄바꿈이 되어 출력되었다. 하지만 둘을 분명히 차이가 있다. 첫 번째는 끝에 불필요한 빈 줄이 하나 출력되었다. 두 번째가 원하는 결과이다.

첫 번째는 왜 빈 줄이 출력된 것일까? w_newline 파일의 내용 끝에 불필요한 개행문자가 붙어있기 때문일까? `w_newline`과 `wo_newline`, 이들 두 파일의 차이는 끝에 붙은 개행문자 뿐이기 때문에 개행문자가 원인일 수 밖에 없다. 하지만 만일 거기서 생각을 멈춘다면 **개행문자가 왜 문제를 일으키는지**에 대한 답은 찾을 수 없을 것이다.

## `read` 명령의 함정: 개행 문자로 끝나지 않는 파일의 마지막 줄

뿐만 아니라 위 결과만 놓고 ‘한 줄짜리 출력에는 끝에 개행문자를 붙이면 안 된다’는 원칙을 강화하는 것은 섣부른 판단이다. 만약에 위의 `read` 명령에서 `-n 40` 옵션이 없으면 어떻게 될까? 과연 그 때도 wo_newline 쪽이 정상적으로 출력될까? 실행해 보자.

```console
$ cat w_newline | while read line; do echo $line; done
5ac375dfd15414bb865f10455de23bacac58c1d146908103f0ad79f520cf00881ad4505e5fa8de056e9a00f32335f2e53d7b9c34382e958370d0dad492dfb87e55beee009a8d932ada058b9fc28a9f256157ecfc9a383b18c6736a4d4ccb061d6dc1960c
$ cat wo_newline | while read line; do echo $line; done
$
```

이번에는 오히려 끝에 개행문자가 붙은 첫 번째(`w_newline` 쪽)가 정상적으로 출력되었다. 하지만 그보다 더 놀라운 것은, 끝에 개행문자가 붙지 않은 두 번째(`wo_newline` 쪽)는 아예 아무것도 출력되지 않았다는 사실이다. 이런 일을 겪으면 몹시 당황하게 된다.

왜 wo_newline 쪽은 아무것도 출력이 되지 않은 것일까? 일단 while 루프 안의 `echo $line`을 `echo foo`로 바꿔서 실행해 보면, 역시 아무것도 안 찍힌다. while 루프 안의 명령이 한 번도 실행되지 않는 것이다. 그래서 인터넷에서 이와 관련된 정보를 찾아보았는데, 같은 증상과 이 문제의 해결 방법을 어렵지 않게 찾을 수 있었다. 찾은 문서의 링크는 아래와 같다.

- [Read a line-oriented file which may not end with a newline](https://unix.stackexchange.com/questions/418060/read-a-line-oriented-file-which-may-not-end-with-a-newline) (Unix Stack Exchange)

위 문서에 나와있는 대로, 아래와 같이 while 루프의 조건식(?)에 `|| [ -n "$line" ]`을 추가하면 wo_newline 쪽도 잘 실행된다.

```console
$ cat wo_newline | while read line || [ -n "$line" ]; do echo $line; done
5ac375dfd15414bb865f10455de23bacac58c1d146908103f0ad79f520cf00881ad4505e5fa8de056e9a00f32335f2e53d7b9c34382e958370d0dad492dfb87e55beee009a8d932ada058b9fc28a9f256157ecfc9a383b18c6736a4d4ccb061d6dc1960c
$
```

`read` 명령이 **개행 문자로 끝나지 않는 파일의 마지막 줄을 읽을 때**, **종료 상태**(exit status)는 **0이 아닌 값**(failure를 의미함)이 되지만 **그 경우에도 마지막 줄의 전체 내용은 변수에 정상적으로 저장된다.**

(아마도, read 명령은 (`-n` 옵션을 지정하지 않았을 경우) **개행 문자** 또는 **파일의 끝**에 도달하기 전까지의 문자들을 읽어서 인수로 지정한 이름의 셸 변수에 저장하는데, 개행 문자에 도달한 경우 종료 상태 0(success를 의미함)을 리턴하고, 파일의 끝에 도달한 경우 종료 상태로 0이 아닌 값(faulure를 의미함)을 리턴하는 방식으로 작동하는 것 같다.)

그러므로 만일, C를 비롯한 여러 프로그래밍 언어에서 하듯이, 혹은 셸 프로그래밍에서 종료 코드를 다루는 보통의 방법처럼, **`read` 명령의 종료 상태만 가지고 루프 탈출 여부를 판단하면 개행 문자로 끝나지 않는 파일에서 마지막 줄의 처리가 누락된다.** (특히, 위 예제처럼 입력 내용이 한줄 뿐이면 전체의 처리가 누락된다.)\
개행 문자로 끝나지 않는 파일에서도 정상 작동하도록 하려면  위와 같이 **`read` 명령이 실패했을 경우** `$line`변수까지 확인해서 **`$line` 변수가 비어 있는 경우에만 루프를 탈출**하도록 코드를 짜야 한다.

물론, `while read line ... `의 입력값이 항상 개행 문자로 끝난다고 보장할 수 있는 경우라면 이런 추가 확인은 생략해도 된다. 하지만 그런 식으로 짜여진 코드가 인터넷에 너무 많고 이들 코드에는 대부분 그 어떤 설명도 없다는 것이 문제다. `read` 명령의 작동 방식에 대해 잘 모르는 상태에서 그러한 코드만 보면, C를 비롯한 여러 프로그래밍 언어가 그러하듯, 또한 셸 프로그래밍에서 종료 코드를 다루는 방식이 그러하듯 그것만으로 충분하다고 오해하기 딱 좋다.

## 데이터 길이가 한 줄에 출력할 문자 개수의 배수가 아니라면?

주목할 만한 사실이 한 가지 더 있다. 지금까지 우리는 200글자를 한 줄에 40개씩 출력했는데, 여기서 200은 40의 배수이다. 만일 전체 글자수가 40의 배수가 아닌 경우라면 어떻게 될까? 이번에는 220개의 랜덤한 16진수 문자를 한 줄에 40개씩 찍어보자. 먼저 아래 명령을 실행하여 `220_w_newline`와 `220_wo_newline` 파일을 만들자.

```console
$ xxd -l 110 -p -c 110 /dev/urandom > 220_w_newline
$ cat 220_w_newline | tr -d '\n' > 220_wo_newline
$ cat 220_w_newline
499d37ed30585bb8da147b781bab873d02659f4e7c6bbb94c7cd5925999112cb605900fe550465b0853f9f744326faf0de74ae574c4a7aab7d91274e4b18fd1e49a97dc0221971f3e24e52e93f39484e7077ebb4c759dcf689116907300bad9e55b7beb5767829784523add178b0
$ cat 220_wo_newline
499d37ed30585bb8da147b781bab873d02659f4e7c6bbb94c7cd5925999112cb605900fe550465b0853f9f744326faf0de74ae574c4a7aab7d91274e4b18fd1e49a97dc0221971f3e24e52e93f39484e7077ebb4c759dcf689116907300bad9e55b7beb5767829784523add178b0$
```

이후 아래 명령을 실행해 보자.

```console
$ cat 220_w_newline | while read -n 40 line; do echo $line; done
499d37ed30585bb8da147b781bab873d02659f4e
7c6bbb94c7cd5925999112cb605900fe550465b0
853f9f744326faf0de74ae574c4a7aab7d91274e
4b18fd1e49a97dc0221971f3e24e52e93f39484e
7077ebb4c759dcf689116907300bad9e55b7beb5
767829784523add178b0
$ cat 220_wo_newline | while read -n 40 line; do echo $line; done
499d37ed30585bb8da147b781bab873d02659f4e
7c6bbb94c7cd5925999112cb605900fe550465b0
853f9f744326faf0de74ae574c4a7aab7d91274e
4b18fd1e49a97dc0221971f3e24e52e93f39484e
7077ebb4c759dcf689116907300bad9e55b7beb5
$
```

이번에도 방금 전처럼 개행 문자가 붙은 첫 번째(`220_w_newline` 쪽)가 정상적으로 출력되었다. 개행 문자가 붙지 않은 두 번째(`220_wo_newline` 쪽)는 마지막 20개가 출력되지 않았다. 그런데 어째 마지막 줄이 누락되는 증상이 방금 전과 비슷하다. 같은 방법으로 해결할 수 있지 않을까? 확인해 보자.

```console
$ cat 220_wo_newline | while read -n 40 line || [ -n "$line" ]; do echo $line; done
499d37ed30585bb8da147b781bab873d02659f4e
7c6bbb94c7cd5925999112cb605900fe550465b0
853f9f744326faf0de74ae574c4a7aab7d91274e
4b18fd1e49a97dc0221971f3e24e52e93f39484e
7077ebb4c759dcf689116907300bad9e55b7beb5
767829784523add178b0
$
```

빙고! 바로 이거다. 저걸 보는 순간 느낌이 딱 왔다. `read` 명령의 `-n` 옵션에 대한 나의 가설은 다음과 같다.

- `read` 명령에 `-n NUM` 옵션을 지정하면, NUM개의 문자가 다 채워지는 것이 **개행 문자(`\n`)를 만난 것과 동일하게 취급된다.**
- 따라서 NUM개의 문자가 다 채워지면 `read` 명령은 종료 상태 `0`(success)을 반환한다. 이 경우 그 뒤에 무슨 문자가 있는지 혹은 파일의 끝인지는 따지지 않는다.
- 다음번 `read` 명령 실행시에는 그 다음 문자(`\n`일수도, 파일의 끝일 수도 있다)부터 처리된다.

가설이 맞는지 바로 실험해 보자.

```console
$ xxd -l 60 -p -c 20 /dev/urandom
bfdd8b5ba47f7fe3c98585a79491d622e99f547d
e300837a506ae0a053cef0b097a5183ffefd2126
a33ed9fbbbb1351dce2004426c4da00264be9012
$ xxd -l 60 -p -c 20 /dev/urandom | while read -n 40 line; do echo $line; done
55c246cf4447e81d14761b2949ccbeec2402d78e

42a912d1c083ed11387726c476923a982f043afa

1e9a8850a370d5f04bcc17ea87c390beaaf81169

$
```
빙고! 역시 내 가설이 틀리지 않았다.\
`xxd -l 60 -p -c 20 /dev/urandom`은 그 자체로 16진수 글자 40개마다 줄바꿈을 하여 총 120개 글자를 출력한다. 이 출력을 파이프로 `while read -n 40 line; do echo $line; done`에 보내면,
- 첫 번째로 실행된 read 명령은 처음 40개 글자를 `$line`에 채우고 종료 상태 0(success)을 리턴한다.
- 두 번째로 실행된 read 명령은 그 다음 글자가 개행 문자이므로 `$line`에는 빈 문자열이 채워지고 종료 상태 0(success)을 리턴한다. 여기서 `$line`을 출력하면 빈 줄이 생긴다.
- 세 번째/다섯 번째로 실행된 read 명령은 첫 번째로 실행된 read 명령과 동일하게 작동한다.
- 네 번째/여섯 번째로 실행된 read 명령은 두 번째로 실행된 read 명령과 동일하게 작동하며, 따라서 빈 줄이 생긴다.
- 일곱 번째로 read 명령이 실행될 때에는 파일의 끝에 도달했으므로 `$line`에는 빈 문자열이 채워지고 0이 아닌 종료 상태(failure)가 리턴되어 while 루프를 빠져나오게 된다. (루프를 빠져나왔기 때문에 `$line`에 채워진 빈 문자열이 출력되지는 않는다.)

앞에서 `cat w_newline | while read -n 40 line; do echo $line; done`를 실행했을 때 맨 끝에 빈 줄이 하나 생기는 것도 방금 전에 한 실험과 동일한 원인의 증상이다. 40글자씩 총 5번을 읽어서 200글자를 모두 출력한 이후에는 그 다음 글자가 개행문자이므로 `$line`에는 빈 문자열이 들어가고 `read` 명령은 종료 상태 0(success)을 리턴하여 `echo $line`이 실행된 결과로 빈 줄이 하나 출력된 것이다. (그 이후에는 파일의 끝에 도달했으므로 `read` 명령이 0이 아닌 종료 상태(failure)를 반환하여 루프에서 빠져나오게 된다.)

`w_newline` 쪽에서 빈 줄이 생기는 원인을 파악했으니, 이제 그 문제도 해결해보자.\
방법은 어렵지 않다. `wo_newline`의 경우와 비슷한 방법을 쓰면 된다. `$line` 변수가 비어있지 않을 때에만 그 값을 출력하도록 코드를 수정하면 된다.

```console
$ cat w_newline | while read -n 40 line && [ -n "$line" ]; do echo $line; done
5ac375dfd15414bb865f10455de23bacac58c1d1
46908103f0ad79f520cf00881ad4505e5fa8de05
6e9a00f32335f2e53d7b9c34382e958370d0dad4
92dfb87e55beee009a8d932ada058b9fc28a9f25
6157ecfc9a383b18c6736a4d4ccb061d6dc1960c
$
```

## 결론

이상의 논의를 정리해보자. 한 줄짜리 입력값의 끝에 개행 문자가 붙은 쪽과 붙지 않은 쪽 모두, 단순히 `while read -n 40 line`으로 루프를 돌려서 `$line`을 출력하면
- 전자는 (개행 문자를 제외한) 입력값의 길이가 **40의 배수인 경우에** 불필요한 빈 줄이 생기며,
- 후자는 입력값의 길이가 **40의 배수가 아닌 경우에** 마지막 줄이 누락된다.

하지만,
- 전자는 `&& [ -n "$line" ]`을
- 후자는 `|| [ -n "$line" ]`을

`while read -n 40 line` 뒤에 덧붙이는 방법으로 상기한 문제들을 쉽게 해결할 수 있다.

문제점을 보완한 코드는 아래와 같이 입력값의 길이에 관계없이 잘 작동한다. 특히, 입력값의 길이가 0인 경우에는 양쪽 모두 아무것도 출력되지 않는다.

```console
$ cat w_newline | while read -n 40 line && [ -n "$line" ]; do echo $line; done
5ac375dfd15414bb865f10455de23bacac58c1d1
46908103f0ad79f520cf00881ad4505e5fa8de05
6e9a00f32335f2e53d7b9c34382e958370d0dad4
92dfb87e55beee009a8d932ada058b9fc28a9f25
6157ecfc9a383b18c6736a4d4ccb061d6dc1960c
$ cat 220_w_newline | while read -n 40 line && [ -n "$line" ]; do echo $line; done
499d37ed30585bb8da147b781bab873d02659f4e
7c6bbb94c7cd5925999112cb605900fe550465b0
853f9f744326faf0de74ae574c4a7aab7d91274e
4b18fd1e49a97dc0221971f3e24e52e93f39484e
7077ebb4c759dcf689116907300bad9e55b7beb5
767829784523add178b0
$ echo | while read -n 40 line && [ -n "$line" ]; do echo $line; done
$
```

```console
$ cat wo_newline | while read -n 40 line || [ -n "$line" ]; do echo $line; done
5ac375dfd15414bb865f10455de23bacac58c1d1
46908103f0ad79f520cf00881ad4505e5fa8de05
6e9a00f32335f2e53d7b9c34382e958370d0dad4
92dfb87e55beee009a8d932ada058b9fc28a9f25
6157ecfc9a383b18c6736a4d4ccb061d6dc1960c
$ cat 220_wo_newline | while read -n 40 line || [ -n "$line" ]; do echo $line; done
499d37ed30585bb8da147b781bab873d02659f4e
7c6bbb94c7cd5925999112cb605900fe550465b0
853f9f744326faf0de74ae574c4a7aab7d91274e
4b18fd1e49a97dc0221971f3e24e52e93f39484e
7077ebb4c759dcf689116907300bad9e55b7beb5
767829784523add178b0
$ echo -n | while read -n 40 line || [ -n "$line" ]; do echo $line; done
$
```


## 일반화

위 while 루프가 완벽한 것은 아니다. 위 명령에서의 while loop는 십육진수 문자들로만 이루어진 문자열에 대해서는 잘 작동하지만, 일반적인 문자열에 대해서는 정상 작동이 보장되지 않는다. 일반적인 문자열에 대해서도 잘 작동하는 코드는 다음과 같다.

* **널 문자와 개행 문자를 포함하지 않는** 일반적인 문자열 데이터를 40글자마다 줄바꿈하여 출력하려면, 다음과 같이 하면 된다. (앞에서와 마찬가지로, 여기서 `w_newline`은 끝에 개행문자가 붙어있는 파일이며, `wo_newline`은 끝에 개행문자가 붙어있지 않은 파일이다. 개행문자는 입력값에 포함되지 않는 것으로 한다.)
    ```
    cat w_newline | while IFS= read -r -n 40 line && [ -n "$line" ]; do printf "%s\n" "$line"; done
    또는
    cat wo_newline | while IFS= read -r -n 40 line || [ -n "$line" ]; do printf "%s\n" "$line"; done
    ```

* **널 문자를 포함하지 않는** 일반적인 문자열 데이터를 40글자마다 줄바꿈하여 출력하려면, 다음과 같이 하면 된다. (여기서 `file`은 입력 데이터를 담고 있는 파일이다. 그리고 여기서는 파일에 포함된 개행 문자를 행 구분 문자로 취급하지 않고 다른 문자와 동일하게 취급한다. 즉, 개행문자 역시 입력값에 포함되는 것으로 한다.)
    ```
    cat file | while IFS= read -r -d "" -n 40 line || [ -n "$line" ]; do printf "%s\n" "$line"; done
    ```

이들 코드를 앞선 코드와 비교해 보면 `IFS=`, `-r` 옵션, `-d` 옵션이 추가되었고 출력문이 `printf "%s\n" "%line"`으로 변경된 것이 눈에 띈다. 여기에 대해서도 설명이 필요하지만, 글이 너무 길어져서 아래 스택오버플로우 링크로 대신하고자 한다. 아래 링크의 첫 번째 답변에 설명이 잘 되어 있다.

- [What does IFS= do in this bash loop: `cat file | while IFS= read -r line; do ... done`](https://stackoverflow.com/questions/26479562/what-does-ifs-do-in-this-bash-loop-cat-file-while-ifs-read-r-line-do) (Stack Overflow)

그리고 만일, 입력 데이터에 널 문자가 포함되어 있다면, 이 방법으로는 불가능하다. 왜냐하면, [적어도 bash에서는, 셸 변수가 널 문자를 수용할 수 없기 때문이다.](https://stackoverflow.com/questions/6570531/assign-string-containing-null-character-0-to-a-variable-in-bash) bash의 셸 변수는 null-terminated string으로 구현되었기 때문에 파일의 내용을 셸 변수로 받아서 출력하면 널 문자 이후가 출력에서 누락된다.\
입력 데이터를 16진수 문자로 변환한 다음 이를 셸 변수로 받아서 다시 바이트로 변환한 값을 출력하는 식으로 우회할 수는 있지만, 루프 안에서 이 작업을 작은 단위로 반복하는 것은 매우 비효율적이다. 이 경우에는 루프와 셸 변수를 이용하지 않는 다른 방법을 써야 한다.  

그리고 중요한 것. `read` 명령의 `-n` 옵션은 모든 환경에서 지원되는 것이 아니다. 이를테면 아래 링크에 나온 대로, FresBSD에서는 `read -n`이 지원되지 않는다고 한다.
- [What is the FreeBSD equivalent of "read -n"?](https://unix.stackexchange.com/questions/366030/what-is-the-freebsd-equivalent-of-read-n) (Unix Stack Exchange)

이런 경우 역시 다른 방법을 이용해야 할 것이다. 다른 방법들도 정리해보고 싶지만, 글이 길어져서 다음에 해야 할 것 같다.

