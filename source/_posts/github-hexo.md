---
title: Github Pages + Hexo
date: 2018-02-02 14:13:10
tags: [github, hexo]
cover: /images/github_hexo_blog.png
categories: [tip]
subtitle: 나만의 블러그를 만들어보자.
---
# --- GitHub Pages 만들기 ---
참고 : https://pages.github.com/

### 1) Github Repository 생성
1. New repository 버튼 클릭
2. Repository name 입력 : arcjjang.github.io
3. Public 선택
4. Create repository 버튼 클릭

### 2) 터미널에서 Clone the repository
``` bash
$ git clone https://github.com/arcjjang/arcjjang.github.io.git
```

### 3) Hello World
``` bash
$ cd arcjjang.github.io
$ echo "Hello World" > index.html
```

### 4) Push it
``` bash
$ git add --all
$ git commit -m "Initial commit"
$ git push -u origin master
```

### 5) go to *https://arcjjang.github.io*

# --- Hexo 적용하기 ---
참고 : https://hexo.io/ko/index.html

### 1) Hexo 설치 및 생성
``` bash
$ npm install hexo-cli -g
$ hexo init blog
$ cd blog
$ npm install
$ npm install hexo-deployer-git --save <-- Git으로 배포하는 경우 설치
$ hexo server
```

### 2) Hexo 환경파일 수정
/_config.yml

#### 2-1) Site 설정
``` yml
# Site
title: Coder`s Frontline
subtitle: For my poor memory...
description: 나만을 위한, 나의 불쌍한(?) 기억력을 위한 지식 저장소
author: arcjjang
language:
timezone:
```

#### 2-2) URL 설정
``` yml
# URL
url: https://arcjjang.github.io
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:
```

#### 2-3) 배포 설정
``` yml
# Deployment
deploy:
  type: git
  repo: https://github.com/arcjjang/arcjjang.github.io.git
```

### 3) 로컬에서 서버 실행하기
``` yml
$ hexo server
```
go to http://localhost:4000/

### 4) 배포하기
``` yml
$ hexo clean
$ hexo deploy --generate
```

※ windows 환경에서 deploy중에 username error가 나면 윈도우자격증명을 다시 설정
``` bash
$ git config --global credential.helper wincred
```

# --- 테마 적용하기 ---
테마 종류 : https://hexo.io/themes/

### 1) 테마 설치
``` yml
$ cd themes
$ git clone https://github.com/klugjo/hexo-theme-clean-blog.git clean-blog
```

### 2) 테마 설정하기
> /clean-blog/_config.yml

``` yml
# Header
menu:
  Home: /
  Archives: /archives
  Tags: /tags
  Categories: /categories
  Github:
    url: https://github.com/arcjjang
    icon: github

# Title on top left of menu. Leave empty to use main blog title
menu_title:

# URL of the Home page image
index_cover: /images/topimage.jpg

# Social Accounts
twitter_url:
twitter_handle:
facebook_url:
github_url: https://github.com/arcjjang
gitlab_url:
linkedin_url:
mailto: arcelias@naver.com
```

### 3) Tags 페이지 만들기
``` bash
$ cd ..
$ hexo new page "tags"
```

/source/tags/index.md 파일 수정
```
---
title: All tags
type: "tags"
---
```

### 4) Categories 페이지 만들기
``` bash
$ hexo new page "categories"
```

/source/categories/index.md 파일 수정
```
---
title: All categories
type: "categories"
---
```

### 5) 테마 선택하기
> /_config.yml 파일

```
theme: clean-blog
```

# --- Post 쓰기 ---
``` bash
$ hexo new post 포스트명
```

/source/posts/포스트명.md 파일 수정
```
---
title: Github Pages + Hexo
date: 2018-02-02 14:13:10
tags: [github, hexo]
cover: # post cover image
categories: [blog]
subtitle: 나만의 블러그를 만들어보자.
---
```

참고 : https://guides.github.com/features/mastering-markdown/