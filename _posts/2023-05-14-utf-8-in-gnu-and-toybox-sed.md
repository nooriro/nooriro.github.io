---
layout: post
title:  "안드로이드의 sed는 UTF-8 문자를 지원하지 않는다"
date:   2023-05-14 23:00:00 +0900
---

우분투의 `GNU sed`는 UTF-8 문자를 지원하지만, 안드로이드의 `toybox sed`는 UTF-8 문자를 지원하지 않는 것 같다.

유니코드 문자열 `aα가😀`를 고려하자.\
이를 구성하는 4개 문자를 UTF-8로 인코딩하면 아래와 같이 각각 1, 2, 3, 4바이트로 표현된다.
 
|문자|코드 포인트|UTF-8 인코딩|UTF-16 인코딩|
|----|-----------|------------|-------------|
|`a`|`U+0061`|`61`|`0061`|
|`α`|`U+03B1`|`CE B1`|`03B1`|
|`가`|`U+AC00`|`EA B0 80`|`AC00`|
|`😀`|`U+1F600`|`F0 9F 98 80`|`D83D DE00`|

이 문자열을 단순히 출력하는 것은 우분투의 `bash`와 안드로이드의 `mksh`에서 모두 가능하다. 단, 몇 가지 고려할 지점이 있다.
- Windows 10의 `conhost.exe`는 이모지 출력을 지원하지 않으므로 명령 프롬프트, 파워셸, 혹은 Ubuntu on Windows를 ‘그냥’ 실행한 경우에는 `😀`가 제대로 출력되는지를 확인하기 어렵다.\
하지만 [Windows Terminal](https://github.com/microsoft/terminal)은 이모지 출력을 지원하므로 이 문제를 가장 쉽게 우회하는 방법은 **Windows Terminal에서** 명령 프롬프트, 파워셸, 혹은 Ubuntu on Windows **탭을 열고 거기서 확인**하는 것이다.

- 명령 프롬프트에서 `adb shell` 명령으로 안드로이드의 interactive shell에 접속하여 한글 등의 유니코드 문자를 입력하거나 붙여넣으면 그 즉시 명령 프롬프트 창이 먹통이 되어 버리는 치명적인 문제가 있다.
  - `a`/`α`는 문제가 없는데, `가`/`😀`를 입력하거나 붙여넣으면 이 증상이 발생한다.
  - `conhost.exe`뿐만 아니라 Windows Terminal에서도 같은 증상이 나타난다.
  - 명령 프롬프트에서 `adb shell` 바로 뒤에 실행할 명령을 기술하여 실행하는 non-interactive 모드에서는 이 문제가 생기지 않는다. (단, [콘솔에서 UTF-8 입출력이 가능하도록 설정](https://stackoverflow.com/questions/57131654/using-utf-8-encoding-chcp-65001-in-command-prompt-windows-powershell-window/57134096#57134096)이 되어 있어야 하며 이 설정은 Windows 10 1903부터 지원된다.) 하지만 non-interactive 모드에서는 일부 명령이 interactive 모드에서와 다르게 동작한다는 문제가 있다.

  `adb shell`의 interactive 모드에서 이 문제를 우회하려면, `가`/`😀` 같은 문자를 **`$' '` 안에서 이스케이프 시퀀스를 이용**하여 나타내야 한다.

- 안드로이드의 `mksh`는 `$' '` 안에서 `\U`로 시작하는 이스케이프 시퀀스를 제대로 처리하지 못한다. `\uAC00`은 제대로 처리되지만, `\U1F600`은 엉뚱한 바이트열로 변환된다. (우분투의 `bash`에서는 양쪽 모두 제대로 처리된다.) 따라서 `mksh`에서 `U+1F600` 같은 **BMP 바깥의 유니코드 문자는 UTF-8 바이트열로 풀어서** `\xF0\x9F\x98\x80`와 같이 나타내야 한다.

이 글의 목적은 **우분투와 안드로이드에서 `sed` 명령의 차이**를 다루는 것이므로, 앞으로 이 글에서는 이상 언급한 문제들을 전부 우회할 생각이다.

> 이하 명령의 실행 결과는 우분투와 안드로이드 모두 Windows Terminal에서 확인했으며, 특히 안드로이드는 Windows Terminal의 명령 프롬프트에서 `adb shell`명령으로 interactive shell에 접속하여 확인한 결과이다.\
> 구체적인 테스팅 환경은 다음과 같다.
> - 우분투: Ubuntu 20.04 LTS on WSL1 on Windows 10 22H2 `19045.2965`
> - 안드로이드:
>   - Android 10 `QQ3A.200805.001` on Pixel 3 XL:
>     - `echo $KSH_VERSION`: `@(#)MIRBSD KSH R57 2019/03/01 Android`
>     - `toybox --version`: `toybox 0.8.0-android`
>   - Android 11 `RQ1A.210205.004` on Pixel 4a: 
>     - `echo $KSH_VERSION`: `@(#)MIRBSD KSH R57 2019/03/01 Android`
>     - `toybox --version`: `toybox 0.8.3-android`

아래 명령을 실행해 보면, 우분투의 `bash`와 안드로이드의 `mksh`에서 모두 동일한 결과가 출력된다.\
출력되는 것은 `aα가😀`와 그것의 바이트열 표현이다.

```console
$ echo $'aα\uAC00\xF0\x9F\x98\x80' | tee /dev/stderr | xxd -g 1
aα가😀
00000000: 61 ce b1 ea b0 80 f0 9f 98 80 0a                 a..........
$
```

하지만, `sed 's/./&|/g'` 명령을 이용하여 `aα가😀`의 각 문자 뒤에 `|`를 추가하는 것은, 우분투의 `GNU sed`에서만 기대한 대로 실행되며 안드로이드의 `toybox sed`에서는 기대한 대로 실행되지 않는다.

우분투에서:
```console
$ echo $'aα\uAC00\xF0\x9F\x98\x80' | sed 's/./&|/g' | tee /dev/stderr | xxd -g 1
a|α|가|😀|
00000000: 61 7c ce b1 7c ea b0 80 7c f0 9f 98 80 7c 0a     a|..|...|....|.
$
```

안드로이드에서:
```console
$ echo $'aα\uAC00\xF0\x9F\x98\x80' | sed 's/./&|/g' | tee /dev/stderr | xxd -g 1
a|�|�|�|�|�|�|�|�|�|
00000000: 61 7c ce 7c b1 7c ea 7c b0 7c 80 7c f0 7c 9f 7c  a|.|.|.|.|.|.|.|
00000010: 98 7c 80 7c 0a                                   .|.|.
$
```

우분투의 `GNU sed`에서는 정규식의 메타 문자 `.`이 실제로 모든 **문자**에 매치되지만, 안드로이드의 `toybox sed`에서는 이것이 모든 **바이트**에 매치되는 것을 확인할 수 있다.

아래와 같이 안드로이드에서 `sed` 실행시 환경 변수 `LANG` / `LC_ALL`의 값을 `C.UTF-8`로 설정하여 실행해 보았는데, 효과가 없었다.
```console
$ echo $'aα\uAC00\xF0\x9F\x98\x80' | LANG=C.UTF-8 sed 's/./&|/g' | tee /dev/stderr | xxd -g 1
a|�|�|�|�|�|�|�|�|�|
00000000: 61 7c ce 7c b1 7c ea 7c b0 7c 80 7c f0 7c 9f 7c  a|.|.|.|.|.|.|.|
00000010: 98 7c 80 7c 0a                                   .|.|.
$ echo $'aα\uAC00\xF0\x9F\x98\x80' | LC_ALL=C.UTF-8 sed 's/./&|/g' | tee /dev/stderr | xxd -g 1
a|�|�|�|�|�|�|�|�|�|
00000000: 61 7c ce 7c b1 7c ea 7c b0 7c 80 7c f0 7c 9f 7c  a|.|.|.|.|.|.|.|
00000010: 98 7c 80 7c 0a                                   .|.|.
$
```

또한 `toybox sed`의 도움말을 확인해 보았지만, 이 문제를 해결할 수 있는 방법에 대한 단서를 찾지는 못했다.

`toybox sed`가 아예 멀티바이트 문자를 다루지 못하는 것인지, 아니면 `toybox sed`가 멀티바이트 문자를 다룰 수는 있지만 안드로이드 셸에 그 기능을 활성화하는 설정이 되어 있지 않아서 이렇게 작동하는 것인지는 아직 모르겠다. 좀 더 조사를 해 봐야겠다.

<br>

**UPDATE (2023.05.16 21:50)**

아래 명령으로 `toybox`를 Ubuntu on WSL1에서 직접 빌드해서 명령을 실행해 봤는데, 제대로 실행된다.\
도대체 뭐가 문젤까?
```shell
git clone https://github.com/landley/toybox.git
cd toybox
make defconfig
make
```
```console
$ ./toybox --version
toybox 0.8.9-118-g216e4d139826
$ echo $'aα\uAC00\xF0\x9F\x98\x80' | ./toybox sed 's/./&|/g' | tee /dev/stderr | xxd -g 1
a|α|가|😀|
00000000: 61 7c ce b1 7c ea b0 80 7c f0 9f 98 80 7c 0a     a|..|...|....|.
$
```

버전을 `0.8.0` (안드로이드 10에 들어있는 `toybox`와 같은 버전)으로 맞춰서 빌드했을 때도 제대로 실행된다. 혼란스럽다.
```shell
git clone https://github.com/landley/toybox.git
cd toybox
git checkout 0.8.0
make defconfig
make
```
```console
$ ./toybox --version
toybox 0.8.0
$ echo $'aα\uAC00\xF0\x9F\x98\x80' | ./toybox sed 's/./&|/g' | tee /dev/stderr | xxd -g 1
a|α|가|😀|
00000000: 61 7c ce b1 7c ea b0 80 7c f0 9f 98 80 7c 0a     a|..|...|....|.
$
```

<br>

**UPDATE (2023.05.17 03:30)**

아래 첫 번째 링크를 통해, 아래 두 번째 링크에서 `toybox`의 공식 프리빌트 바이너리를 다운로드할 수 있고 그것이 안드로이드에서도 실행된다는 사실을 알게 되었다. (단, 아키텍처가 일치해야 실행된다.)
- <https://github.com/landley/toybox/issues/335#issuecomment-1111689430>
- <https://landley.net/toybox/downloads/binaries/latest/>

그래서 `toybox-aarch64` 파일을 다운로드하고 Pixel 3 XL의 Android 10에서 실행해보았는데, 이번에도 `sed 's/./&|/g'` 명령이 기대한 대로 잘 실행되었다.
```batchfile
adb push "%USERPROFILE%\Downloads\toybox-aarch64" /data/local/tmp/
adb shell
```
```console
$ cd /data/local/tmp
$ ls -l toybox-aarch64
-rw-rw-rw- 1 shell shell 825176 2023-05-17 02:52 toybox-aarch64
$ sha1sum toybox-aarch64
627ca0bacd43cdc5f1f5fb668a8e092bf11994d5  toybox-aarch64
$ chmod u+x toybox-aarch64
$ ./toybox-aarch64 --version
toybox 0.8.9
$ echo $'aα\uAC00\xF0\x9F\x98\x80' | ./toybox-aarch64 sed 's/./&|/g' | tee /dev/stderr | xxd -g 1
a|α|가|😀|
00000000: 61 7c ce b1 7c ea b0 80 7c f0 9f 98 80 7c 0a     a|..|...|....|.
$ exit
```

위 첫 번째 링크의 설명에 따르면, 이들 바이너리는 `musl-libc`를 정적 링크하여 빌드했다고 한다.
- 그렇다면 안드로이드의 `bionic`이 문제인 것일까?
- 설마 AOSP쪽에서 `toybox sed`가 유니코드 문자를 지원하지 **않도록** 빌드 설정을 수정한 것일까?
- 아니면 유니코드 문자를 지원하도록 하여 실행하는 방법이 있는데, 내가 못 찾은 것일까?
- 혹시 안드로이드 최신 버전에서도 이 문제가 발생할까? (사정상 지금 확인해보기 어렵다.)

어쨌든, `toybox sed` 그 자체는 UTF-8 문자를 지원한다는 것이 확인되었기 때문에 글의 제목을 “안드로이드의 sed는 UTF-8 문자를 지원하지 않는다”로 수정했다.
