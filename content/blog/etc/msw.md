---
title: 'MSW로 목데이터 만들기'
date: 2023-09-13 12:20:15
category: 'etc'
draft: true
---

issu-tracker 프로젝트 당시 MSW를 사용해서 목서버를 띄웠던 경험이 있다. 그 당시에는 사용법만 익혀서 부랴부랴 적용하기 바빴다. 이후 과제를 진행하던 중 API가 동작하지 않아 목서버를 띄우면서 문득 의문이 들었다.

> 네트워크 요청을 어떻게 가로채는 걸까?

한켠에 자리잡고 있던 고민들을 해소하고 싶었고 MSW 동작방식을 이해하고 싶어 이번에 공부를 해봤다. 차근차근 MSW에 대해서 알아보면서 제대로 이해해보자!

## 프로젝트 이상과 현실

![](./images/msw/msw-1.jpeg)

웹 개발 단계에서 가장 이상적인 방법은 `기획 → 백엔드 개발 → 프론트엔드 개발` 순서로 개발이 진행되는 것이다. 하지만 언제나 마감기한은 존재하며, 현실은 백엔드, 프론트엔드 개발이 동시에 진행되는 경우가 많다.

이러한 상황에서 백엔드에서 API가 나오지 않았다면 프론트엔드 개발이 지연되는 문제가 발생할 수 있다.

팀 프로젝트를 진행하다보면 다음과 같은 대화를 많이 들어봤을 것이다.

```
👩🏻‍💻 프론트: 혹시.. api 언제쯤 나올 수 있을까요..?
(이틀 뒤)
👩🏻‍💻 프론트: api 나왔나요?
🧑🏻‍💻 백엔드: ...
```

프론트는 API가 나올 때까지 대기해야 하고, 백엔드는 지속적인 압박을 받는 부담이 있을 것이다. 그러나 API가 나올 때까지 프론트 개발을 멈추는 건 비효율적이다. 이러한 상황에서 효율적으로 개발을 하는 방법을 뭘까?

바로 백엔드가 API 개발이 완료될 때까지 기다리지 않고 Mocking을 하는 것이다!

## Mocking? 그게 뭔데?

Mocking의 사전적 의미는 `거짓된, 가짜의` 라는 뜻이다. 즉, 실제 값이 아닌 가짜 값을 만드는 것이다.

그럼 현재 우리의 상황에서의 Mocking은 가짜 API를 생성한다고 생각하면 된다. 한가지 주의할 점은 Mocking을 하기 전에 백엔드와 먼저 API 스펙을 협의한 다음 정해야하는 점을 명시해야한다.
API 스펙을 제대로 정하지 않으면 프론트 개발을 다 마치고나서 백엔드 API와 데이터가 달라 다시 개발을 하게 될 수 있다.

요약하면 아래와 같은 순서로 진행된다고 볼 수 있다.

> 기획 완료 -> 백엔드와 API 스펙을 협의 -> 스펙을 토대로 Mocking

그럼 이제 Mocking을 어떻게 처리하는지에 대해서 알아보자.

## 기존 Mocking 처리 방식

#### 직접 Mockup 데이터를 서비스 로직에 추가하는 방식

- 장점 : 구현이 쉽고 빠르다.
- 단점 : 서비스 로직을 수정해야 하며, HTTP 메서드 및 응답 상태에 따른 대응이 어려움.

#### 네이티브 모듈(http, https, XMLHttpRequest)을 다른 함수로 대체하여 Mocking하는 방식

- 장점 : 원하는 응답을 받을 수 있음.
- 단점 : 실제 환경과의 차이가 있으며, 목업 코드를 수동으로 작성하고 유지 관리해야 함.

#### 목업 서버 직접 구축하여 응답 데이터를 받아 처리하는 방식

- 장점 : 서비스 로직 수정이 필요하지 않음.
- 단점 : 구현에 시간이 많이 소요되며, 실제 서버와 다른 방식으로 동작할 수 있어 기존 코드 수정이 필요할 수 있음.

<br>

기존 Mocking 처리 방식의 문제점을 해결할 수 있는 것이 바로 MSW (Mock Service Worker) 다.
MSW는 네트워크 요청을 중간에서 가로채기 때문에 가로채기 위한 코드를 따로 작성하지 않아도 되며, 서비스 로직을 수정하지 않아도 된다. MSW를 사용하면 복잡한 처리없이 간편하게 Mocking을 할 수 있다.

그럼 이제 MSW가 무엇인지 알아보자!

## MSW (Mock Service Worker)란?

공식 문서에 따르면 MSW는 실제 요청을 서비스 워커를 통해 가로채서 목서버로 대체해주는 Mocking 라이브러리이다.

### 언제 사용할 수 있을까?

아까 이야기 했던 프론트와 백엔드 개발이 동시에 이루어지는 상황도 있겠지만 그 외에도 테스트, 디버깅 용도로도 사용이 가능하다.

