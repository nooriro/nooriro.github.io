---
layout: post
title:  "sed에 대해 이제서야 알게 된 매우 중요한 사실"
date:   2023-05-11 23:00:00 +0900
---

[sed를 이용하여 일정한 길이마다 줄바꿈하여 출력하기](https://nooriro.github.io/230506/using-sed-to-insert-newline-every-n-chars)를 쓰고나서 [Sed - An Introduction and Tutorial by Bruce Barnett](https://www.grymoire.com/Unix/Sed.html)을 읽다가 알게 된 충격적인 사실을 기록해둔다.

[Delete with d](https://www.grymoire.com/Unix/Sed.html#uh-30) 섹션의 맨 마지막에 있는 내용이다.

> This demonstrates the pattern space sed uses to operate on a line. The actual operation sed uses is:
> 
> - Copy the input line into the pattern space.
> - Apply the first
> - sed command on the pattern space, if the address restriction is true.
> - Repeat with the next sed expression, again
> - operating on the pattern space.
> - When the last operation is performed, write out the pattern space
> - and read in the next line from the input file.

나는 이 부분을 읽기 전까지, 지난 몇 년 동안, `sed`의 여러 명령을 `;`으로 연결하여 실행할 경우 (혹은 명령들을 각각 `-e` 옵션으로 나열하여 실행할 경우) N번째 명령이 파일 전체에 대해 실행되고 나서 N+1번째 명령이 실행된다고 생각하고 있었다. 하지만 **그게 아니었다.**

저 부분을 읽었을 때 머리를 망치로 한 대 얻어맞은 기분이었다. 교차 검증을 하려고 인터넷에서 검색을 좀 해 봤는데, `sed`의 메커니즘을 직접 다룬 글은 아직 찾지 못했지만 스택오버플로우의 아래 두 링크를 보아하니 역시 저 내용이 틀리지 않은 것 같다.

- [the 'd' command in the sed utility](https://stackoverflow.com/questions/8226439/the-d-command-in-the-sed-utility)
- [The Concept of 'Hold space' and 'Pattern space' in sed](https://stackoverflow.com/questions/12833714/the-concept-of-hold-space-and-pattern-space-in-sed)

위에서 인용한 내용에 따르면, `sed`는 주어진 명령 1부터 명령 N까지를 연달아서 실행하되, 이 과정을 **각 줄마다 따로 따로 수행**한다.\
입력 파일의 첫 번째 줄부터 마지막 줄까지, 각 줄에 대해 주어진 명령들을 (그 줄에만) 순서대로 연달아 실행하고 (그 줄의) 실행 결과를 출력(단, `sed`에  `-n` 옵션을 주어 실행하면 이 출력 과정은 생략된다)하기를 반복하는 방식으로 `sed`는 작동한다.

의사 코드로 이 과정을 **대략** 표현해 보자면 다음과 같다. (address를 처리하는 부분은 완벽하지 않고, 중괄호로 묶은 블럭의 처리는 고려하지 않았다.)

```java
for (int i = 0; i < lines.length; i++) {
  StringBuilder patternSpace = new StringBuilder(lines[i]);
  boolean skipPrint = hasOption("n");
  for (int j = 0; j < commands.length; j++) {
    if (addresses[j].notContains(i)) continue;
    if (commands[j].equals("d")) { skipPrint = true; break; }
    apply(commands[j], parternSpace);
  }
  if (!skipPrint) System.out.println(patternSpace.toString());
}
```

이러한 `sed`의 작동 방식을 알고 나니, 그 동안 전혀 이해되지 않았던 많은 것들이 한번에 이해되기 시작했다.

`sed`의 **pattern space**가 버퍼라는 것은 예전부터 알고 있었지만, 거기에 무엇이 저장되며 그것이 구체적으로 어떻게 사용되는지는 전혀 알지 못했다. 하지만 이제는 다음과 같이 자신있게 말할 수 있다. 패턴 스페이스란 **현재 처리중인 줄의 상태를 담고 있는 버퍼**이다. 입력 파일의 첫 번째 줄부터 마지막 줄까지 순서대로, 각 줄에 대해 아래 과정이 수행된다.

- 입력 파일에 있는 해당 줄의 내용을 패턴 스페이스로 복사한 다음, 명령 1부터 명령 N까지를 패턴 스페이스에 순서대로 적용한다. (만일 명령 앞에 address가 지정되어 있다면 우선 현재 처리중인 줄이 그 address에 포함되는지를 먼저 판단하고 그 경우에만 명령을 실행한다.)
- 모든 명령의 실행이 끝나면 패턴 스페이스의 내용을 출력하고 다음 줄로 넘어간다. (단, `sed`에 `-n`옵션을 지정하여 실행한 경우에는 이 디폴트 출력을 수행하지 않고 다음 줄로 넘어간다.)

`d` 명령과 `p` 명령이 하는 일 또한 완전히 새로 이해하게 되었다.

- **`d` 명령**을 단순히 줄을 지우는 명령으로만 알고 있었는데, 이렇게 이해하는 것은 너무 피상적이며 `d` 명령에 대해 제대로 이해하는 것을 방해한다. `d` 명령의 진짜 의미는 **그 줄에 대한 이후 처리를** 디폴트 프린트까지 포함해서 **전부 skip하고 다음 줄을 처리하라**는 것이다.
디폴트 프린트까지 포함해서 해당 줄의 이후 처리가 전부 skip되므로 **결과적으로** 출력에서 그 줄이 빠지게 되는 것이다.
  - 이를 의사 코드로 나타내자면 입력 파일의 각 줄에 대한 바깥쪽(`i`) 루프 안에서 `continue;`를 실행하는 것과 동일한데, 실제로 명령들의 실행은 안쪽(`j`) 루프에서 이루어지므로 같은 결과를 얻으려면 플래그 변수와 `break`를 이용해야 한다. 위 의사 코드에서는 `d` 명령을 만났을 때 불리언 변수 `skipPrint`를 `true`로 설정하고 `break;`로 안쪽(`j`) 루프를 빠져나오도록 했다. 이후 디폴트 프린트는 건너뛰게 되며 다음 줄의 처리로 넘어간다. 이렇게 코드를 짜야 할 때마다 C-like 언어들의 문법에 `break n` 명령과 `continue n` 명령이 없다는 것이 너무나 아쉽게 느껴진다.

- **`p` 명령** 역시 그 줄을 프린트하라는 명령인 것은 알고 있었지만, 이를테면 `sed 'p' filename`과 같은 명령 실행시 같은 내용이 **왜 연달아서** 두 번씩 출력되는지 그 이유에 대해서는 자신있게 말할 수 없었다. 하지만 이제는 분명해졌다. 같은 내용이 연달아서 출력되는 이유는 다른 게 아니라, `p` 명령이 **실행되는 시점에 실제로 pattern space의 내용이 출력**되기 때문이다.\
`sed 'p' filename`의 경우 각 줄에 대하여, `p` 명령이 실행될 때 pattern space의 내용이 (한 번) 출력되고, 모든 명령(여기서는 `p` 명령 뿐이지만)이 실행된 이후에 pattern space의 내용이 (다시 한 번) 출력되므로 결국 같은 내용이 연달아서 두 번씩 출력되는 것이다.

이 사실을 알게 됨에 따라 지난 번에 쓴 [sed를 이용하여 일정한 길이마다 줄바꿈하여 출력하기](https://nooriro.github.io/230506/using-sed-to-insert-newline-every-n-chars)에 풀어놓은 장광설은 대규모 수정이 불가피해졌다. (단, 그 글에서 제시한 최종 명령이 기대한 대로 정확하게 작동한다는 점은 변함이 없다.) 아마 뒷부분 절반 혹은 그 이상을 날리고 새로 써야 할 것 같은데, 설명에 필요한 `sed`의 기본적 작동원리는 이 글에서 이미 다 설명했기 때문에 많은 분량을 들어내더라도 약간의 설명만으로 충분할 것 같다. 먼 길을 돌고 돌아 왔지만 결과적으로 글이 훨씬 간결해질 듯 하여 다행이다.

글을 고치면서 `_w_newline` 파일들과 `_wo_newline` 파일들을 모두 처리할 수 있는 통합 `sed` 명령을 제시하는 것도 생각해 봤는데, 핵심이 되는 `sed '$a\'` 명령이 `busybox` 환경에서 기대대로 작동하지 않아서 (단, 우분투와 안드로이드의 기본 셸 환경에서는 정상 작동한다) 일단 그건 하지 않기로 했다. 나중에 그 주제만 가지고 새 글을 쓰는 것이 더 좋을 것 같다.\
참고로 `sed '$a\'` 명령은 (일부 셸 환경에서) 개행 문자로 끝나지 않는 파일 끝에만 개행 문자를 추가하는 것이 기대 동작이며, 이 명령의 출처는 Unix Stack Exchange의 아래 링크이다.
- [How to add a newline to the end of a file?](https://unix.stackexchange.com/questions/31947/how-to-add-a-newline-to-the-end-of-a-file)

하지만 지난번 글에서 이 부분은 꼭 같이 수정해야 할 것 같다. 여러 정규식의 종류(BRE, ERE, P(C)RE)를 이야기하면서 BRE/ERE만 잘 익혀서 활용해도 충분할 것 같다고 이야기했는데, 글을 쓴 이후에 '긍정적 룩어헤드 / 부정적 룩어헤드' 같은 lookaround 기능들이 BRE/ERE에서는 지원되지 않는다는 것을 확인하게 되었다. 이를테면, 줄 끝에 있는 특정 패턴을 매치시키는 것은 BRE/ERE로도 가능하지만 **줄 끝에 있지 않은 특정 패턴**을 매치시키는 것은 **부정적 룩어헤드**를 사용해야 하므로 BRE/ERE로는 불가능하다. (아래 링크들 참고)

- [What is a regex to match a string NOT at the end of a line?](https://stackoverflow.com/questions/10475015/what-is-a-regex-to-match-a-string-not-at-the-end-of-a-line)
- [POSIX BRE syntax for a negative look ahead/not followed by](https://stackoverflow.com/questions/43665626/posix-bre-syntax-for-a-negative-look-ahead-not-followed-by)
- [BASH: How to use Regex Negative Lookahead in sed command for a string?](https://stackoverflow.com/questions/54067485/bash-how-to-use-regex-negative-lookahead-in-sed-command-for-a-string)
- [Does lookbehind work in sed?](https://stackoverflow.com/questions/26110266/does-lookbehind-work-in-sed)

`sed`가 정규표현식만으로 모든 것을 처리하지는 않으므로 어떻게 해서든 대체 방법이 있겠지만, 그렇다고 해서 lookaround를 지원하지 않는 BRE/ERE의 기능이 충분해지는 것은 아니다. 그러므로 해당 부분의 서술은 글을 수정하면서 같이 수정할 생각이다.
