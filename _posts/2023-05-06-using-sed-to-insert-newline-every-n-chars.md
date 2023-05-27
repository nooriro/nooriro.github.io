---
layout: post
title:  "sed를 이용하여 일정한 길이마다 줄바꿈하여 출력하기"
date:   2023-05-06 17:00:00 +0900
---

아래 글에서 이어지는 내용.

- [셸에서 랜덤한 N바이트를 16진수로 출력하기](https://nooriro.github.io/230414/random-n-bytes-as-hex-string-in-shell)
- [셸에서 `read -n` 명령으로 일정한 글자수마다 줄바꿈하여 출력하기](https://nooriro.github.io/230501/read-n-in-shell)

지난번 글에서는 `read -n` 명령을 `while`문으로 반복 실행하여 텍스트 문자열을 40글자마다 줄바꿈하여 출력하는 것을 다루었다. 이번에는 같은 작업을 `sed`를 이용하여 해 보자. 

## 데이터 파일 만들기

우선 [지난번 글](https://nooriro.github.io/230501/read-n-in-shell)에서 했던 것 처럼 작업에 사용할 더미 데이터 파일들을 만들자. 아래 명령을 실행하면 된다.

```shell
{ xxd -p -l 100 /dev/urandom | tr -d '\n ' | tee 200_wo_newline; echo; } > 200_w_newline
{ xxd -p -l 110 /dev/urandom | tr -d '\n ' | tee 220_wo_newline; echo; } > 220_w_newline
```

위 명령은 우분투와 안드로이드에서 모두 잘 실행된다. 아래 링크에 위 명령에 대한 자세한 설명이 있으니 필요하다면 참고.
- [안드로이드 셸에서 xxd 실행하기](https://nooriro.github.io/230513/xxd-on-android)

실행하면 아래와 같이 4개의 파일이 생성된다.

```console
$ { xxd -p -l 100 /dev/urandom | tr -d '\n ' | tee 200_wo_newline; echo; } > 200_w_newline
$ { xxd -p -l 110 /dev/urandom | tr -d '\n ' | tee 220_wo_newline; echo; } > 220_w_newline
$ ls -l 2*_newline
-rw-r--r-- 1 nooriro nooriro 201 May  6 15:46 200_w_newline
-rw-r--r-- 1 nooriro nooriro 200 May  6 15:46 200_wo_newline
-rw-r--r-- 1 nooriro nooriro 221 May  6 15:46 220_w_newline
-rw-r--r-- 1 nooriro nooriro 220 May  6 15:46 220_wo_newline
```

파일 이름의 규칙은 다음과 같다. 이름만 봐도 각각이 무슨 파일인지 알 수 있다.
- `200_`로 시작하는 파일: 랜덤한 십육진수 문자 200자리의 데이터
- `220_`로 시작하는 파일: 랜덤한 십육진수 문자 220자리의 데이터
- `_w_newline`으로 끝나는 파일: 끝에 개행문자가 덧붙여져 있다
- `_wo_newline`으로 끝나는 파일: 끝에 개행문자가 덧붙여져 있지 않다

각 파일의 내용을 확인하려면 `cat 200_w_newline` 처럼 `cat` 명령을 실행하면 된다.


## `sed`의 가장 기본적인 `s` 명령

구글에서 [add newline at n bytes in shell](https://www.google.com/search?q=add+newline+at+n+bytes+in+shell&newwindow=1) 검색어로 방법을 찾아봤더니 Unix Stack Exchange에 올라온 아래 질문글이 먼저 눈에 들어왔다.
- [How to insert newline characters every N chars into a long string \[duplicate\]](https://unix.stackexchange.com/questions/489775/how-to-insert-newline-characters-every-n-chars-into-a-long-string/489792#489792)
- [How do I insert a space every four characters in a long line?](https://unix.stackexchange.com/questions/5980/how-do-i-insert-a-space-every-four-characters-in-a-long-line/5981#5981)

첫 번째 문서는 한 줄에 출력할 글자수만 다를 뿐 여기서 하려는 것과 같은 질문이고, 두 번째 문서는 첫 번째 문서에 링크된 글인데 질문이 조금 다르다. 이들 문서에 나온 방법을 참고하여 우선 아래와 같은 명령을 생각해보자.
```shell
sed 's/.\{40\}/&\
/g' filename
```

위 명령에서 작은따옴표(`' '`)로 둘러싸인 부분이 sed script이다. 이 sed 스크립트를 분석해 보자.\
sed의 사용 방법은 구글에서 [sed tutorial](https://www.google.com/search?q=sed+tutorial&newwindow=1) 혹은 [sed manual](https://www.google.com/search?q=sed+manual&newwindow=1) 혹은 [sed 사용 방법](https://www.google.com/search?q=sed+%EC%82%AC%EC%9A%A9+%EB%B0%A9%EB%B2%95&newwindow=1) 등으로 검색하면 많이 나오는데, 그 중에서 나는 [Bruce Barnett이 작성한 sed 튜토리얼](https://www.grymoire.com/Unix/Sed.html)을 주로 참고했다.

- **`s/REGEXP/REPLACEMENT/[FLAGS]`**는 **substitute command**(치환 명령)이다. `REGEXP` 패턴을 찾아서 `REPLACEMENT`로 바꾼다. \([참고](https://www.grymoire.com/Unix/Sed.html#uh-1)\)

- 치환 명령의 끝에 붙은 **`g`**는 **global replacement**를 뜻하는 **플래그**이다. g 플래그가 없으면 각각의 줄에서 첫 번째로 매치된 `REGEXP`만 `REPLACEMENT`로 바뀐다. g가 있어야 나머지도 전부 바뀐다. \([참고](https://www.grymoire.com/Unix/Sed.html#uh-6)\)

- **`.\{40\}`**는 ~~개행 문자를 제외한~~ 임의의 문자(`.`) 40개(`\{40\}`)에 매치되는 패턴이다. 수량 지정자(quantifier)에 쓰이는 `{`와 `}`는 저렇게 백슬래시로 이스케이프를 해야 한다. 우리가 흔히 알고 있는 정규식과 이스케이프 방법이 다르므로 유의. \([참고](https://www.grymoire.com/Unix/Regular.html#uh-8)\)

- **`&`**는 `REGEXP`에 매치된 문자열 전체(즉, 찾은 문자열 전체)를 뜻한다. \([참고](https://www.grymoire.com/Unix/Sed.html#uh-3)\)\
참고로, `REGEXP` 안에서 `\(`...`\)`로 캡처한 그룹을 `REPLACEMENT`로 가져올 때에는 `\1`, `\2`, `\3`, ..., `\9`를 사용한다. 그룹 지정에 쓰이는 괄호 `(` `)` 역시 이스케이프를 한다는 것에 유의할 것. \([참고](https://www.grymoire.com/Unix/Sed.html#uh-4)\)

- **`\` 뒤에 바로 줄바꿈이 된 것**은 개행 문자를 뜻한다. 원래 셸 명령행에서 작은따옴표(`' '`)로 둘러싸인 부분은 글자 그대로의 문자열로 취급되며 개행 문자 역시 이스케이프를 필요로 하지 않는다. 하지만 `REPLACEMENT` 안에서 개행 문자를 나타내려면 그 앞에 `\`를 붙여서 이스케이프를 해야 한다.\
(`\n`이 아니라, `\` 뒤에 ‘글자 그대로의 개행문자’이므로 혼동하지 않도록 주의.)\
참고로 `REGEXP` 안에서는 개행 문자를 `\n`으로 나타내야 한다. 굉장히 혼란스럽다. \([참고](https://www.grymoire.com/Unix/Sed.html#uh-nl)\)


## 여러 정규 표현식의 종류

`sed`가 디폴트로 받는 정규식은 **POSIX basic regular expression \(BRE\)**라는 것인데, 여러 명령행 도구에서 이 BRE가 디폴트로 사용된다. \([참고](https://www.grymoire.com/Unix/Regular.html#uh-8)\)\
하지만 앞서 말했듯이 BRE에서는 자주 사용되는 그룹 지정자 `(` `)`와 수량 지정자 `{` `}` 앞에 `\`를 붙여 이스케이프를 해야 한다. 이는 흔히 사용되는 정규표현식의 문법과 달라서 오류를 내기 쉬우며 또한 가독성도 떨어진다.\
뿐만 아니라 BRE는 매우 기본적인 기능만 지원하며 `?` `+` `|` 조차 지원되지 않는다. `?`는 `\{0,1\}`로, `+`는 `\{1,\}`로 대신할 수 있지만 `|`가 지원되지 않는 것은 치명적이다. \([참고1](https://www.regular-expressions.info/posix.html), [참고2](https://en.wikipedia.org/wiki/Regular_expression#Standards)\) GNU grep과 GNU sed에서는 BRE mode에서 확장 문법으로 `\?` `\+` `\|`를 지원하지만 이는 GNU grep/sed에서만 되는 확장 문법일 뿐이다. \([참고](https://learnbyexample.github.io/gnu-bre-ere-cheatsheet/)\)

**POSIX extended regular expression \(ERE\)**라는 것도 있다. ERE는 메타 문자로 사용된 `(` `)` `{` `}`를 이스케이프하지 않으며 BRE보다 많은 기능을 지원한다. 특히, `?` `+` `|`를 지원한다. `grep`에서는 **`-E` 옵션**을, `sed`에서는 **`-E` 또는 `-r` 옵션**을 지정하면 ERE를 사용할 수 있다. \([참고](https://www.grymoire.com/Unix/Sed.html#uh-4a)\)\
[이 Unix Stack Exchange 답변](https://unix.stackexchange.com/questions/145402/regex-alternation-or-operator-foobar-in-gnu-or-bsd-sed/145404#145404)에 의하면, `awk`에서는 디폴트로 ERE를 사용한다고 한다.

하지만 오래 전부터 사실상의 표준으로 통용되어 온 정규표현식은 BRE도 ERE도 아닌 **Perl의 regular expression**이다. (여기서 Perl은 PHP나 Python같은 스크립트 언어이다.) Perl 정규식은 ERE보다도 기능이 더 방대하다. 상당수의 프로그래밍 언어가 이 Perl 정규식을 지원한다고는 하는데, 모든 기능을 동일하게 구현한 것은 아마 없을테고 Perl 정규식의 일부 기능만을 구현했을 것으로 생각된다. (원래 방대한 기능을 100% 동일하게 구현한다는 것은 엄청나게 어려운 일이다.)

**Perl compatible regular expression \(PCRE\)**라는 것도 있다. 이름만 들으면 정규식의 한 종류인 것처럼 들리지만, 사실 이건 Perl의 정규식을 다른 곳에서도 쓸 수 있게 구현한 라이브러리이다. 이것 역시 Perl의 정규식과 100% 같지는 않지만, 아파치, PHP같은 대형 프로젝트에서도 이 PCRE 라이브러리를 사용하고 있다고 하니, 꽤 유명하고 널리 사용되는 라이브러리인 듯 하다.

`grep`/`sed`/`awk` 등의 표준 명령행 도구가 ERE(까지)만 지원한다는 것은 매우 아쉬운 일인데, BRE/ERE에서는 ‘긍정적 룩어헤드 / 부정적 룩어헤드’ 같은 lookaround 기능이 지원되지 않기 때문이다. 이를테면, 줄 끝에 있는 특정 패턴을 매치시키는 것은 BRE/ERE로도 가능하지만 **줄 끝에 있지 않은 특정 패턴**을 매치시키는 것은 **부정적 룩어헤드**를 사용해야 하므로 BRE/ERE로는 불가능하다. \([참고](https://nooriro.github.io/230511/very-important-thing-about-sed)\) \([참고2](https://stackoverflow.com/questions/10475015/what-is-a-regex-to-match-a-string-not-at-the-end-of-a-line)\)\
`sed`가 정규표현식만으로 모든 것을 처리하지는 않으므로 어떻게 해서든 대체 방법이 있겠지만, 그렇다고 해서 lookaround를 지원하지 않는 BRE/ERE의 기능이 충분해지는 것은 아니다. 

## `sed`의 첫 실행과 해결할 문제 확인

다시 원래 하던 얘기로 돌아와서, 일단 위 sed 명령
```shell
sed 's/.\{40\}/&\
/g' filename
```

을 4개 파일에 대하여 각각 실행해보자.\
실행해 보면, 4개 중에서 2개만 기대한 대로 실행된다. 각 파일의 처리 결과는 아래와 같다.
```console
$ sed 's/.\{40\}/&\
> /g' 200_w_newline
09908c0b5a22ec99b35467d7aaac50e8b5e0322d
d35404c1602fa63aaf44c1a3bf40942933b41406
1ed87da7f968eba4fdced8ff4894709512ed2f24
70387275aafb21cdd1817557c163927342e50716
1d8305c72b4ba8075ca830e51fd2dda8c3316740

$ sed 's/.\{40\}/&\
> /g' 220_w_newline
f62b844fbde0541e6bf29dcdd313d7396a7161ae
0b9a2afe5fd76557fdaab060d803120baef2ea36
9abb2579e6de10c81803fe83bdf7c84d3d25901d
dec3ea988d5a42ce2443d308080368b3a658eba9
56bf4fcc58d1d8e68564624eca322da797c753d2
0b7beb71e5fc01f0d522
$ sed 's/.\{40\}/&\
> /g' 200_wo_newline
09908c0b5a22ec99b35467d7aaac50e8b5e0322d
d35404c1602fa63aaf44c1a3bf40942933b41406
1ed87da7f968eba4fdced8ff4894709512ed2f24
70387275aafb21cdd1817557c163927342e50716
1d8305c72b4ba8075ca830e51fd2dda8c3316740
$ sed 's/.\{40\}/&\
> /g' 220_wo_newline
f62b844fbde0541e6bf29dcdd313d7396a7161ae
0b9a2afe5fd76557fdaab060d803120baef2ea36
9abb2579e6de10c81803fe83bdf7c84d3d25901d
dec3ea988d5a42ce2443d308080368b3a658eba9
56bf4fcc58d1d8e68564624eca322da797c753d2
0b7beb71e5fc01f0d522$
```

[지난번 글](https://nooriro.github.io/230501/read-n-in-shell)에서 처음에 한 것(`while read -n 40 line` 루프 안에서 `echo $line`을 실행)과는 달리, 이번에는 결과가 규칙적이며 예측했던 대로다.

위 `sed` 명령은 **앞에서부터 40글자마다 개행 문자를 그 끝에 삽입**한다. 따라서,

- `_w_newline` 쪽은 전체 글자수가 40의 배수가 아닌 경우에는 원하는 결과가 나오지만, 전체 글자수가 40의 배수인 경우에는 맨 끝에 (불필요한) 개행 문자가 삽입되어 끝에 불필요한 빈 줄이 생긴다.
- `_wo_newline` 쪽은 반대로 전체 글자수가 40의 배수인 경우에 원하는 결과가 나오고, 전체 글자수가 40의 배수가 아닌 경우에 끝에 줄바꿈이 되지 않는 문제가 생긴다.

넷 중에서 문제가 되는 것은 `200_w_newline`(끝에 개행문자가 중복됨)과 `220_wo_newline`(끝에 개행문자가 없음)이며 이제 이들을 디버깅할 차례다. 하지만 그 전에 **반드시** 짚고 넘어가야 하는 것이 있다.


## `sed`에 대한 매우 중요한 사실

sed script는 여러 개의 명령으로 구성되는데, **입력 파일의 첫 번째 줄에 대하여 명령 1부터 명령 N까지를 연달아서 적용하고 그 결과를 화면(표준출력)에 출력한 다음, 입력 파일의 두 번째 줄을 (같은 식으로) 처리힌다.**\
이 작업 순서를 정확하게 이해하는 것이 매우 중요하다. 명령 1부터 명령 N까지 처리하는 과정이 **안쪽 루프**에 해당하며, 입력 파일의 첫번째 줄부터 마지막 줄까지 처리하는 과정은 **바깥 루프**에 해당한다.

그러므로 `sed`의 작동 방식을 다음과 같이 설명해서는 **안 된다.**
> ~~입력 파일의 **각 줄에 대하여** 명령 1부터 명령 N까지를 연달아 적용한 결과를 출력한다.~~

저 설명이 틀렸다고는 할 수 없다. 하지만 저렇게 설명하면 두 루프의 중첩이 어떻다는 것인지가 모호해진다. 두 루프의 중첩을 반대로 이해하여 입력 파일의 첫번째 줄부터 마지막 줄까지 각 줄마다 명령 1을 적용한 다음, 그 결과에 명령 2를 역시 각 줄마다 적용하는 식으로 작동한다고 오해할 수도 있다. 만일 그렇게 오해하면 `sed`에 대한 거의 모든 것이 이해되지 않을 것이며, `sed`가 그저 어렵게만 느껴질 것이다.


## `sed`에서 `.`은 개행 문자에도 매치된다

또한 인터넷을 검색하다 보면 정규식의 메타문자 `.`이 개행 문자를 **제외한** 모든 문자에 매치된다고 서술된 문서를 간혹 보게 되는데, 이는 **틀린 서술**이다. 적어도 `sed`에서는 그렇다.

1개의 명령으로 구성된 sed script만 다룬다면 저렇게 오해해도 문제가 생기지는 않는다. sed script의 첫 번째 명령이 실행되는 단계에서는 대상 문자열이 개행 문자를 포함하지 않기 때문이다. 하지만 sed script가 두 개 이상의 명령으로 구성될 경우, 첫 번째 명령에 의해 대상 문자열이 개행 문자를 포함하게 될 수 있으므로 `.`이 개행 문자에도 매치되는지 그렇지 않은지를 따지는 것이 중요해진다. 결론만 말하자면 **`.`은 분명히 개행 문자에도 매치된다.**


## `_w_newline` 쪽을 먼저 디버깅해보자

만일 BRE/ERE가 부정적 룩어헤드를 지원한다면, 앞서 실행한 `s` 명령의 정규표현식 `.\{40\}`을 ‘**맨 끝에 있지 않은** 40개 문자’에 매치되도록 수정하기만 하면 된다. 하지만 BRE/ERE는 부정적 룩어헤드를 지원하지 않으므로 이 방법을 쓸 수는 없다.

그렇다고 해서 방법이 없는 것은 아니다. 더 좋은 방법이 있는지는 모르겠지만, 40개 문자 끝에 개행문자를 추가하는 명령 1을 수행한 다음, **맨 끝의 (중복된) 개행 문자를 제거**하는 명령 2를 **추가로 수행**하도록 하면 된다. 전체 `sed` 명령은 아래와 같다.
```shell
sed 's/.\{40\}/&\
/g; s/\n$//' _w_newline
```

앞에서 이야기한 대로 `s` 명령의 형식 `s/REGEXP/REPLACEMENT/[FLAGS]`에서 `REGEXP` 자리에는 개행 문자를 `\n`으로 나타내야 한다. 한편 정규식 `\n$`은 2회 이상 매치될 수 없기 때문에 두 번째 `s` 명령에 `g` 플래그를 줄 필요는 없다.

위 `sed` 명령의 실행 결과는 다음과 같다.

```console
$ sed 's/.\{40\}/&\
> /g; s/\n$//' 200_w_newline
09908c0b5a22ec99b35467d7aaac50e8b5e0322d
d35404c1602fa63aaf44c1a3bf40942933b41406
1ed87da7f968eba4fdced8ff4894709512ed2f24
70387275aafb21cdd1817557c163927342e50716
1d8305c72b4ba8075ca830e51fd2dda8c3316740
$
```
```console
$ sed 's/.\{40\}/&\
> /g; s/\n$//' 220_w_newline
f62b844fbde0541e6bf29dcdd313d7396a7161ae
0b9a2afe5fd76557fdaab060d803120baef2ea36
9abb2579e6de10c81803fe83bdf7c84d3d25901d
dec3ea988d5a42ce2443d308080368b3a658eba9
56bf4fcc58d1d8e68564624eca322da797c753d2
0b7beb71e5fc01f0d522
$
```


## `_wo_newline` 쪽도 디버깅해보자

비슷한 방법으로 `_wo_newline` 쪽도 디버깅할 수 있다.\
앞서 `_w_newline` 쪽은 `s/\n$//`를 추가로 실행하여 끝에 붙은 개행문자를 제거했다. 하지만 이번에는 오히려, 개행문자로 끝나지 않는 경우 끝에 개행문자를 추가해야 한다.

수정된 전체 `sed` 명령은 아래와 같다.
```shell
sed 's/.\{40\}/&\
/g; s/[^\n]$/&\
/' _wo_newline
```

`[^\n]`는 **개행 문자를 제외한 모든 문자**를 뜻한다. (참고로 이같은 정규식의 문법을 [negated character class](https://www.google.com/search?q=negated+character+class&newwindow=1)라고 한다.)\
그러므로 정규식 `[^\n]$`은 맨 끝의 문자가 개행 문자가 아닌 경우 그 문자에 매치되며 위 두 번째 `s` 명령은 그 매치된 문자 끝에 개행 문자를 추가한다. 역시 앞에서 언급했듯이, `s` 명령의 형식 `s/REGEXP/REPLACEMENT/[FLAGS]`에서 `REPLACEMENT` 자리에는 개행 문자를 **`\` 뒤에 글자 그대로의 개행문자**(즉, 실제 줄바꿈)로 나타내야 한다. 

실행 결과는 아래와 같다.
```console
$ sed 's/.\{40\}/&\
> /g; s/[^\n]$/&\
> /' 200_wo_newline
09908c0b5a22ec99b35467d7aaac50e8b5e0322d
d35404c1602fa63aaf44c1a3bf40942933b41406
1ed87da7f968eba4fdced8ff4894709512ed2f24
70387275aafb21cdd1817557c163927342e50716
1d8305c72b4ba8075ca830e51fd2dda8c3316740
$
```

```console
$ sed 's/.\{40\}/&\
> /g; s/[^\n]$/&\
> /' 220_wo_newline
f62b844fbde0541e6bf29dcdd313d7396a7161ae
0b9a2afe5fd76557fdaab060d803120baef2ea36
9abb2579e6de10c81803fe83bdf7c84d3d25901d
dec3ea988d5a42ce2443d308080368b3a658eba9
56bf4fcc58d1d8e68564624eca322da797c753d2
0b7beb71e5fc01f0d522
$
```

## `_wo_newline`을 처리하는 다른 방법

```shell
# 첫 번째 방법
sed 's/.\{40\}/&\
/g; s/[^\n]$/&\
/' _wo_newline

# 두 번째 방법
sed 's/.\{40\}/&\
/g; s/\n$//; s/$/\
/' _wo_newline

# 세 번째 방법
sed 's/.\{40\}/&\
/g; s/\n\{0,1\}$/\
/' _wo_newline

# 네 번째 방법
sed 's/.\{0,40\}/&\
/g' _wo_newline
```

첫 번째 방법은 방금 전에 실행해 본 방법이다.

두 번째 방법은 그 전에 실행해 본 `_w_newline` 파일을 처리하는 방법을 응용한 것이다. 두 번째 `s` 명령으로 맨 끝의 개행 문자를 제거한 다음, 세 번째 `s` 명령으로 맨 끝에 개행 문자를 추가한다.

세 번째 방법은 두 번째 방법을 간소화한 것이다. 정규식 `\n\{0,1\}$`는 맨 끝에 개행문자가 있으면 그 개행문자에 매치되며 그렇지 않으면 맨 끝의 길이가 0인 문자열에 매치되므로 매치된 부분이 개행 문자로 치환되면 두 번째 방법과 같은 결과를 얻는다.

네 번째 방법은 `.`을 고정된 갯수 40개가 아니라 **40개 이하**에 매치시켜 그 뒤에 개행 문자를 삽입하도록 수정한 방법이다. 이렇게 하면 마지막에 남는 글자수가 40개 미만이더라도 그 뒤에 개행문자가 삽입되므로 원하는 결과를 얻게 된다. 이 방법은 **`s` 명령을 한 번만 사용**하므로 넷 중에서 가장 간결한 방법이다.  


## 마무리하며

사실 `sed`를 잘 모르는 상태에서 이 글을 쓰기 시작했고, 처음에 문제를 해결하는 과정에서 상당한 시행착오를 겪었다. 어쨌든 문제는 해결되어 그 시행 착오의 과정을 며칠 동안 공을 들여 장문의 글로 풀어놨는데, 이후 `sed`가 작동하는 방식이 내가 어렴풋이 생각했던 것과는 완전히 다르다는 것을 알게 되어 상당한 충격을 받았다. 이에 대한 것은 아래 링크의 글에서 이야기한 바 대로이다.

- [sed에 대해 이제서야 알게 된 매우 중요한 사실](https://nooriro.github.io/230511/very-important-thing-about-sed)
- [안드로이드 셸에서 xxd 실행하기](https://nooriro.github.io/230513/xxd-on-android)

따라서 글을 대폭 수정하는 것이 불가피해졌다. 그 수정 작업을 열흘 넘게 미뤄 오다가, 지난번에 쓴 장광설을 다 지우고 그 동안 새로 알게 된 내용들을 반영하여 거의 글을 새로 쓰다시피 했다. (2023.05.27-28) 금방 될 줄 알았던 수정 작업은 예상 외로 오래 걸렸으며 쉽지 않았다. 지금 매우 힘들고, 이 글은 여기서 마무리를 지어야겠다.

[Bruce Barnett의 sed 튜토리얼](https://www.grymoire.com/Unix/Sed.html)은 절반 정도 읽고 나서 이후 열흘동안 쳐다도 안 봤는데, 생각난 김에 이것부터 마저 읽어야겠다.
