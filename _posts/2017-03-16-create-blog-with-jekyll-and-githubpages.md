---
layout: post
title: GitHubPages + Jekyll 블로그 만들기
published: false
---

> 무언가를 알게 되었다가 늘 얼마 지나지 않아 잊게 되고 언젠가 다시 찾게 되는 것이 아쉬워 기록이나 남겨두려고 사적인 아카이브를 만든다. 첫 번째 기록으로 이 곳을 만드는 과정을 남긴다.


### GithubPages와 Jekyll

[GitHubPages](https://pages.github.com/)는 Github 저장소의 내용을 웹 페이지로 서비스해주는 기능이다.
무료로 웹 서비스를 호스팅할 수 있지만 고정된 컨텐츠만 보여줄 수 있다.
[Jekyll](https://jekyllrb-ko.github.io/) 이라는 정적 사이트 생성기와 결합되어 컨텐츠를 올리면 자동으로 사이트를 생성하여 서비스할 수 있다.


### 저장소 생성
`<GitHub username>.github.io`라는 빈 repo를 만든다.

[Jekyll Themes](http://jekyllthemes.org/)에서 마음에 드는 테마를 고른다. 나의 경우 [Lanyon](https://github.com/poole/lanyon)을 골랐다.

해당 theme을 다운로드 받는다.

내 repo를 clone한 곳에 다운로드 받은 theme을 풀어놓는다.

jekyll을 설치한다.

```
gem install jekyll

```

빌드하고 실행해본다.

```
jekyll server
```

브라우저에서 `http://localhost:4000`으로 접속해본다.



### 다듬기

`_config.yaml`에서 기본 설정을 수정한다.

```
yaml
```

웹폰트를 추가해서 적용했다.

```
@import
@import
```

archive 페이지를 추가했다.

```
archive.html
```




### 글쓰기

`_post` 폴더 아래에 새로운 마크다운 파일을 만든다. 파일 형식은 다음과 같다.

```
:year-:month-:day-:title.md
```

파일 제일 위에 `layout`과 `title`을 지정해준다. `published: false`를 추가하면 발행을 막을 수 있다.

```
---
layout: post
title: Jekyll + GitHubPages 로 블로그 만들기
---
```


### 발행

로컬 서버에서 정상적으로 글이 발행되는 것이 확인되면 commit하고 GitHub으로 push하면 즉시 사이트가 빌드되면서 반영된다.
문제가 있어서 빌드가 실패할 경우 이메일로 알려준다.

