### 로딩 성능 최적화

- 이미지 사이즈 최적화
- Code Split
- 텍스트 압축

### 렌더링 성능 최적화

- Bottlenect 코드 최적화

---

## 이미지 사이즈 최적화

- 1200x1200 사이즈 이미지를 120x120px로 표시해주고 있다면 리소스 낭비이다.
- 그러면 이미지를 120x120 사이즈로 보내주는건 또 아니다. 디스플레이에 따라 보여주는 픽셀이 다르기 때문이다.
- 그러면 240x240px로 받으면서 120x120px로 렌더링 해줘야한다.

### CDN (Contents Delivery Network)

- 물리적 거리의 한계를 극복하기 위해 소비자(사용자)와 가까운 곳에 컨텐츠 서버를 두는 기술

### Image CDN (Image Processing CDN)

- 이미지를 사용자에게 보내기 전에 특정 형태로 가공하여 사용자에게 이미지를 전달한다.

```
http://cdn.image.com?src=[img src]&width-200&height=100
```

https://imgix.com/

### webp

- jpg, png 파일보다 가볍다. mac 프로그램의 xnconvert로 jpg파일을 webp로 변환하면 거의 2배 가까이 가벼워진다.

---

## Bottleneck 코드 최적화

- 특수 문자를 효율적으로 제거하기
  - replace 함수와 정규식 사용
  - 마크다운의 특스문자를 지워주는 라이브러리 사용 (remove-markdown)
- 작업하는 양 줄이기

### Code splitting & Lazy Loading

- ListPage, ViewPlage 따로따로 정크파일을 만들어서 각 페이지에 필요한 모듈만 로드하는 방식
- 불필요한 코드 또는 중복되는 코드가 없이 적절한 사이즈의 코드가 적절한 타이밍에 로드될 수 있도록 하는 것

```jsx
// App.jsx
import React, { lazy, Suspense } from 'react';
import { Switch, Route } from 'react-router-dom';
import './App.css';

const ListPage = lazy(() => import('./pages/ListPage/index'));
const ViewPage = lazy(() => import('./pages/ViewPage/index'));

function App() {
  return (
    <div className='App'>
      <Suspense fallback={<div />}>
        <Switch>
          <Route path='/' component={ListPage} exact />
          <Route path='/view/:id' component={ViewPage} exact />
        </Switch>
      </Suspense>
    </div>
  );
}

export default App;
```

### 압축

개발자도구 - 네트워크 탭 - 응답 헤더의 `content-encoding`이면 압축이 된 것.

네트워크 크기가 2KB 이하이면 압축을 하지 않는게 좋고 2KB 이상이면 압축 하는게 좋다.

---

## 애니메이션 최적화 (Reflow, Repaint)

- 렌더링 성능 최적화
- 쟁크 현상: 애니메이션이 버벅이는 현상 (초당 30프레임, 20프레임)

브라우저 렌더링 과정

DOM(html, body, div, 'hello', 'students') + CSSOM(font-size: 10px; color: red;) -> Render Tree(DOM + CSSOM 합침) -> Layout(위치, 크기 계산 width, height, margin, padding) -> Paint(background-color, color 등 색상을 집어 넣음) -> Composite (각 레이어 합성)

### Reflow

- width, height 위치나 크기 변경
- DOM + CSSOM -> Render Tree -> Layout -> Paint -> Composite (모두 재실행)

### Repaint

- color, background-color 색상 변경
- DOM + CSSOM -> Render Tree -> Paint -> Composite (Layout 생략)

### GPU 도움 받아 Reflow, Repaint 피하기

- transform, opacity (GPU가 관여할 수 있는 속성) 변경
- DOM + CSSOM -> Render Tree -> Composite (Layout, Paint 생략)

```css
// 기존
width: ${({ width }) => width}%;
transition: width 1.5s ease;

// 변경
width: 100%;
transform: scaleX(${({ width }) => width / 100});
transform-origin: center left;
transition: transform 1.5s ease;
```

## 컴포넌트 Lazy Loading (Code Splitting)

- 로딩 성능 최적화

```shell
yarn add -D cra-bundle-analyzer
npx cra-bundle-analyzer
```

`react-image-gallery` 라이브러리가 처음 페이지를 불러올 때 같이 포함되서 불러온다.
하지만 현재 로직에서는 버튼을 눌렀을 때 해당 라이브러리가 포함된 컴포넌트를 불러오기 때문에
처음 페이지에 같이 불러올 필요가 없다.

```tsx
// src/App.js
import React, { useState, Suspense, lazy } from 'react';
// import ImageModal from './components/ImageModal'

const LazyImageModal = lazy(() => import('./components/ImageModal'));

<Suspense fallback={null}>
  {showModal ? (
    <LazyImageModal
      closeModal={() => {
        setShowModal(false);
      }}
    />
  ) : null}
</Suspense>;
```

## 컴포넌트 Preloading

- 로딩 성능 최적화

Lazy Loading의 단점이 여기서 나타나는데 버튼을 클릭한 순간 컴포넌트를 불러와 약간의 딜레이가 발생한다.
버튼을 클릭 하기 전에 컴포넌트를 미리 로드할 수 있다. 그러면 버튼을 클릭한 순간 해당 컴포넌트를 바로 확인할 수 있다.

### 컴포넌트 Preload 타이밍

1. 버튼 위에 마우스를 올려 놨을 때
   1. 불러올 컴포넌트의 파일이 너무 크다면 로드할 때 딜레이가 발생한다. 그럴 때는 버튼 위에 마우스를 올렸을 때 보다 먼저 불러오는게 낫다.

```jsx
// src/App.js
const LazyImageModal = lazy(() => import('./components/ImageModal'));


const handleMouseEnter = () => {
  // 할당이 필요하지만, 사용하지 않아도 된다.
  const ImageComponent = import('./components/ImageModal');
};

return (
  <div className='App'>
    <Header />
    <InfoTable />
    <ButtonModal
      onClick={() => {
        setShowModal(true);
      }}
      onMouseEnter={handleMouseEnter}
      >
      올림픽 사진 보기
    </ButtonModal>
    <SurveyChart />
    <Footer />
    <Suspense fallback={null}>
      {showModal ? (
        <LazyImageModal
          closeModal={() => {
            setShowModal(false);
          }}
          />
      ) : null}
    </Suspense>
  </div>
);
```

2. 최초 페이지가 로드가 되고, 모든 컴포넌트의 마운트가 끝났을 때

```jsx
// src/App.js
useEffect(() => {
  const ImageComponent = import('./components/ImageModal');
}, []);
```

```jsx
const lazyWithPreload = importFunction => {
  const Component = React.lazy(importFunction);
  Component.preload = importFunction;
  return Component;
};
const LazyImageModal = lazyWithPreload(() => import('./components/ImageModal'));

useEffect(() => {
  LazyImageModal.preload();
}, []);
```

## 이미지 Preloading

- 로딩 성능 최적화

기본적으로 이미지를 캐싱하기 때문에 똑같은 주소를 미리 불러와 로드할 수가 있다.

```jsx
useEffect(() => {
    LazyImageModal.preload();

    const img = new Image();
    img.src =
      'https://stillmed.olympic.org/media/Photos/2016/08/20/part-1/20-08-2016-Football-Men-01.jpg?interpolation=lanczos-none&resize=*:800';
  }, []);
```

