---
title: 지킬 블로그에 글쓰기
subtitle: 기술블로그
layout: post
author : seha
avatar: "assets/img/portfolio/cake.png"
categories : [Etc]
tags: [Etc]
comment : true
---
### 목차
* TOC
{:toc}

--- 

## 로컬서버에 띄울 수 있는 환경 만들기
우선 헤렌 블로그를 clone 합니다. [[링크]](https://github.com/herrenofficial/herrenofficial.github.io)

1. 루비 설치
    * https://rubyinstaller.org/downloads/
    * 위 사이트에서 WITH DEVKIT 으로 다운받습니다.
    * 
     ![](https://i.imgur.com/TL90A8l.png)
    * `Ruby Installer` 에서 반드시 **3번**(MSYS2 and MINGW development toolchain)을 선택해 설치합니다.

2. 지킬 및 번들러 설치

* ![](https://i.imgur.com/oNM4RIV.png)
설치된 ruby 프롬프트를 실행합니다.
    * `gem install bundler`   
    * `gem install jekyll`
    * `gem install rake`
    * `bundler install`
3. 로컬 서버 실행 
```
bundle exec jekyll serve
```
일반적으로 4000포트로 열립니다.



## 파일 생성 및 제목 지정

* 포스팅 기본 문법은 마크다운 입니다. [[마크다운 문법 정리]](https://gist.github.com/ihoneymon/652be052a0727ad59601)
* `_post` 폴더에 `.md` 확장자인 파일을 작성합니다. 
* 포스트 파일의 형식은 `yyyy-MM-dd-제목` 이며, 공백은 하이픈(-)으로 대체해야합니다.

예시)
```
2019-01-01-제목-입니다.md
```
위와 같은 파일의 제목은 2019년 1월1일에 작성된 것으로 간주됩니다.
### 팁
```
* TOC
{:toc}
```
위의 구문은 자동으로 목차를 생성해줍니다.
**줄바꿈**은 이전 라인의 마지막 문장끝에 **공백 두개**를 추가합니다.

## 포스트 헤더 작성
포스트의 헤더는 다음과 같으며 [[Front Matter]](https://jekyllrb.com/docs/front-matter/) 라고 불립니다.
다음과 같이 사용할 수 있습니다.
```
---
title: 타이틀 입니다.
subtitle: 부제목 블라블라
layout: post
author : seha
avatar : "assets/img/blogpost/oss_0.jpg" // 작성자 아바타입니다.
feature-img: "assets/img/blogpost/oss_0.jpg" //포스트 타이틀에 노출될 이미지입니다.
categories : [Seminar] 
tags: [seminar]
comments: true
---
```
카테고리와 태그는 콤마로 구분합니다.

## 카테고리 생성
`category` 폴더에 만들으려 하는 카테고리 명으로 `md` 파일을 생성합니다.
생성한 `md` 파일의 내용으로 
```
---
layout: categories
title: 생성할 카테고리명
---
```
위와같이 작성합니다.

