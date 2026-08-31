---
title: browser
version: canary
---

<Intro>

<Canary>

**`browser` API는 현재 React의 카나리 및 실험적 채널에서만 사용할 수 있습니다.**

[React의 배포 채널에 대해 더 알아보세요.](/community/versioning-policy#all-release-channels)

</Canary>

`browser`를 사용하면 서버 렌더링 동안 컴포넌트를 브라우저 전용으로 표시할 수 있습니다.

```js
use(browser(reason?))
```

</Intro>

<InlineToc />

---

## 레퍼런스 {/*reference*/}

### `browser(reason?)` {/*browser*/}

[`use`](/reference/react/use) 내부에서 `browser`를 호출하여 서버 렌더링 동안 컴포넌트를 브라우저 전용으로 표시합니다.

```js
import { use } from 'react';
import { browser } from 'react-dom';

function BrowserOnly() {
  use(browser('This component requires browser APIs.'));
  return <BrowserContent />;
}
```

서버 렌더링 동안 `use(browser())`는 컴포넌트의 렌더링을 멈추고 가장 가까운 [`<Suspense>`](/reference/react/Suspense) 경계의 폴백을 그 자리에 남깁니다. 브라우저에서 `use(browser())`는 `undefined`를 반환하므로 컴포넌트가 정상적으로 렌더링됩니다.

[아래에서 더 많은 예시를 확인하세요.](#usage)

#### 매개변수 {/*parameters*/}

* `reason`**(선택사항)**: 콘텐츠가 왜 브라우저에서 렌더링되어야 하는지 설명하는 문자열 또는 함수입니다. 문자열 또는 함수의 반환값은 [`onBrowserBailout`](#reporting-browser-only-rendering-on-the-server)에 전달되는 `Error`의 `cause`가 됩니다. 서버 렌더러가 `browser`가 반환한 값을 마주칠 때마다 React는 `reason` 함수를 호출하지만, 브라우저에서는 호출하지 않습니다. `reason`을 만드는 비용이 크다면 `() => new Error(...)`와 같은 함수를 전달하세요.

#### 반환값 {/*returns*/}

`browser`는 컴포넌트 내부의 `use`에 전달하거나 [서버 렌더링을 중단할](#aborting-pending-server-rendering-for-the-browser) 때 `reason`으로 사용할 수 있는 불투명한 값을 반환합니다. 브라우저에서 이 값을 `use`에 전달하면 `undefined`를 반환합니다.

#### 주의 사항 {/*caveats*/}

* `use(browser())`는 서버 렌더링 동안 `<Suspense>` 경계 내부에 있어야 합니다. 경계가 없으면 서버 렌더링이 실패합니다.
* React 서버 컴포넌트 앱에서 `use(browser())`는 [서버 컴포넌트](/reference/rsc/server-components)가 아니라 [클라이언트 컴포넌트](/reference/rsc/use-client)에서 호출해야 합니다.
* `browser()`를 단독으로 호출하는 것은 아무 효과가 없습니다. 컴포넌트를 브라우저 전용으로 표시하려면 `browser`가 반환한 값을 `use`에 전달하세요. 이 값을 throw하지 마세요.

---

## 사용법 {/*usage*/}

### 브라우저에서만 콘텐츠 렌더링하기 {/*rendering-content-only-in-the-browser*/}

브라우저에서만 렌더링되어야 하는 컴포넌트의 `use` 안에서 `browser`를 호출하세요.

`typeof window`를 확인하거나, 마운트 상태를 설정하는 [`Effect`](/reference/react/useEffect)를 기다리거나, 서버 렌더링을 비활성화하는 프레임워크 옵션을 사용하는 대신 이 방법을 사용할 수 있습니다.

초기 HTML에서 로딩 폴백을 보려면 **Reload**를 클릭하세요. 하이드레이션 이후 React는 `localStorage`에서 불러온 초안을 표시합니다.

<Sandpack>

```js src/App.js active
import { Suspense, use, useState } from 'react';
import { browser } from 'react-dom';

function SavedDraft() {
  use(browser('The draft is stored in localStorage.'));
  const [draft, setDraft] = useState(
    () => localStorage.getItem('draft') ?? ''
  );

  function handleChange(event) {
    const nextDraft = event.target.value;
    setDraft(nextDraft);
    localStorage.setItem('draft', nextDraft);
  }

  return (
    <label>
      Draft:
      <textarea
        value={draft}
        onChange={handleChange}
        rows={4}
        cols={30}
      />
    </label>
  );
}

export default function App() {
  return (
    <>
      <h1>Saved draft</h1>
      <Suspense fallback={<p>Loading draft...</p>}>
        <SavedDraft />
      </Suspense>
    </>
  );
}
```

```js src/Document.js hidden
import App from './App.js';

export default function Document() {
  return (
    <html lang="en">
      <head>
        <title>Saved draft</title>
        <style>{`
          h1 { font-size: 24px; margin-top: 0; }
          label, textarea { display: block; }
          textarea { margin-top: 5px; }
        `}</style>
      </head>
      <body>
        <App />
      </body>
    </html>
  );
}
```

```js src/index.js hidden
import { hydrateRoot } from 'react-dom/client';
import { renderToReadableStream } from 'react-dom/server';
import Document from './Document.js';
import { flushReadableStreamToFrame } from './demo-helpers.js';
import './styles.css';

async function main(frame) {
  const stream = await renderToReadableStream(<Document />);
  await flushReadableStreamToFrame(stream, frame);

  // Wait so both the fallback and hydrated content are visible.
  await new Promise(resolve => setTimeout(resolve, 1200));
  hydrateRoot(frame.contentDocument, <Document />);
}

main(document.getElementById('preview'));
```

```js src/demo-helpers.js hidden
export async function flushReadableStreamToFrame(readable, frame) {
  const doc = frame.contentWindow.document;
  const decoder = new TextDecoder();
  const reader = readable.getReader();

  while (true) {
    const {done, value} = await reader.read();
    if (done) {
      break;
    }
    doc.write(decoder.decode(value, {stream: true}));
  }

  doc.write(decoder.decode());
  doc.close();
}
```

```html public/index.html hidden
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Browser-only rendering</title>
</head>
<body>
  <iframe id="preview" title="Rendered page"></iframe>
</body>
</html>
```

```css src/styles.css hidden
iframe {
  width: 100%;
  height: 160px;
  border: 0;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "19.3.0-canary-eb8feb71-20260814",
    "react-dom": "19.3.0-canary-eb8feb71-20260814",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

<Note>

React 서버 컴포넌트 앱에서 `use(browser())`는 클라이언트 컴포넌트에서 호출해야 합니다. 사용 중인 프레임워크가 기본적으로 서버 컴포넌트를 사용한다면 해당 파일에 [`'use client'`](/reference/rsc/use-client) 지시어를 추가하거나 호출을 자식 클라이언트 컴포넌트로 옮기세요.

```js {1}
'use client';

import { use, useState } from 'react';
import { browser } from 'react-dom';

export default function SavedDraft() {
  use(browser('The saved draft is stored in localStorage.'));
  const [draft] = useState(() => localStorage.getItem('draft') ?? '');
  return <DraftEditor initialDraft={draft} />;
}
```

</Note>

---

### 브라우저에서 조건부로 렌더링하기 {/*conditionally-rendering-in-the-browser*/}

다른 [`use`](/reference/react/use) 호출과 마찬가지로 `use(browser())`를 조건부로 혹은 커스텀 Hook 내부에서 호출할 수 있습니다. 예를 들어 `Suspense`를 지원하는 데이터 페칭 라이브러리의 `useQuery`를 감싸서 초기 데이터가 없을 때 서버 렌더링을 생략할 수 있습니다.

```js {3}
function useBrowserQuery(query, options) {
  if (options.initialData === undefined) {
    use(browser('useBrowserQuery: No initial data was provided.'));
  }

  return useQuery(query, options);
}

function ProductDetails({ productId, initialData }) {
  const product = useBrowserQuery(`/api/products/${productId}`, {
    initialData,
  });

  return <h1>{product.name}</h1>;
}
```

서버에서 `useBrowserQuery`는 초기 데이터가 있을 때만 `useQuery`를 호출합니다. 그렇지 않으면 가장 가까운 `Suspense` 경계의 폴백이 HTML에 유지됩니다. 브라우저에서는 `use(browser())`가 `undefined`를 반환하기 때문에 쿼리 라이브러리는 데이터를 페치하거나 클라이언트 캐시에서 읽어올 수 있습니다.

---

### 서버에서 브라우저 전용 렌더링 보고하기 {/*reporting-browser-only-rendering-on-the-server*/}

브라우저 전용 렌더링을 보고하려면 서버 렌더러에 `onBrowserBailout` 콜백을 전달하세요. React가 브라우저를 위해 Suspense 폴백을 남길 때는 서버 렌더러의 `onError` 콜백이나 [`hydrateRoot`의 `onRecoverableError`](/reference/react-dom/client/hydrateRoot#error-logging-in-production) 콜백을 호출하지 않습니다. 이 예시에서는 `reason`도 함께 전달하며, 이 `reason`은 보고된 에러의 `cause`로 확인할 수 있습니다.

```js
import { Suspense, use, useState } from 'react';
import { browser } from 'react-dom';
import { renderToPipeableStream } from 'react-dom/server';

function SavedDraft() {
  use(browser(() => new Error('The saved draft is stored in localStorage.')));
  const [draft] = useState(() => localStorage.getItem('draft') ?? '');
  return <DraftEditor initialDraft={draft} />;
}

const { pipe } = renderToPipeableStream(
  <Suspense fallback={<p>Loading saved draft...</p>}>
    <SavedDraft />
  </Suspense>,
  {
    onShellReady() {
      pipe(response);
    },
    onBrowserBailout(error, errorInfo) {
      logBrowserBailout(error, errorInfo);
    }
  }
);
```

`onBrowserBailout`은 두 개의 인수를 받습니다.

1. 브라우저 전용 렌더링을 설명하는 `Error`. `browser`에 `reason`을 전달했다면 에러의 `cause`로 확인할 수 있습니다.
2. 브라우저 전용 렌더링이 발생한 위치를 보여주는 `componentStack`을 담은 `errorInfo` 객체.

`reason` 함수는 어떤 값이든 반환할 수 있습니다. 새로운 `Error`를 반환하면 브라우저에서 `Error`를 생성하지 않고도 `cause`에 자체 스택 트레이스를 부여할 수 있습니다. React는 `reason`을 HTML로 직렬화하지 않습니다.

폴백을 제공할 Suspense 경계가 없으면 서버 렌더링이 실패합니다. 이때 React는 `onBrowserBailout` 대신 렌더러의 일반적인 에러 콜백을 통해 실패를 보고합니다.

---

### Aborting pending server rendering for the browser {/*aborting-pending-server-rendering-for-the-browser*/}

If you call a server rendering API directly, you can stop waiting for pending content and let the browser finish rendering it. Pass the value returned by `browser` as the reason when aborting the server render. React then leaves pending Suspense boundaries in their fallback state and renders their content in the browser:

```js {1,8}
import { browser } from 'react-dom';
import { renderToPipeableStream } from 'react-dom/server';

const { pipe, abort } = renderToPipeableStream(<App />, {
  onShellReady() {
    pipe(response);
    setTimeout(() => {
      abort(browser('The server render timed out.'));
    }, 10000);
  }
});
```

A `browser` abort reason does not trigger the server renderer's `onError` callback or `hydrateRoot`'s `onRecoverableError` callback. Instead, the server renderer reports each recovered Suspense boundary to `onBrowserBailout`.

For server rendering APIs that accept an [`AbortSignal`](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal), pass `browser()` as the reason to [`AbortController.abort`](https://developer.mozilla.org/en-US/docs/Web/API/AbortController/abort).
