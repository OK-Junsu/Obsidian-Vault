---
title: "[SQLite 공부하기] SQLite 설치하고 DB 만들기"
source: "https://m.blog.naver.com/rickman2/222850909657"
author:
  - "[[네이버 블로그 | Data Miner]]"
published:
created: 2025-05-16
description: "0. SQLite 설치 지난 포스트에서는 SQLite를 소개한 적이 있습니다. 이번 포스트에서는 SQLite를 ..."
tags:
  - "clippings"
---
[본문 바로가기](https://m.blog.naver.com/rickman2/#ct)

## 블로그

## 카테고리 이동

[SQL/검색엔진(DB)](https://m.blog.naver.com/PostList.naver?blogId=rickman2&categoryNo=33&logCode=0&categoryName=SQL%2F%EA%B2%80%EC%83%89%EC%97%94%EC%A7%84%28DB%29#postlist_block)

\[공지\] \[SQLite 공부하기\] SQLite 설치하고 DB 만들기

[![프로필](https://blogpfthumb-phinf.pstatic.net/20190323_139/rickman2_1553329746348hcuUM_JPEG/20140929_1136332.jpg?type=s1)](https://m.blog.naver.com/PostList.naver?blogId=rickman2)

[**rickman2**](https://m.blog.naver.com/PostList.naver?blogId=rickman2)

2022\. 8. 17. 17:47

**0\. SQLite 설치**

환경은 윈도우 환경에서 설치를 해보도록 하겠습니다. 설치 전에 먼저 알아두어야 할 점은 SQLite는 별도의 설치 파일이 존재하지 않다는 점입니다.

다운로드 링크

[https://www.sqlite.org/download.html](https://www.sqlite.org/download.html)[**SQLite Download Page**

Small. Fast. Reliable. Choose any three. Home About Documentation Download License Support Purchase Search SQLite Download Page Source Code sqlite-amalgamation-3390200.zip (2.44 MiB) C source code as an amalgamation, version 3.39.2. (sha3: deb2abef617b6305525e3b1a2b39a5dc095ffb62f243b4d1b468ba5f419...

www.sqlite.org

](https://www.sqlite.org/download.html)

다운로드 링크 화면을 보면, 아래와 같이 setup 파일이나 msi와 같은 파일을 찾을 수 없습니다. 그러므로 설치를 위해서는 Precompiled Binaries를 받아서 환경변수 설정을 통해 설치를 해야합니다. 처음에는 복잡해 보이지만, 천천히 따라 하다보면 크게 어렵지 않습니다.

그럼, 설치를 진행해 보겠습니다. 먼저, 아래 그림과 같이 Precompiled Binaries For windows 영역에서 파란색으로 표시된 파일을 다운로드 받습니다. 본 포스트에서는 64bit를 기준이기 때문에 해당 DLL 파일을 다운로드 받았습니다. 만약 여러분이 32bit 설치를 원하신다면, 위에 있는 32bit 파일을 다운로드 받으시면 됩니다.

두 번째 다운로드 받아야 하는 파일은 SQLite Tool입니다. 이 파일은 Command Line을 통해 SQLite DB를 조작할 때 사용하는 도구입니다. 예를 들어 DB를 생성한다거나 DB에 접속해서 쿼리를 실행할 때 이용됩니다.

해당 파일을 모두 다운로드 받았다면, 원하시는 위치에 해당 파일을 옮겨 놓습니다. 본 예시에서는 D:/SQLite 위치에 파일을 옮겨 저장하였습니다.

**1\. SQLite 시스템 환경 설정하기**

해당 파일을 Command Line을 통해 실행하기 위해서는 환경변수(시스템 환경변수)에 등록이 필요합니다. 이 과정은 일반적인 환경변수 등록과 큰 차이는 없습니다.

윈도우 아이콘을 클릭해서 아래와 같이 '시스템 환경변수 편집'에 들어갑니다. 시스템 속성 > 환경변수를 클립합니다.

해당 화면에서 "시스템 변수"의 " Path"를 편집해 아래와 같이 SQLite 폴더를 등록시키고, PC를 한번 재부팅 시켜 줍니다.

여기까지 완료되었다면, SQLite를 설치는 완료되었습니다. 설치가 정상적으로 되었는지 확인을 해보겠습니다.

윈도우 아이콘을 선택한 후에 cmd를 입력한 후 명령프롬프트를 선택합니다. 그러면 아래와 같이 Command Line Interface(명령프롬프트)가 나타납니다.

**2\. SQLite DB 데이터베이스 만들기**

이 절에서는 SQLiite의 데이터베이스를 만들어 보겠습니다. 앞서 알 수 있듯 데이터베이스 생성 명령어는 ".open" 입니다. 물론 이 명령어는 데이터베이스에 연결에 사용되나 해당 파일이 실행위치에 존재하지 않을 때는 데이터베이스를 생성합니다.

저는 Command Line Interface에서 위치를 "D:/SQLite"로 변경하였고, 여기서 아래의 명령어를 입력해 데이터베이스를 생성하였습니다.

.open databaseTest.sqlite

해당 위치에 정상적으로 파일이 생성된 것을 확인해 볼 수 있습니다.

**3\. HeidiSQL 연결**

HeidiSQL은 공개용 데이터베이스 SQL IDE로,. 최근 SQLite도 지원을 하게 되었습니다. 이 도구를 이용하 경우 쿼리를 작성하거나 테스트 하는데 큰 도움이 됩니다. 마지막으로 heidiSQL에서 앞에서 만든 데이터베이스를(databaseTes)t 파일을 연결해 보겠습니다.

\* HeidiSQL 설치는 다음포스트에서 다시 설명하다록 하겠습니다.

세션 관리자에서 신규를 선택합니다. 아래의 네트워크 유형에서 SQLite를 선택합니다.

아래와 같이 앞서 생성한 "" 파일을 선택학고, 세션관리자에서 열기를 선택합니다.

접속이 완료되면 아래와 같은 화면이 나타납니다.

4\. 맺음말

이번 포스트에서 SQLite를 설치하는 방법, SQLite 데이터베이스 만드는 방법, HeidiSQL에 연결하는 방법에 대해서 알아보았습니다. SQLite는 MS Access보다 다양한 확장성을 가지고 있습니다. 예전에는 Excel의 제한사항을 MDB나 ACCESS를 통해서 처리해왔었는데요, SQLite를 이용하게 되면 훨씬 더 편리하게 다양한 응용을 개발할 수 있습니다.

이후 다른 설명에서 종종 이용할 예정이니 여러분들도 시간이 나실 때 설치해보시면 좋을 것 같습니다. 그럼 오늘도 즐거운 하루되시고, 계속적으로 제 블로그에 많은 관심 부탁드립니다.

여러분의 구독과 좋아요는 포스트를 작성하는데 많은 도움이 됩니다

[*0*](https://m.blog.naver.com/rickman2/#)

[댓글 *9*](https://m.blog.naver.com/CommentList.naver?blogId=rickman2&logNo=222850909657) 공유하기