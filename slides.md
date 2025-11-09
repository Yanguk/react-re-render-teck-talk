---
# try also 'default' to start simple
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: React Re-rendering
info: |
  tech-talk
# apply UnoCSS classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
---

# React Re-rendering

리액트 동작 방식을 이해하고 리액트 최적화까지

<p class="abs-br m-12 text-xl">
  허양욱 @ Daekyo
</p>

---
layout: center
---

# 리액트 렌더링 과정

<img src="/render-process.png" alt="렌더링 과정" />

https://ko.react.dev/learn/render-and-commit

---

# Trigger 조건

1. 컴포넌트의 초기 렌더링인 경우
```tsx {0|all|2|0}
const root = createRoot(document.getElementById('root'))
root.render(<App />);
```

<br />

2. 컴포넌트의 state가 업데이트 된 경우

```tsx {0|all|2,5}
function Counter() {
  const [count, setCount] = useState(0)

  const handleOnClick = () => {
    setCount(count + 1)
  }

  return (
    <Layout>
      {...}
      <button onClick={handleOnClick}>{count}</button>
    </Layout>
  )
}
```

---

# Render -> Commit

<v-clicks>

- Render는 React에서 컴포넌트를 호츨하는 것
- Commit은 DOM이 최신 렌더링 출력과 일치하도록 하는 것
  - 이때 React는 렌더링 간에 차이가 있는 경우에만 DOM 노드를 변경한다.

</v-clicks>

<br />

<v-click>

<iframe src="https://stackblitz.com/edit/vitejs-vite-ccptunzv?embed=1&file=src%2FClock.jsx&theme=dark"
  style="width:100%; height:280px; border:0; border-radius:6px; overflow:hidden;"
></iframe>

</v-click>

---
layout: section
---
# 최적화 하기
Render 단계를 최소화 하기

---

# 흔히 사용하는 최적화 방법

<br />

<v-clicks>

## <span class="text-purple-400">React.memo</span>
컴포넌트의 props가 변경되지 않으면 리렌더링을 방지

## <span class="text-blue-400">useMemo</span>
계산 비용이 높은 값을 메모이제이션

## <span class="text-blue-400">useCallback</span>
함수를 메모이제이션하여 불필요한 재생성 방지

</v-clicks>

---

# React.memo 예시

```tsx {all|1-2|14-15,20-21|}
// 해당 컴포넌트는 data가 바뀌었을경우 렌더링됨
const ExpensiveComponent = React.memo(({ data }) => {
  const processedData = heavyCalculation(data);

  return (
    <div>
      <h3>{processedData}</h3>
      <button onClick={onClick}>Click me</button>
    </div>
  );
});

function Parent() {
  const [count, setCount] = useState(0);
  const data = { value: 100 };

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      {/* count가 변경되어도 리렌더링이 되지 않음 */}
      <ExpensiveComponent data={data} />
    </div>
  );
}
```

---

# useMemo & useCallback 예시

```tsx {all|6-9|10-14}
function Parent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');

  // 복잡한 계산 결과를 메모이제이션
  const expensiveValue = useMemo(() => {
    return heavyCalculation(count);
  // 의존성 배열의 값이 바꿜때 만 재 계산됨
  }, [count]);

  // 함수를 메모이제이션
  const handleClick = useCallback(() => {
    console.log('Clicked!');
  }, []);

  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <ExpensiveComponent value={expensiveValue} onClick={handleClick} />
    </div>
  );
}
```

---
layout: center
---

# 하지만... 🤔

<v-clicks>

## 이 방법들의 문제점

<div class="mt-8 text-xl">

- 코드 가독성을 떨어뜨림
  - 렌더링 사이클에서 의존성배열을 같이 봐야하는게 자연스럽게 읽히지는 않음
- 불필요한 복잡성 증가
- 항상 이전값과 비교하는 연산이 들어가서 오히려 성능이 저하 될 수 있음

</div>

</v-clicks>

---
layout: center
class: text-center
---

# 다른 방식 소개

<div class="text-3xl mt-12">
  <v-clicks>
    <div class="mb-6">Children 패턴</div>
    <div class="text-2xl opacity-60">+</div>
    <div class="mt-6">Context API</div>
  </v-clicks>
</div>

---

# Children 패턴이란?

<v-clicks>

- React의 기본 기능을 활용한 최적화 패턴
- 부모 컴포넌트가 리렌더링되어도 children은 리렌더링되지 않음

</v-clicks>

---

# 일반적인 패턴 (문제 상황)

```tsx {all|2,7-8,12-17}
function ExpensiveChild() {
  console.log('ExpensiveChild rendered');
  return <div>비싼 연산을 하는 컴포넌트</div>;
}

function Parent() {
  const [count, setCount] = useState(0);
  console.log('Parent rendered');

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Count: {count}
      </button>

      {/* count 변경시 매번 실행됨! */}
      <ExpensiveChild />
    </div>
  );
}
```

<v-click>

<div class="mt-4 p-3 bg-red-500/10 rounded border-l-4 border-red-500">
  count가 변경될 때마다 ExpensiveChild도 리렌더링됩니다
</div>

</v-click>

---

# Children 패턴 적용 (해결)

```tsx {all|2-3,7-10|16-23}
function Parent({ children }) {
  const [count, setCount] = useState(0);
  console.log('Parent rendered');

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Count: {count}
      </button>
      {children}
    </div>
  );
}

// 사용
function App() {
  return (
    <Parent>
      {/* count가 변경 되어도 리렌더링 되지 않음 */}
      <ExpensiveChild />
    </Parent>
  );
}
```
---

<iframe src="https://stackblitz.com/edit/vitejs-vite-re4d2gss?embed=1&file=src%2FApp.jsx&theme=dark"
  style="width:100%; height:100%; border:0;"
></iframe>

---

