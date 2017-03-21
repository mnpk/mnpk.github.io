---
layout: post
title: GitHubPages + Jekyll 블로그 만들기
---

> 오늘 배우지만 내일 잊고 기억을 더듬을 모레를 위해 사적인 아카이브를 만든다. 첫 번째 기록으로 이 곳을 만드는 과정을 남긴다.


## GithubPages와 Jekyll

[GitHubPages](https://pages.github.com/)는 Github 저장소의 내용을 웹 페이지로 서비스해주는 기능이다.
무료로 웹 서비스를 호스팅할 수 있지만 고정된 컨텐츠만 보여줄 수 있다.
[Jekyll](https://jekyllrb-ko.github.io/) 이라는 정적 사이트 생성기와 결합되어 컨텐츠를 올리면 자동으로 사이트를 생성하여 서비스할 수 있다.


## 저장소 생성
GitHub에서 `<username>.github.io` 이름으로 빈 repo를 만든다.

[Jekyll Themes](http://jekyllthemes.org/)에서 마음에 드는 테마를 골라 다운로드 받는다. 나의 경우 [Lanyon](https://github.com/poole/lanyon)을 골랐다.

내 repo를 clone한 곳에 다운로드 받은 theme을 그대로 풀어놓는다.

jekyll을 설치하고

```
gem install jekyll

```

빌드, 실행

```
jekyll server
```

그리고, `http://localhost:4000`으로 접속해본다.



## 다듬기

`_config.yaml`에서 기본 설정을 수정한다.

```yaml
# permalink 형태를 지정. 자세한 내용은 Jekyll 문서 https://jekyllrb.com/docs/permalinks/ 참조.
permalink:           pretty

# Setup
title:               mp
tagline:             'archive'
description:         ''
url:                 http://mnpk.github.io
# baseurl = URL 이하에 고정으로 붙는 base url이 있을 경우 지정
baseurl:             ''
paginate:            5

# About/contact
author:
  name:              mnpk
  url:               https://mnpk.github.io/
  email:             mnncat@gmail.com

# Custom vars
version:             1.0.0

# gem required
gems: [jekyll-paginate]
```

테마에 따라 다르지만 단순한 테마의 경우 몇 개의 짧은 html 파일들과 몇 개의 css 파일들로 이루어져 있다.
입맛에 따라 html파일과 css파일을 수정해본다.
나의 경우 불필요해 보이는 부분을 지우고 폰트 크기를 조절하고 내가 좋아하는 웹폰트를 추가해서 적용했다.

```css
...
@import url(https://fonts.googleapis.com/css?family=Merriweather:400,400italic,700);
@import url(https://fonts.googleapis.com/earlyaccess/nanummyeongjo.css);

...
```

그리고, 글 목록만 요약된 페이지가 있으면 좋을 것 같아서 archive 페이지를 추가했다.


## 글쓰기
새 글을 쓰려면 `_post` 폴더 아래에 새로운 markdown 파일을 만든다. 이름은 `:year-:month-:day-:title.md` 형태로 한다.

파일을 열고 제일 앞에 `layout`과 `title`을 쓴다.
```
---
layout: post
title: Jekyll + GitHubPages 로 블로그 만들기
---

그리고 본문을 써내려간다.
...
```

markdown 문법은 [여기](https://guides.github.com/features/mastering-markdown/)를 참조한다.



## 발행

commit하고 GitHub으로 push하면 즉시 사이트가 빌드된다. 만약 문제가 있어서 빌드가 실패할 경우 이메일로 알려준다.

commit과 push는 하고 싶지만 글을 발행하고 싶지 않다면 글 상단에 `published: false`를 지정한다.

