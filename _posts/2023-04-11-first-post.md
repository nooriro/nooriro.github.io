---
layout: post
title:  "블로그를 만들었다"
date:   2023-04-11 02:00:00 +0900
---
사실 2년 전에 이미, **마크다운\(Markdown\)**으로 글을 작성하는 방식의 블로그를 만들고 싶다는 생각을 해 본 적이 있다.
당시 검색을 통해 **깃허브 페이지\(GitHub Pages\)**와 **지킬\(Jekyll\)**을 이용하면 된다는 것 까지는 쉽게 알아낼 수 있었다. 하지만 지킬을 다루는 방법이 생각보다 복잡하고 불편해 보여서 금방 엄두가 나지 않았다. 그래서 블로그 생각은 일단 접어두고 이후 한동안 잊어버리고 있었다.

그러다가 대략 한 달쯤 전부터 다시 블로그 생각이 절실해졌다. 이유는 많은 사람들이 그렇게 하듯이, 알게 된 것들을 글로 정리해 두었다가 나중에 다시 보고 싶어서이다.

# 원하는 기능

내가 원하는 기능은 대략 다음과 같았다.

1. **깃허브 페이지\(GitHub Pages\)**를 이용
2. **마크다운\(Markdown\)** 문법으로 포스트 작성이 가능해야 함. [GitHub Flavored Markdown][gfm][^gfm1]이 지원되면 좋음.
3. **레이텍\(LaTeX\)** 문법으로 수식을 입력하고 보여줄 수 있어야 함.
4. 깃허브 계정으로 로그인해서 댓글 다는 기능을 붙일 수 있어야 함. 댓글을 작성하고 보여주는 UI 및 기능은 깃허브 Issues의 그것과 비슷해야 함.
5. 블로그 글의 수정 이력을 보여줄 수 있어야 함.