- 개발 단계에서 백엔드 api가 준비되지 않은 상황
- 데이터를 디버깅해보고 싶을 때
  <br><br>

MSW에서 핵심은 실제 요청을 가로챈다는 것인데 이 역할을 서비스 워커가 해준다.

## 서비스 워커 (Service Worker)란?

![](./images/msw/msw-2.jpeg)

서비스 워커는 웹 브라우저의 백그라운드에서 동작하는 스크립트이다. 그렇기 때문에 웹 애플리케이션의 프론트엔드 코드와 별도로 실행된다. 이러한 특징 덕분에 애플리케이션의 UI Blocking 없이 연산을 처리할 수 있으며, 오프라인 환경에서도 작동이 가능하다.

서비스 워커의 특징들을 이용해서 다음과 같이 활용 가능하다.

- 캐시와 상호작용 : 요청이 오면 가지고 있던 캐시에서 자료를 전달한다.
- 푸쉬 알림 : 브라우저 창이 닫힌 상태에서도 동작하므로, 푸쉬알림을 구현할 수 있다.
- 백그라운드 동기화 : 채팅, 사진 업로드 등 작업 도중 오프라인 상태가 되는 경우 실패하지 않고 온라인 상태가 되었을 때 작업을 마저 완료할 수 있다.
  <br><br>

MSW는 이러한 서비스 워커의 기능 중에서 요청을 가로채고 캐싱을 활용하여 모의 응답을 생성하고 관리한다. 이를 통해 프론트엔드가 백엔드 API에 의존하지 않도록 도와준다.

## MSW 동작 원리

![](./images/msw/msw-3.jpeg)

MSW 동작 원리를 순서도로 그려보았다. 풀어서 설명하면 아래와 같다.

1. 먼저 브라우저에 서비스 워커를 등록한다. 네트워크 요청이 발생하면 서비스 워커가 가로챈다.

2. 서비스 워커는 가로챈 요청을 복사하여 MSW로 전송한다. 이로써 실제 서버로의 요청은 보류되고, 대신 클라이언트 사이드에 있는 MSW가 요청을 처리하게 된다.

3. MSW는 이전에 설정한 핸들러에 따라 해당 요청에 대한 모의 응답을 생성하여 서비스 워커로 전송한다.

4. 서비스 워커는 받은 모의 응답을 브라우저로 전달한다.

이러한 과정을 통해, Mocking이 가능해진다.

## MSW 사용법과 실습

동작 원리를 이해했으니 이제 간단한 실습을 해보자.

### MSW 설치

먼저 MSW를 설치해준다.

```shell
npm install msw --save-dev
# or
yarn add msw --dev
```

### 서비스 워커 등록

MSW를 사용하려면 브라우저에 서비스 워커를 등록해야한다.

아래 명령어를 실행하면 서비스 워커 등록을 위한 파일(mockServiceWorker.js)이 `public` 폴더에 추가된다.

```shell
npx msw init public/ --save
```

![](./images/msw/msw-4.png)

### 요청 핸들러 작성

이제 핸들러를 작성하여 어떻게 모의 응답을 설정할 수 있는지 알아보겠다.
REST API를 모킹하는 경우, msw 모듈의 rest 객체를 사용한다.

`rest` 객체는 다음과 같은 3가지 인자를 받는다.

- `req` : 요청에 대한 정보
- `res` : 모의 응답을 생성하는 함수
- `ctx` : 모의 응답에 상태코드, 헤더 본문을 설정하는 함수.

<br>

핸들러를 작성하는 코드는 다음과 같다.

```js
// src/mocks/handlers.js

import { rest } from 'msw'

const foodList = [
  // 응답 데이터 형태
  {
    id: 1,
    name: '브라우니',
  },
  {
    id: 2,
    name: '볶음밥',
  },
]

export const handlers = [
  // HTTP 메서드 + API URL
  rest.get('/foods', (req, res, ctx) => {
    return res(
      ctx.status(200), // 응답 상태 설정
      ctx.delay(3000), // 응답 시간 설정
      ctx.json(foodList) // 응답 결과 설정
    )
  }),

  rest.post('/foods', (req, res, ctx) => {
    let foodID = foodList.length + 1

    foodList.push({
      id: foodID,
      name: req.body, // 요청 바디에 넘어온 데이터를 foodList에 추가
    })

    return res(ctx.status(201), ctx.json(foodList))
  }),
]
```

위 코드에서 `/foods` 경로에 대한 GET 요청은 상태 코드 200, 3초 딜레이 후 응답으로 foodList를 반환한다. POST 요청은 요청 바디로 받은 데이터를 foodList에 추가하고 상태 코드 201을 반환한다.

### 워커 설정

핸들러를 작성한 후, 해당 핸들러를 MSW에서 사용할 수 있도록 워커를 설정해야 한다. 다음과 같이 설정할 수 있다.

```js
// src/mocks/browser.js

import { setupWorker } from 'msw'
import { handlers } from './handlers'

export const worker = setupWorker(...handlers)
```

