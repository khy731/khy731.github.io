---
title: "프론트엔드에서 CORS 에러 해결하기"
excerpt: "에러 메세지는 생각보다 친절하다"

categories:
    - react
tags:
    - react

toc: true
toc_sticky: true
---

## 프론트엔드에서 한 번쯤은 경험하게 되는 에러

프론트엔드 코드를 짜다가 외부 API를 호출할 때, 한 번쯤은 이런 에러 메세지를 접하게 된다.

![CORS](https://user-images.githubusercontent.com/97890886/199764752-6ca52293-654e-42b1-81e5-0432464a9006.png)

무슨 말인지는 잘 모르겠지만, 비교적 친숙한 다른 영어들에 비해 `CORS`라는 단어가 유독 눈에 띈다. 아마도 저게 에러의 주된 원인일 것이다.

우리는 매일 수십번의 에러를 만나며 골머리를 앓지만, 사실 에러 메세지는 의외로 한정되어 있으며 ~~모국어가 아니라는 점만 빼면~~ 생각보다 친절하다. 나는 개발 초보자이기 때문에, 개념부터 해결까지 차근차근 짚어갈 생각이다.

## CORS란 무엇인가?

> `CORS`는 웹에서 다른 출처로의 리소스 요청에 대한 정책을 의미한다.
> 

![CORS2](https://user-images.githubusercontent.com/97890886/199764776-4aa312e4-2082-48d2-b2b2-5480ab70416a.png)

`CORS`란 `Cross Origin Resource Sharing`의 약자로, 직역하면 `교차 출처 자원 공유`정도가 되겠다.  말 그대로 서로 다른 출처끼리 자원을 공유한다는 뜻인데, 뭔가 딱딱하지만 우리가 CORS 에러를 마주친 시점이 **외부 API를 사용했을 때**인것을 생각해 보면 어느정도 감이 온다. 아마 API를 제공하는 곳과 내 로컬이 다른 출처를 가졌고, API(자원)를 가져다 쓰려는 시점에서 뭔가가 문제가 생긴 것 같다.

### 그렇다면 출처는 무엇인가?

내 로컬이 뭘 잘못했길래 오픈소스로 제공되는 API도 못 쓰게 하는 것인가! 출처가 대체 뭐길래!

![CORS3](https://user-images.githubusercontent.com/97890886/199764846-99fe8760-a2f3-4d85-90e2-93655349d002.jpg)

`출처(origin)`은 url로 서버에 정보를 요청할 때, 서버를 찾아갈 수 있는 가장 기본적인 정보들이다. url은 단 한 줄로 이루어져 있지만 이를 쪼개면 여러 부위로 구분할 수 있다.

- **protocol**
- **host**
- path
- query string
- fragment

이 중 protocol과 host가 바로 `origin`이다. `origin`이 같다는 것은 **찾아갈 서버가 같다**는 말이다. 그렇다면 몇 가지를 더 유추할 수 있다. 예를 들면 이런 것: 내가 자원을 요청한 곳의 `origin`과 나의 `origin`이 달라서(url의 가장 기본적인 부분이 달라서) 어디에선가 이 요청을 막은 것이 아닐까?

## CORS의 동작 원리와 에러가 발생하는 이유

이제 본격적으로 CORS 에러가 왜 발생했는지 알아보자. 일단 결론부터 말하고 시작하겠다. 이는 **브라우저** 측 정책이며, 브라우저가 다른 `origin`에 대한 리스폰스를 차단한 것이다.

> 웹 생태계는 매우 위태롭고 취약한 곳이다.

최근에 읽은 책 중에 브라이언 커니핸의 <1일 1로그 100일 완성 IT 지식>이라는 책이 있는데, 이 책의 저자는 웹에 대해 완전히 통달하여 웹 메일조차 사용하지 않고(보안상 위험하다는 이유만으로) 리눅스 위에서 뭔가 항상 우회를 돌려가며 메일을 주고받는다고 한다. 나는 웹에 통달하지 않았기 때문에 무엇이 얼마나 위험한지는 모르지만, 매일같이 OO 웹 사이트 개인정보 탈취 등의 뉴스가 뜨는 것을 보면 해커들이 공격하기 쉬운 공간인 것은 분명하다. 

그래서 브라우저는 자원 공유에 원칙을 하나 추가한다. 바로 `SOP(same origin policy)`이다: “*같은 출처에서만 갖다 쓸 수 있어!”* 그러나 이건 마치 “*고속도로에서 교통사고가 많이 나니까 도로를 매워 버립시다.”* 와 비슷하다. 그래서 SOP의 예외로 있는 정책이 `CORS`이다. “*다른 출처에서도 갖다 쓸 수 있어요.* *그런데 이제 **허용된 출처**만.”*

웹 어플리케이션은 통신할 때 일반적으로
1. `OPTION` 메소드로 `preflight request(사전 요청)`을 보낸 후에
2. 본 요청을 보낸다.

![CORS4](https://user-images.githubusercontent.com/97890886/199764817-9d7ac9f4-0d1b-49e1-8517-f49651351767.png)


- `preflight request`의 `header`에는 `origin`이라는 필드가 있는데, 여기에 바로 나의 origin이 들어간다.
- `response`의 `header`에는 `Acces-Control-Allow-Origin`이라는 필드가 있는데. 이곳에 **접근이 허용된 origin**이 명시되어 있다.

즉, `preflight request`를 보내고 받은 `response`의 origin과 나의 origin을 **브라우저가 비교를 하여 다른 origin일 경우 요청을 컷하는 것**이다.

정리하자면,

> 예비 요청에서 보낸 나의 origin과 서버 측에서 허용하고 있는 origin이 일치하지 않을 경우 **브라우저**가 자원 요청을 거부하고 에러를 띄운다.
> 

## 에러를 해결해보자

지금까지 CORS 정책이 무엇이고, 왜 에러가 발생했는지 알아 보았다. 그렇다면 여러가지 해결 방식이 떠오른다. 가장 먼저 떠오른 방법은 **서버의 접근 허용 origin에 내 origin을 포함하도록 수정하는 것**이다. 그런데 만약, 나처럼 순전히 프론트엔드만을 개발하고 있어서 고유 서버가 없는 경우라면? 가장 쉬운 방법은 무엇일까? 어쩌면 CORS 정책을 검사하는 과정에서 **브라우저를 속일수도 있을 것**이다. 두 방법 모두 시도해보자.

### Proxy 서버로 우회

![CORS5](https://user-images.githubusercontent.com/97890886/199764679-39bd5088-8dc2-4286-8e3c-dc5389d09b15.png)

**프록시 서버**는 자신을 통해서 다른 네트워크 서비스에 간접적으로 접속할 수 있게 해 주며, 서버와 클라이언트간 일종의 중계 기능의 역할을 한다. 즉 원래 방식이 “나→외부API” 였다면, 이제 “나→프록시→외부API”로 브라우저를 속이는 것이다.

```bash
https://cors-anywhere.herokuapp.com/
```

이 프록시 서버를 사용해 보겠다.

일단, [위 url](https://cors-anywhere.herokuapp.com/)에 **접속**하여 프록시 서버를 구동시킨다. 그리고 API 요청 코드에서 엔드포인트 바로 앞에 복사+붙여넣기 한다.

이런 식이다.

```jsx
const getSearchMovie = async (search) => {
        if (!search) return;

        const response = await fetch(`https://cors-anywhere.herokuapp.com/https://openapi.naver.com/v1/search/movie.json?query=${search}`, {
            headers: {
                'X-Naver-Client-Id': ID_KEY,
                'X-Naver-Client-Secret': SECRET_KEY
            }
        })
        .then(res => res.json())
        .catch(err => console.error(err));
/* 생략 */
    };
```

이렇게 하면 정말 급한 불은 끈 느낌이다. 그냥 우회해서 해결한 것이다. 다른 방법도 한 번 해보자.

### Access-Control-Allow-Origin 세팅

만약 고유 서버가 있다면, 이 방법도 시도해 볼 수 있다. CORS 에러가 저쪽에서 허용시키는 origin과 내 origin을 비교하여 다르다고 잘라 버린 것에서 나온 에러라면, **애초에 내 origin을 허용시키면** 된다.

```jsx
const express = require('express');
const app = express();

app.get('/api', (requset, response) => {
    res.header("Access-Control-Allow-Origin", "내 origin");
    /* 생략 */
});
```

`Access-Control-Allow-Origin : *` 옵션을 사용할 수도 있겠지만, 아까 말했듯이 웹 생태계는 매우 취약하기 때문에 이처럼 “모든 출처를 허용한다!”는 *해커들도 환영합니다*와 다를 게 없다. 지양하는 것을 권장한다.

## 더 생각해 볼 것

사실 CORS는 **브라우저 측 정책**이기 때문에, 고유 서버가 있는 경우 브라우저(클라이언트)를 거치지 않고 서버 간 통신으로 데이터를 가져오면 CORS 정책에 위반되지 않는다. 설사 뭔가 문제가 생긴다고 하더라도 다양한 `미들웨어`, `라이브러리`를 통해 보다 본질적으로 접근하여 확실하게 해결할 수 있다. 만약 로컬 환경에서 프론트엔드만을 구축하다가 생긴 오류라면, `프록시 서버`를 활용하는 것이 가장 빠를 것이다. 실제로 배포하는 서비스의 경우 `서버 측`에서 보다 견고하게 구축해줘야 할 것이다.

### 참고

[https://lo-victoria.com/introduction-to-cross-origin-resource-sharing-cors](https://lo-victoria.com/introduction-to-cross-origin-resource-sharing-cors)

[https://beomy.github.io/tech/browser/cors/](https://beomy.github.io/tech/browser/cors/)