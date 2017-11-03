---
layout: post
title: bash on ubuntu on windows 10 폰트와 컬러 변경
---

![](https://i.imgur.com/SphGNdl.png)


windows 10에서 bash를 깔고 나서 폰트와 컬러를 바꾸고 싶었으나 아무리 바꿔도 저장이 되지 않고 다시 켜면 원래 굴림체로 돌아가는 현상이 있었다.

bash를 실행하는 방법에 따라서 폰트나 컬러 설정을 가져오는 레지스트리 위치가 다른데 Win + R > bash 로 켤 경우 `HKEY_USERS\S-1-5-21-555200176-2817983154-709514845-27452\Console\%SystemRoot%_system32_bash.exe` 아래에서 설정을 가져온다.

하지만 bash.exe에 문제가 있어서 codepage가 한글(CP949)로 되어 있을 경우 폰트/컬러 설정이 저장되지 않는다.(관련 github issue https://github.com/Microsoft/WSL/issues/757)

레지스트리 위 경로에 "CodePage" 이름으로 dword 를 하나 만들고 `65001`을 입력해 bash의 codepage설정을 UTF-8로 바꾸어주면 그 후로 변경하는 폰트와 컬러 설정이 정상적으로 저장된다.

CodePage 설정과 함께 Ubuntu Mono폰트, Ubuntu bash 컬러를 셋팅한 레지스트리는 다음과 같다.


```
Windows Registry Editor Version 5.00
[HKEY_USERS\S-1-5-21-555200176-2817983154-709514845-27452\Console\%SystemRoot%_system32_bash.exe]
"InsertMode"=dword:00000000
"ScreenBufferSize"=dword:23290075
"WindowSize"=dword:00380075
"FaceName"="Ubuntu Mono"
"WindowAlpha"=dword:000000ff
"ColorTable00"=dword:00240a30
"ColorTable01"=dword:00246534
"ColorTable02"=dword:00069a4e
"FontSize"=dword:00120000
"ColorTable03"=dword:00809806
"ColorTable04"=dword:000000cc
"ColorTable05"=dword:007b5075
"ColorTable06"=dword:0000a0c4
"ColorTable07"=dword:00cfd7d3
"ColorTable08"=dword:00535755
"ColorTable09"=dword:00cf9f72
"ColorTable10"=dword:0034e28a
"ColorTable11"=dword:00e2e234
"ColorTable12"=dword:002929ef
"ColorTable13"=dword:00a87fad
"ColorTable14"=dword:004fe9fc
"ColorTable15"=dword:00eeeeee
"CodePage"=dword:0000fde9
```

