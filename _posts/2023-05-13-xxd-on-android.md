---
layout: post
title:  "안드로이드 셸에서 xxd 실행하기"
date:   2023-05-13 23:00:00 +0900
---

지난번에 쓴 [sed를 이용하여 일정한 길이마다 줄바꿈하여 출력하기](https://nooriro.github.io/230506/using-sed-to-insert-newline-every-n-chars)에서 수정하고 싶은 부분이 또 있다. 저 글을 처음 작성했을 때에는 아래 명령들을 실행하여 더미 데이터 파일 4개를 생성하면 된다고 서술했다.

```shell
xxd -l 100 -p -c 100 /dev/urandom 200_w_newline
xxd -l 110 -p -c 110 /dev/urandom 220_w_newline
tr -d '\n' < 200_w_newline > 200_wo_newline
tr -d '\n' < 220_w_newline > 220_wo_newline
```

이 명령들은 우분투의 셸에서는 잘 실행되지만, 안드로이드의 셸에서는 제대로 실행되지 않는다. 정확히 말하자면, 위 명령들 중에서 1행과 2행에 있는 `xxd` 명령이 안드로이드 셸에서는 실행되지 않고 에러를 내뿜는다. (참고로 이는 Pixel 3 XL의 Android 10 `QQ3A.200805.001` 펌웨어에서 확인한 결과이다.)


## 수정된 명령

도대체 왜 안 되는 것일까? 안드로이드 셸에서 직접 확인해본 바에 따르면, 안드로이드에 내장된 `toybox xxd`는 다음과 같은 특징을 가지고 있다. (참고: `toybox xxd`의 [소스 파일](https://github.com/landley/toybox/blob/master/toys/other/xxd.c)과 [수정 이력](https://github.com/landley/toybox/commits/master/toys/other/xxd.c))

- **출력 파일명을 명령행 인수로 지정할 수 없다.** 즉, **무조건 표준출력(stdout)으로 결과를 출력**한다.
- **`-p` 옵션을 지정**하여 실행하면 **`-c` 옵션으로 지정한 값은 무시되며, 무조건 30바이트(십육진수 문자 60개)마다 줄바꿈**하여 출력한다.
  - 이 경우 줄바꿈시 개행 문자만 삽입되는 것이 아니라 **스페이스(space) 문자와 개행(newline) 문자의 조합**(`0x20 0x0A`)이 삽입된다. 그래서 줄 끝에 스페이스 문자가 하나씩 붙는다.\
  (단, 마지막 줄은 십육진수 문자 60개를 가득 채운 경우에만 끝에 스페이스 문자가 붙는다.)
  
그러므로 이같은 `toybox xxd`의 특징을 고려하여 위의 명령들을 아래와 같이 수정하면, 우분투와 안드로이드에서 모두 잘 실행된다.
```shell
xxd -p -l 100 /dev/urandom | tr -d '\n ' > 200_wo_newline
xxd -p -l 110 /dev/urandom | tr -d '\n ' > 220_wo_newline
{ cat 200_wo_newline; echo; } > 200_w_newline
{ cat 220_wo_newline; echo; } > 220_w_newline
```

## 수정된 명령에 대한 설명

자, 원래 명령이 무슨 과정을 거쳐서 저렇게 수정된 것인지를 하나씩 이야기해보자.

- 일단 `xxd` 명령에서 **`-c` 옵션을 제거**했다. (안드로이드의 `xxd`에서는 `-p` 옵션을 지정하면 `-c` 옵션은 무시되므로 이는 삭제해도 무방하다.)
- 안드로이드에서 `xxd -p` 명령만으로는, 중간에 개행문자가 없고 마지막에만 개행문자가 들어간 `_w_newline` 파일들을 생성할 수 없다. (이것 역시, `-c` 옵션이 듣지 않기 때문이다.) 그래서 일단, `xxd`의 출력을 **`tr -d '\n '` 명령**(스페이스가 추가되었음에 유의)으로 보내서 개행 문자와 스페이스 문자를 모두 제거한 **`_wo_newline` 파일들을 먼저 생성**하도록 했다.
- 이후 `_wo_newline` 파일들 **끝에 개행문자를 덧붙여서 `_w_newline` 파일들을 생성**하면 작업이 완료된다.

마지막에 개행문자를 덧붙이는 방법은 한 번에 생각해 낸 것이 아니다. 깔끔한 방법을 찾기 위해 며칠 동안 나름 고심한 끝에 생각해 낸 방법이다. 

- 처음에 생각해 본 방법은 `cp 200_wo_newline 200_w_newline` 명령으로 파일을 복사한 다음 `echo >> 200_w_newline` 명령으로 복사한 파일 끝에 개행 문자를 덧붙이는 것이었는데, 솔직히 좀 지저분해 보여서 마음에 들지 않았다. (`200_w_newline` 파일의 최종 결과물을 얻기 위해 두 단계의 쓰기 작업을 해야 한다.)

- 다음으로 생각한 방법은 개행 문자 1개만을 담은 임시 파일을 이를테면 `newline` 같은 이름으로 만들고 `cat 200_wo_newline newline > 200_w_newline` 명령을 실행하는 방법이었는데, 역시 지저분해 보여서 마음에 들지 않았다. (`200_w_newline` 파일은 한번에 만들어지지만, 사전에 `newline` 파일을 생성하고 사후에 `newline` 파일을 제거해야 한다.)\
사실 이 방법을 생각하기 전에 먼저 떠올린 명령은 `cat 200_wo_newline $'\n' > 200_w_newline` 이었는데, 물론 저건 완전 엉터리 명령이고 저렇게 하면 안 된다.

- 이 문제를 해결하는 방법을 인터넷에서 찾다가 알게 된 것이 `sed '$a\'` 명령이다. ([참고](https://unix.stackexchange.com/questions/31947/how-to-add-a-newline-to-the-end-of-a-file)) `sed '$a\' 200_wo_newline` 처럼 실행하면 되는데 보다시피 명령이 매우 간결하다. 또한 이 명령은 파일 끝에 개행 문자가 없는 경우에만 개행 문자를 덧붙이므로 활용 가치가 높다. 문제는 이 방법이 일부 환경에서만 기대한 대로 작동한다는 것이다. 직접 확인해 본 결과, 우분투와 안드로이드의 기본 셸에서는 잘 작동했지만 비지박스(`busybox`)에서는 그렇지 않았다. 그래서 이 방법은 일단 보류하기로 했다.

- 한참을 돌고 돌아서 정착한 방법은 위와 같이 **`cat 200_wo_newline` 명령의 출력과 `echo` 명령의 출력을 합쳐서 `200_w_newline` 파일에 저장**하는 방법이다. 이렇게 두 명령의 실행 결과를 합쳐서 저장하고 싶다면 **두 명령을 전부 중괄호 `{ }`로 둘러싸서 파일로 리디렉션(`>`)**하면 된다.\
참고로 저렇게 아무런 명령행 인수 없이 `echo`를 실행하면 딱 개행문자만 출력된다.\
덧붙이자면 위와 같이 `{ }` 부분을 중괄호까지 포함해서 한 줄로 작성할 경우에는 중괄호 안의 각 명령 끝에 세미콜론 `;`을 붙여야 하며, 특히 **마지막 명령 끝에 세미콜론을 빼먹으면 안 된다!!** 이 사실을 알고 있어도 무의식적으로 마지막 세미콜론을 빼먹고 에러를 내는 경우가 많으니 주의!!

중괄호 `{ }`의 존재는 전부터 알고 있었지만, 중괄호를 저렇게 **여러 명령의 출력을 합쳐서 리디렉션**하기 위한 용도로 사용해본 적은 사실 그 동안 단 한 번도 없었다. 최근 며칠 동안 셸 관련 자료를 이것 저것 찾아보던 중에 눈에 들어왔던 중괄호가 기억에 희미하게 남았고 오늘 이 문제의 해법을 다시 생각하던 중에 문득 그 중괄호가 생각나서 이렇게 하면 될 것 같았다. 그래서 해봤는데, 다행스럽게도 기대한 대로 잘 실행되었다.

이것으로 핵심은 다 이야기했다. 이후에는 위 수정된 명령들의 몇 가지 바리에이션을 탐구해 보자.


## w 파일만 (혹은 w 파일 먼저) 생성하기

앞에서 언급했듯이, 위의 수정된 명령들은 끝에 개행문자가 없는 `_wo_newline` 파일들을 우선 만든 다음, 그 끝에 개행문자를 덧붙여서 `_w_newline` 파일들을 생성해낸다.\
하지만 만일 `_wo_newline` 파일들이 필요하지 않다면, 아래와 같이 한번에 `_w_newline` 파일들만 생성하는 것이 효과적이다.
```shell
{ xxd -p -l 100 /dev/urandom | tr -d '\n '; echo; } > 200_w_newline
{ xxd -p -l 110 /dev/urandom | tr -d '\n '; echo; } > 220_w_newline
```

이 경우에는 `xxd 어쩌고 | tr 저쩌고`의 출력을 굳이 파일로 저장했다가 다시 `cat` 명령으로 불러올 필요가 없다. `xxd 어쩌고 | tr 저쩌고`를 실행한 다음 **바로 이어서 `echo`를 실행**하면 앞선 명령의 출력 끝에 개행 문자가 추가된다. 그러므로 명령 전체를 중괄호 `{ }`로 감싸고 그 전체를 리디렉션하여 파일로 저장하면 된다.

참고로 이들 `_w_newline` 파일로부터 아래와 같이 마지막 문자 1개를 제거하면 `_wo_newline` 파일들을 얻을 수 있다.
```shell
head -c -1 200_w_newline > 200_wo_newline
head -c -1 220_w_newline > 220_wo_newline
```
`head -c -1` 명령으로 파일 끝의 1바이트를 무조건 제거했는데 여기서는 이 방법이 가장 좋다.\
`tr -d '\n'` 명령을 사용해도 결과는 같지만, 파일 전체를 바이트 단위로 스캔하는 불필요한 작업을 하게 되므로 여기서 그 방법을 쓰는 것은 좋지 않다.


## `tee` 명령을 이용하여 w 파일과 wo 파일을 한번에 만들기

앞에서 소개한 수정된 명령들을 다시 생각해보자.
```shell
xxd -p -l 100 /dev/urandom | tr -d '\n ' > 200_wo_newline
xxd -p -l 110 /dev/urandom | tr -d '\n ' > 220_wo_newline
{ cat 200_wo_newline; echo; } > 200_w_newline
{ cat 220_wo_newline; echo; } > 220_w_newline
```

여기서 `_wo_newline` 파일들은 그 자체가 결과물이지만 동시에 `_w_newline` 파일을 만들기 위한 중간 단계의 역할도 한다. 그렇기 때문에 일단 리디렉션 `>`으로 `_wo_newline` 파일을 저장한 다음, 다시 `cat` 명령으로 방금 저장한 `_wo_newline` 파일을 불러와서 이후 과정을 작업해야 했다.

만일 위 명령의 과정에서, 리디렉션 `>`으로 `_wo_newline` 파일을 저장하는 것과 `cat` 명령으로 방금 저장한 `_wo_newline` 파일을 불러오는 것을 **하나의 명령으로 통합**할 수 있다면, `_w_newline` 파일 생성에 이르는 전 과정을 하나의 파이프라인으로 처리할 수 있지 않을까?

POSIX에는 정확히 이것을 가능케 하는 명령이 있다. 바로 **`tee`**라는 명령이다. `tee` 명령은 표준입력으로부터 받은 데이터를 그대로, **표준출력**과 **인수로 지정한 파일들** 모두에 동일하게 출력한다.\
(참고: `toybox tee`의 [소스 파일](https://github.com/landley/toybox/blob/master/toys/posix/tee.c))

실행 방법은 아래와 같다. 위 명령과 비교하여 어느 부분이 어떻게 바뀌었는지를 눈여겨 보자.
```shell
{ xxd -p -l 100 /dev/urandom | tr -d '\n ' | tee 200_wo_newline; echo; } > 200_w_newline
{ xxd -p -l 110 /dev/urandom | tr -d '\n ' | tee 220_wo_newline; echo; } > 220_w_newline
```

다시 말하자면 **리디렉션과 `cat` 명령의 조합**을 위와 같이 **`tee` 명령 하나로 대신할 수 있다.** 

조금 다른 관점에서 생각해보자. 이렇게 `tee` 명령을 이용하면 코드가 더 간결해진다고 할 수 있을까?

명령의 단계가 통합되면서 전체 글자 수는 102글자에서 88글자로 줄어들었고 두 줄이 한 줄로 줄어든 것 또한 사실이다. 그러나 그 한 줄의 길이(88글자)는 이전(57글자, 45글자)보다 늘어났기 때문에 바뀐 쪽이 더 간결해졌다고 말하기가 조금 애매하다. 판단은 사람마다 다를 수 있지만, 한 줄의 길이가 너무 길어지면 코드가 한 눈에 들어오지 않게 되어 오히려 가독성이 떨어질 수도 있다.

하지만 이렇게 전체 명령을 한 줄(one-liner)로 만들어 놓으면 **복사/붙여넣기로 명령을 실행하는 것은 확실히 수월해진다.** 그것만으로도 충분히 가치가 있다고 생각한다. 

그리고 중요한 것. `tee` 명령의 한 가지 **단점**은 이 명령이 셸 빌트인(내부 명령)이 아니라 **외부 명령**이라는 것이다. (우분투와 안드로이드에서 전부 그렇다.) 반복 횟수가 많은 루프 안에서는 외부 명령의 개수를 최소화하는 것이 중요하다. 그러므로 루프 안에서 리디렉션과 `cat`의 조합을 `tee`로 바꾸는 것은 생각을 좀 해 봐야 한다.


## `for` 루프를 이용하여 최대로 압축하기

지금까지 보인 명령들은 전부, 완전히 동일한 절차의 명령을 인수만 달리하여 두 번 실행하는 구문이다. 따라서, 비록 반복의 횟수가 2번 뿐이므로 그 효과가 크진 않지만, 아래와 같이 `for` 루프를 이용하여 반복문으로 나타낼 수 있다.\
(`while` 루프로도 가능은 하지만 여기서는 `for` 루프를 쓰는 게 좀 더 간단하다. 언제 기회가 되면 둘의 차이에 대해서도 한번 다뤄보고 싶다.)
```shell
for i in 200 220; do
  xxd -p -l $((i/2)) /dev/urandom | tr -d '\n ' > ${i}_wo_newline
  { cat ${i}_wo_newline; echo; } > ${i}_w_newline
done
```

두 번째 줄에 있는 **`$((i/2))`**는 **arithmetic substitution**이라고 하는 것인데, **`$((`와 `))` 사이에 기술된 수식을 계산한 값**으로 그 전체가 **치환**되는 구문이다. `bash`와 `mksh` 모두 이 문법을 지원한다.\
(참고: 구글 검색 [arithmetic substitution in shell](https://www.google.com/search?q=arithmetic+substitution+in+shell&newwindow=1))\
$i$ 자리 십육진수는 $i/2$ 바이트 데이터에 해당하므로 `xxd`의 `-l` 옵션에는 **`$i`를 2로 나눈 값**을 전달해야 하는데 이렇게 **값을 계산해야 하는 경우**에는 위와 같이 arithmetic substitution을 사용하면 된다.

또한 여기서 parameter substitution을 `$i` 대신 `${i}`로 나타낸 이유는 바로 뒤에 언더스코어(`_`)가 있기 때문이다. 저 자리에 그냥 `$i`를 쓰면 뒤에 이어지는 내용과 연결되어 그 전체가 변수명으로 인식되므로 `$i_wo_newline`이라는 혹은 `$i_w_newline`이라는 전혀 다른 변수를 의미하게 된다.

그리고 위의 `for` 루프 전체를 한 줄로 나타낼 수 있다. 아래와 같이 **`do`와 `done` 사이의 각 줄 끝에 `;`을 붙이고 전체를 한 줄로 연결**하면 된다.
```shell
for i in 200 220; do xxd -p -l $((i/2)) /dev/urandom | tr -d '\n ' > ${i}_wo_newline; { cat ${i}_wo_newline; echo; } > ${i}_w_newline; done
```
가독성은 최악이지만 복사/붙여넣기로 명령을 실행할 때에는 이런 **one-liner**가 상당히 유용하다.


`tee` 명령을 이용한 구문도 반복문으로 나타내보자.
```shell
for i in 200 220; do
  { xxd -p -l $((i/2)) /dev/urandom | tr -d '\n ' | tee ${i}_wo_newline; echo; } > ${i}_w_newline
done
```
```shell
for i in 200 220; do { xxd -p -l $((i/2)) /dev/urandom | tr -d '\n ' | tee ${i}_wo_newline; echo; } > ${i}_w_newline; done
```
`for` 루프를 이용하지 않았을 때에는 `tee` 명령을 이용한 쪽의 가독성이 그렇지 않은 쪽보다 크게 좋아 보이지는 않았다. 하지만 양 쪽을 모두 `for` 루프를 이용하여 나타내고 둘을 비교해보니, **`tee` 명령을 이용한 쪽이** 그렇지 않은 쪽보다 **훨씬 눈에 잘 들어온다.** `tee` 명령의 진가가 여기서 발휘되는 것 같다.

