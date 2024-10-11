# 3. Provider 패턴

> **Q.** 여러 개의 컴포넌트가 데이터에 접근해야 하는 상황, **데이터를 어떻게 전달**할 것인가❓
> 

**A. `props`**를 통해서 데이터를 전달한다.

1. 모든 컴포넌트가 데이터에 접근해야 하는 경우, 코드 작성이 번거롭다.
2. 이 경우, ‘***prop drilling*** ’ 이라 불리는 *안티 패턴*이 사용된다.
    - *prop drilling*이란, 컴포넌트 트리 구조에서 데이터를 하위 컴포넌트로 전달하기 위해 중간 컴포넌트를 통해 프로퍼티를 내려주는 것을 말한다.
        
         예를 들어,  `App` 컴포넌트가 가진 데이터를 마지막 노드인 `ListItem`, `Header`, `Text` 컴포넌트가 각각 필요로 한다고 할 때, 이들에게 데이터를 주려면  `SideBar`와 `Content` 같은 중간 컴포넌트에게도 데이터를 전달해야 한다.
        
    
    ![image](https://github.com/user-attachments/assets/db5e7868-c98c-4a80-a101-63d0c431cf68)

    
    - props에 의존하는 컴포넌트들을 업데이트하기 힘들어지는 것은 물론, 어떤 데이터가 어디서부터 전해져 오는지 추적하기 어렵다는 단점이 있다.
    - *prop drilling*의 단점을 코드로 살펴보자!
        
        ```jsx
        function App() {
          const data = { ... }
        
          return (
            <div>
              <SideBar data={data} />
              <Content data={data} />
            </div>
          )
        }
        
        const SideBar = ({ data }) => <List data={data} />
        const List = ({ data }) => <ListItem data={data} />
        const ListItem = ({ data }) => <span>{data.listItem}</span>
        
        const Content = ({ data }) => (
          <div>
            <Header data={data} />
            <Block data={data} />
          </div>
        )
        const Header = ({ data }) => <div>{data.title}</div>
        const Block = ({ data }) => <Text data={data} />
        const Text = ({ data }) => <h1>{data.text}</h1>
        ```
        
         모든 컴포넌트가 `data`라는 props로 값을 전달 받고 있다. 만약 이 data라는 프로퍼티의 이름을 변경해야 하는 경우, **모든 컴포넌트를 수정**해야 한다.
        

** 안티 패턴*이란? 습관적으로 **많이 사용**하는 패턴이지만 성능, 디버깅, 유지 보수, 가독성 측면에서 부정적인 영향을 줄 수 있어 **지양하는** 패턴을 *안티 패턴*이라 한다.

---

## Provider 패턴?

<aside>
💡

특정 데이터를 필요로 하지 않는 컴포넌트는 해당 props를 받지 않는 것이 바람직하다!

</aside>

 이 경우 **Provider 패턴**을 이용하면, *prop drilling*에 의존하지 않고 각 컴포넌트들이 데이터에 직접 접근할 수 있도록 구현할 수 있다.

```jsx
const DataContext = React.createContext()

function App() {
  const data = { ... }

  return (
    <div>
      <DataContext**.Provider** **value={data}**>
        <SideBar />
        <Content />
      </DataContext**.Provider**>
    </div>
  )
}
```

- 모든 컴포넌트를 **`Provider` 키워드**로 감싸서 패턴을 사용할 수 있다.
- **`value`라는 prop**으로 하위 컴포넌트에 내려줄 데이터를 받는다. 그러면 이 컴포넌트의 자식 컴포넌트가 해당 provider를 통해 value prop에 접근할 수 있다.

- Provider는 HOC로, **`Context` 객체를 제공**한다. React가 제공하는 `createContext` 메소드를 활용하여 Context 객체를 만든다.
    - [Week1] HOC
        
         HOC(High Order Component, 고차 컴포넌트) 패턴은 컴포넌트를 가져와 **로직을 적용한 새 컴포넌트를 반환**하는 것을 말한다.
        
    
     즉, 어떤 컴포넌트를 Provider라는 고차 컴포넌트로 만들면 자식 컴포넌트는 Provider가 가진 value라는 prop을 직접 접근해 사용하는 로직을 적용할 수 있는 하위 컴포넌트가 되는 것이다.
    

 

 *그럼 자식 컴포넌트는 **`data`**에 어떻게 접근할까?*

```jsx
const DataContext = React.createContext();

function App() {
  const data = { ... }

  return (
    <div>
      <**DataContext**.Provider value={data}>
        <SideBar />
        <Content />
      </**DataContext**.Provider>
    </div>
  )
}

// 컴포넌트 트리
const SideBar = () => <List />
const List = () => <ListItem />
const Content = () => <div><Header /><Block /></div>

// useContext 훅으로 DataContext 객체를 받아와 data 값에 접근
function ListItem() {
  const { data } = React**.useContext(DataContext)**;
  return <span>{data.listItem}</span>;
}

function Text() {
  const { data } = React**.useContext(DataContext)**;
  return <h1>{data.text}</h1>;
}

function Header() {
  const { data } = React**.useContext(DataContext)**;
  return <div>{data.title}</div>;
}
```

 `Provider` 키워드를 통해 자식 컴포넌트에게 `data`를 직접 접근해서 사용할 권한을 전달하면, 각 컴포넌트는 **`useContext`** 훅을 사용하여 `data` 값에 접근할 수 있다.

 위 예제에서는 data를 필요로 하는 컴포넌트만 `useContext` 훅을 사용해 Provider 고차 컴포넌트에서 생성한 **`DataContext`**를 받아 data 값에 접근한다.

---

## Provider 사용법

Provider 패턴의 사용법을 정리하자면,

![image](https://github.com/user-attachments/assets/7ef474b2-f06d-4b41-bf6f-ddbc32b7f8d2)

1. 데이터를 전달할 **상위 컴포넌트**는 Provider로 묶어서 `Context` 객체를 만든다.
2. 데이터를 사용할 **자식 컴포넌트**는 `useContext` 훅을 사용해 Provider에서 만든 `Context` 객체를 받아 데이터에 접근한다.

---

## 패턴 활용 사례 분석

 일부 라이브러리는 자식 컴포넌트가 값을 쉽게 사용할 수 있도록 자체적인 Provider를 제공한다. 예를 들어 **styled-components**가 있다.

 styled-components는 `ThemeProvider`를 제공하므로 HOC를 직접 구현할 필요가 없다.

https://codesandbox.io/p/sandbox/provider-3-gbuls?file=/src/App.js:20,16&from-embed

```jsx
// styled-components 라이브러리가 자체적으로 제공하는 ThemeProvider를 import해 사용
**import { ThemeProvider } from 'styled-components'**

export default function App() {
  const [theme, setTheme] = useState('dark')

  function toggleTheme() {
    setTheme(theme === 'light' ? 'dark' : 'light')
  }

  return (
    <div className={`App theme-${theme}`}>
      <**ThemeProvider** theme={themes[theme]}>
        <>
          <Toggle toggleTheme={toggleTheme} />
          <List />
        </>
      </**ThemeProvider**>
    </div>
  )
}
```

---

## Provider 장/단점

### 장점

- 하위 컴포넌트를 향해 연결된 모든 컴포넌트에 데이터를 전달하지 않아도 데이터를 필요로 하는 다수의 몇몇 컴포넌트에만 데이터를 전달할 수 있다. 따라서 리팩토링 과정이 수월해진다.
- *prop-drilling*을 하지 않기 때문에 업데이트와 데이터 흐름의 추적이 비교적 쉽다.
- 컴포넌트들이 전역 영역에 접근할 수 있다.

### 단점

- 컨텍스트를 참조하는 모든 컴포넌트는 컨텍스트가 변경 될 때마다 **모두 리렌더링**되므로 Provider 패턴을 과하게 사용할 경우 특정 상황에서 성능 이슈가 발생할 수 있다.
    - 아래의 카운터 예제에서 `Increment` 버튼은 `Button` 컴포넌트 안에 있고, `Reset` 버튼은 `Reset` 컴포넌트 안에 있다. RESET COUNT 버튼을 누르면 카운트가 0으로 초기화되며 오른쪽에 카운터가 초기화된 시간이 나타난다.
        
        http://codesandbox.io/p/sandbox/provider-4-4ke0w?file=/src/index.js:1,1-61,1&from-embed
        
    - `Button` 컴포넌트와 `Reset` 컴포넌트가 모두 `CountContextProvider` 컨텍스트를 참조하고 있으므로 INCREMENT 버튼을 누르면 **모든 컴포넌트가 리렌더링되어 카운트만 증가하지 않고 시간도 함께 바뀌는 것**을 확인할 수 있다.
    - 예제 코드
        
        ```jsx
        import React, { useState, createContext, useContext, useEffect } from "react";
        import ReactDOM from "react-dom";
        import moment from "moment";
        
        import "./styles.css";
        
        const CountContext = createContext(null);
        
        function Reset() {
          const { setCount } = useCountContext();
        
          return (
            <div className="app-col">
              <button onClick={() => setCount(0)}>Reset count</button>
              <div>Last reset: {moment().format("h:mm:ss a")}</div>
            </div>
          );
        }
        
        function Button() {
          const { count, setCount } = useCountContext();
        
          return (
            <div className="app-col">
              <button onClick={() => setCount(count + 1)}>Increment</button>
              <div>Current count: {count}</div>
            </div>
          );
        }
        
        function useCountContext() {
          const context = useContext(CountContext);
          if (!context)
            throw new Error(
              "useCountContext has to be used within CountContextProvider"
            );
          return context;
        }
        
        function CountContextProvider({ children }) {
          const [count, setCount] = useState(0);
          return (
            <CountContext.Provider value={{ count, setCount }}>
              {children}
            </CountContext.Provider>
          );
        }
        
        function App() {
          return (
            <div className="App">
              <CountContextProvider>
                <Button />
                <Reset />
              </CountContextProvider>
            </div>
          );
        }
        
        ReactDOM.render(<App />, document.getElementById("root"));
        ```
        

→ 이렇게 컴포넌트를 쓰지 않는 값의 업데이트로 인해 불필요하게 렌더링되는 것을 막기 위해 여러 Provider로 쪼개는 방법을 사용할 수 있다.

---

## +) Hooks

 각 자식 컴포넌트에서 `useContext`를 직접 import해서 사용하는 대신, **필요로 하는 컨텍스트를 직접 반환하는 훅**을 구현할 수 있다.

```jsx
function **useThemeContext()** {
  const theme = useContext(ThemeContext)
  return theme
}
```

 이렇게 해두면, 하위 컴포넌트들은 `ThemeContext`에 접근하기 위해 아래 코드처럼 `useThemeContext` 훅을 사용할 수 있다.

```jsx
export default function ListItem() {
  **const theme = useThemeContext()**

  return <li style={theme.theme}>...</li>
}
```