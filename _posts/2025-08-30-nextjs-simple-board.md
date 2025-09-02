---
layout: post
title: 'Next.js + Auth.js 게시판 만들기'
date: 2025-08-30 21:40:00 +0900

tags: [fullstack, next.js, auth.js, javescript, react, tailwind, supabase, oauth2, social login]
---

새로운 분야를 공부하는 가장 좋은 방법은 뭔가를 만들면서 계속 씨름 해 보는것이라고 생각한다.  
최근 웹 개발은 어떤식인지 대략이라도 알기 위해
Next.js 란 것으로 기본적인 게시판을 만드는 것에 도전해 보았다.  
{: .notice--primary}

Next.js v15 app router + Auth.js + tailwind + Supabase 를 이용해서 만들었고

- 데모 사이트  
[https://nextjs-simpleboard.vercel.app](https://nextjs-simpleboard.vercel.app){:target="_blank"}

- 소스코드  
[https://github.com/jeremyko/nextjs-simpleboard](https://github.com/jeremyko/nextjs-simpleboard){:target="_blank"}

### 사용된 패키지들

현 시점에서 최신 버전들을 사용했다.

    next:15.4.3
    next-auth:5.0.0-beta.29
    react:19.1.0

### 기능

next.js server action 과 react server component 를 사용해서 다음을 구현했다.

- social login
- 댓글 및 대댓글 기능
- pagination
- 검색
- 반응형 디자인
- 조회수 관리 기능 (남용 방지 위한 쿠키 사용)
- refresh token rotation 처리(oauth 로그인 access token 갱신 위한)

### 후기

- 알아야 할 종류가 많다. js, typescript, css, tailwind, react, nextjs, auth.js, shadcn, more...
- 그걸 대충이라도 알아야 개발을 할수 있으니 이번에 한번씩 문서를 다 읽긴 했다
- next.js 는 page, app router 2개가 제 각각 존재? 새로 배우는 입장에서는 난감
- auth.js 는 각종 callback 의 종류가 많아서 이해하는데 시간이 많이 걸렸다.
- react 의 useEffect 이건 왜 함정이 이리 많을까. 잘 쓰는게 어려울듯.
- css 배우는게 나에게는 난이도 최상
- auth.js 는 github 로그인 예제와 공식 문서가 일치하지 않아 많이 헤멨다
- 그래서 [PR](https://github.com/nextauthjs/next-auth/pull/13166){:target="_blank"} 제출했더니 돌아온 반응은 그건 별 문제 아니다라는(좀 이상했고)
