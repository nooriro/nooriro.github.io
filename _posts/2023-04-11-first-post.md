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
물론 kramdown이 GitHub Flavored Markdown 문법을 얼마나 잘 지원하는지는 실제로 써 보기 전에 섣불리 말하기 어렵다. 하지만 이 정도 서술만으로도 일단 안심이 된다. 

[^kramdown1]: <https://jekyllrb.com/docs/configuration/markdown/#kramdown-processor>
[^kramdown2]: <https://www.markdownguide.org/tools/jekyll/>

3은 Jekyll에서 직접 지원하지는 않지만, MathJax를 이용하면 가능하다. 웹에서의 수식 사용성은 10년 전과 지금이 별반 다르지 않은 듯 하여 좀 많이 아쉽다.

4와 5는 지금 당장 필요한 건 아닌데, 찾아봤더니 Jekyll의 플러그인을 이용하면 가능하다고 한다.

이상이 전부 확인되었으니, Jekyll을 설치하지 않을 이유가 있을까?

# 어려웠던 점

작업간에 어려웠던 점은 다음과 같다. (이하 작성중)