[gfm]: https://github.github.com/gfm/
[^gfm1]: GitHub Docs의 [기본 쓰기 및 서식 지정 구문](https://docs.github.com/ko/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)에 GFM의 주요 문법이 잘 정리되어 있다. \(영문 원문: [Basic writing and formatting syntax](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)\) 

1을 전제로 하면 툴 선택의 폭이 좁아지는데, 굳이 깃허브와의 연동성을 포기하면서까지 선택의 폭을 늘리고 싶지는 않았다. 그래서 2년 전에 미리 찜해둔 **지킬\(Jekyll\)**에 대해 좀더 검색해 보았다.

2와 관련하여, Jekyll은 kramdown이라는 마크다운 프로세서가 GitHub Flavored Markdown을 지원하고 심지어 이게 디폴트 설정[^kramdown1]이다. 또한 원한다면 마크다운 프로세서를 바꿀 수도 있다[^kramdown2]고 한다.\
물론 kramdown이 GitHub Flavored Markdown 문법을 얼마나 잘 지원하는지는 실제로 써 보기 전에 섣불리 말하기 어렵다.[^kramdown3] 하지만 이 정도 서술만으로도 일단 안심이 된다. 

[^kramdown1]: <https://jekyllrb.com/docs/configuration/markdown/#kramdown-processor>
[^kramdown2]: <https://www.markdownguide.org/tools/jekyll/>
[^kramdown3]: 역시 우려했던 대로였다. 당장 이 글을 작성하면서 [깃허브 웹에서 이 마크다운](https://github.com/nooriro/nooriro.github.io/blob/main/_posts/2023-04-11-first-post.md)을 확인해 보았는데, 마크다운을 다루는 방식에 차이가 있었다. 깃허브 쪽에서는 `**...**` 의 `...`부분에 소괄호 `(` `)` `\(` `\)` 중 어느 하나가 포함되어 있으면서 `**...**` 바로 뒤에 조사가 붙는 경우 \(예: `**마크다운\(Markdown\)**으로`\) 볼드 속성이 적용되지 않고 `...` 앞뒤의 `**`가 그대로 표시된다. 반면에 Jekyll에서는 이 경우에도 볼드 속성이 잘 적용된다. Jekyll이 깃허브의 마크다운 파서를 그대로 가져다 쓰지 않는 이상 이런 세부적인 차이는 생기게 마련인데, 구체적으로 어떤 부분이 어떻게 다른지는 이렇게 직접 둘을 비교해 가면서 써 보기 전까지는 알기 어렵다는 것이 문제다.

3은 Jekyll에서 직접 지원하지는 않지만, MathJax를 이용하면 가능하다. 웹에서의 수식 사용성은 10년 전과 지금이 별반 다르지 않은 듯 하여 좀 많이 아쉽다.

4와 5는 지금 당장 필요한 건 아닌데, 찾아봤더니 Jekyll의 플러그인을 이용하면 가능하다고 한다.

이상이 전부 확인되었으니, Jekyll을 설치하지 않을 이유가 있을까?

# 어려웠던 점

작업간에 특히 어려웠던 점은 다음과 같다.

1. 루비(Ruby)를 평소에 쓰지 않아서 \(사실, 나는 Ruby에 대해 전혀 모른다\) Ruby와 관련된 패키지 설치부터 먼저 해야 했는데, 사용중인 리눅스 배포판\(Ubuntu 20.04 on WSL\)에서 `apt` 명령으로 기본 설치되는 루비 관련 패키지\(무슨 패키지였는지는 오래되어 기억나지 않음\)의 버전이 낮아서 이를 해결하는 데 시간이 좀 걸렸다.

2. MathJax를 붙여서 간단한 테스트를 해 보았는데 처음에는 인라인 수식\(이를테면 $\zeta(2) = \sum_{n=1}^{\infty} \frac{1}{n^2} = \frac{\pi^2}{6}$ 처럼 문장 중간에 삽입된 수식\)이 표시되지 않아서 당황했다.\
그리고 슬프게도 MathJax 3에서는 인라인 수식의 자동 줄바꿈\(auto wrap 혹은 auto linebreak\)이 지원되지 않는다. 이 경우 수식의 길이가 문단 폭보다 길어지면 레이아웃이 깨지기 때문에, 수식이 화면의 폭보다 길어지면 밑에 스크롤바가 생기도록 하여 레이아웃이 깨지지 않도록 했다.\
이를테면 다음 수식 $\int \ln x \, dx = \int 1 \cdot \ln x \, dx = x \cdot \ln x - \int x \cdot \frac{1}{x} \, dx =  x \ln x - x + C $는 모바일에서 한 줄에 다 표시되지 않지만 레이아웃이 깨지지 않고 수식 밑에 스크롤바가 생길 것이다. 

3. 기본으로 제공되는 minima 테마를 수정하려고 방법을 구글에서 검색해 봤는데, 한글로 쓰여진 어느 블로그 글에서는 무슨무슨 파일을 열어서 수정하면 된다고 나와 있었다. 그래서 그 파일을 찾아 봤는데, 없어서 당황했다. 뿐만 아니라, 사실 테마 관련된 파일이 내 저장소에 하나도 없었다.\
테마 파일이 내 사이트에 전혀 없는데 어떻게 테마가 적용된 것일까? 알고 봤더니, minima같은 gem-based theme는 테마 파일 전부를 내 사이트에 복사해놓고 그걸 수정하는 게 아니라, 테마 원본 파일 중에서 수정하려는 파일만 내 사이트 디렉토리로 복사해와서 수정하면 된다고 한다.[^theme1][^theme2]\
테마 원본 파일의 경로를 확인하려면 `bundle info --path THEME` \(minima 테마의 경우, `bundle info --path minima`\) 명령을 실행하면 된다.[^theme2]\
아울러 gem-based theme는 `bundle update THEME` \(minima 테마의 경우 `bundle update minima`\) 명령으로 업데이트가 가능하다.[^theme1]

[^theme1]: <https://jekyllrb.com/docs/themes/#understanding-gem-based-themes>
[^theme2]: <https://jekyllrb.com/docs/themes/#overriding-theme-defaults>

(이하 작성중)

# 각주