`setupWorker` 함수를 사용하여 방금 작성한 핸들러를 워커에 등록한다.

### 워커 실행

마지막으로 MSW를 실행하도록 설정해야 한다. 이를 위해 `src/index.js` 파일에서 다음 코드를 추가해준다. 이 코드는 개발 환경에서만 MSW를 실행한다.

```js
// src/index.js

if (process.env.NODE_ENV === 'development') {
  const { worker } = require('./mocks/browser')
  worker.start()
}
```

이제 MSW가 정상적으로 동작하면 콘솔에 `[MSW] Mocking enabled` 라는 메시지가 표시된다.

![image](https://github.com/saseungg/how-dns-works-translation/assets/115215178/f67e4b63-bfd6-4492-8332-dab6bcedea0f)

### 데이터 GET 요청

간단하게 fetch를 이용해서 GET 요청을 보내보겠다. fetch로 데이터를 받아와서 콘솔에 출력하도록 작성하였다.

```js
fetch('/foods')
  .then(res => res.json())
  .then(data => console.log(data))
```

콘솔에 데이터가 제대로 출력되고 응답 상태에 대한 정보도 정확하게 표시된다.

![](./images/msw/msw-6.png)

네트워크 탭을 확인해보면 해당 요청이 Service Worker에서 응답이 왔다는 걸 확인할 수 있다.

### UI 구현

React를 사용하여 간단한 음식 목록을 표시하고, 새로운 음식을 추가하는 UI를 구현했다. 이 코드에서는 GET 요청으로 음식 목록을 가져오고, POST 요청으로 새로운 음식을 추가한다. 리액트 코드에 대한 설명은 생략하겠다.

```js
import { useEffect, useState } from 'react'
import './index.css'

function App() {
  const [foodList, setFoodList] = useState([])
  const [newFood, setNewFood] = useState('')

  //  GET 요청을 사용하여 초기 음식 목록을 가져온다.
  const fetchData = () => {
    fetch('/foods')
      .then(res => res.json())
      .then(data => setFoodList(data))
  }

  useEffect(() => {
    fetchData() // 처음 마운트시만 음식 목록을 가져옴.
  }, [])

  const handleAddFood = async e => {
    e.preventDefault()

    // POST 요청을 사용하여 음식 추가
    await fetch('/foods', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(newFood),
    })

    fetchData() // POST로 추가한 음식을 목록에 반영하기 위해서 다시 GET
    setNewFood('') // 입력 필드 비우기
  }

  return (
    <div>
      <p>🍕 오늘 내가 먹은 음식은? 🍔</p>
      {foodList.map(food => (
        <li key={food.id}>{food.name}</li>
      ))}
      <form onSubmit={handleAddFood}>
        <input
          type="text"
          placeholder="또 뭐먹었어?"
          onChange={e => setNewFood(e.target.value)}
          value={newFood}
        ></input>
        <button type="submit">추가</button>
      </form>
    </div>
  )
}

export default App
```

![](./images/msw/msw-7.gif)

GET 요청일 때 딜레이를 3초로 해놨기 때문에 네트워크 탭을 보면 데이터를 가져올 때 `pending` 상태에 머물러 있다가 status가 200으로 바뀌면서 값이 들어오는 것을 확인 할 수 있다. 음식을 입력하고 추가하면 POST가 제대로 되는 것도 확인 가능하다.

해당 코드를 참고할 수 있도록 [코드 링크](https://codesandbox.io/s/msw-example-8t6x8p?file=/src/App.js)를 첨부한다. 다만 샌드박스에서는 서비스워커 에러가 자주 나서 정말 코드 참고만 하는 용도로 올린다. 실제 vscode에서는 잘 동작한다.

## 마무리

이번에 정리하면서 MSW 사용법과 동작원리를 제대로 이해할 수 있었고 아직 공부하지 않은 에러나 테스트 하는 방법을 나중에 학습하고 업데이트 해야겠다. 프로젝트를 할 때 API가 지연된다면 MSW는 거의 필수인 것 같다.

## References

- [Getting started - Mock Service Worker](https://mswjs.io/docs/getting-started)
- [Service Worker API - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
- [MSW로 백엔드 API 모킹하기 | Engineering Blog by Dale Seo](https://www.daleseo.com/mock-service-worker/)
- [Mocking으로 생산성까지 챙기는 FE 개발 – tech.kakao.com](https://tech.kakao.com/2021/09/29/mocking-fe/)
- [ServiceWorker 이모저모 이야기 | SOSOLOG](https://so-so.dev/web/service-worker/)
- [MSW - 더 나이스한 목킹을 위한 고민 | MADTECH](https://tech.madup.com/mock-service-worker/)
- [MSW를 활용하는 Front-End 통합테스트 | 카카오엔터테인먼트 FE 기술블로그](https://fe-developers.kakaoent.com/2022/220825-msw-integration-testing/)
