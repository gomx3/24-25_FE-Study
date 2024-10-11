## 패턴 활용 예제

 Provider 패턴은 전역 데이터를 공유하기에 좋다. 주로 ****UI 스타일을 여러 컴포넌트들이 공유해 사용하기 위해 쓴다.

 예제는 사용자가 스위치를 토글해서 라이트/다크모드를 전환할 수 있도록 구현하려는 상황이다. 스위치를 클릭하여 모드를 전환할 때 배경과 텍스트 색상이 변경되어야 한다.

 이때 현재 보이는 UI 스타일 데이터를 직접 전달하는 대신, 컴포넌트들을 `ThemeProvider`로 감싸서 컬러값을 Provider에 전달한다.

```jsx
export const ThemeContext = React.createContext()

// 테마 데이터
const themes = {
  light: {
    background: '#fff',
    color: '#000',
  },
  dark: {
    background: '#171717',
    color: '#fff',
  },
}

export default function App() {
  const [theme, setTheme] = useState('dark')

  function toggleTheme() {
    setTheme(theme === 'light' ? 'dark' : 'light')
  }
	
	// 테마 데이터와 모드를 변경하는 토글 함수 (value)
  const providerValue = {
    theme: themes[theme],
    toggleTheme,
  }

  return (
    <div className={`App theme-${theme}`}>
      <ThemeContext.Provider value={providerValue}>
        <Toggle />
        <List />
      </ThemeContext.Provider>
    </div>
  )
}
```

1. `Toggle`과 `List` 컴포넌트가 `ThemeContext` Provider의 자식 컴포넌트로 존재하는 동안, `value`로 넘긴 테마 데이터와 모드를 변경하는 토글 함수(`toggleTheme`) 값에 접근할 수 있다.

1. `Toggle` 컴포넌트는 `useContext`를 통해 테마 컨텍스트를 불러왔기 때문에 모드 업데이트를 위한 **`toggleTheme` 함수를 직접 호출**할 수 있다.
    
    ```jsx
    import React, { useContext } from "react";
    import { ThemeContext } from "./App";
    
    **export default function Toggle() {
      const theme = useContext(ThemeContext);**
    
      return (
        <label className="switch">
          <input type="checkbox" onClick={theme.toggleTheme} />
          <span className="slider round" />
        </label>
      );
    }
    ```
    
2. `List` 컴포넌트는 현재 테마 값을 사용하지 않지만, `ListItem`은 `useContext`를 통해 테마 컨텍스트를 불러왔기 때문에 **`theme` 데이터를 직접 사용**할 수 있다.
    
    ```jsx
    // List.jsx
    
    import React from "react";
    import ListItem from "./ListItem";
    
    export default function List() {
      return (
        <ul className="list">
          {new Array(10).fill(0).map((x, i) => (
            <ListItem key={i} />
          ))}
        </ul>
      );
    }
    ```
    
    ```jsx
    // ListItem.jsx
    
    import React, { useContext } from "react";
    import { ThemeContext } from "./App";
    
    **export default function ListItem() {
      const theme = useContext(ThemeContext);**
    
      return (
        <li style={theme.theme}>
          Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod
          tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim
          veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
          commodo consequat.
        </li>
      );
    }
    ```
    

 이렇게 테마(UI 스타일)를 사용하지 않는 컴포넌트가 불필요하게 데이터를 받지 않도록 구현할 수 있다. https://codesandbox.io/p/sandbox/provider-2-9djpl?from-embed

![image](https://github.com/user-attachments/assets/61387dd9-c3a7-4299-8388-95afb6697c67)

![image](https://github.com/user-attachments/assets/32381a4f-c633-4ad7-b6ea-ea66244bb767)


💥 참고로, 컴포넌트들을 `ThemeContext.Provider`로 직접 감싸는 대신, HOC를 만들어서 **컨텍스트 로직과 렌더링 로직을 분리**하여 **재사용성을 증가**시킬 수 있다.

```jsx
// HOC를 만들어서 Provider 컨텍스트 생성 로직 분리
function **ThemeProvider**({ children }) {
  const [theme, setTheme] = useState('dark')

  function toggleTheme() {
    setTheme(theme === 'light' ? 'dark' : 'light')
  }

  const providerValue = {
    theme: themes[theme],
    toggleTheme,
  }

  return (
    <ThemeContext.Provider value={providerValue}>
      {children}
    </ThemeContext.Provider>
  )
}

export default function App() {
  return (
    <div className={`App theme-${theme}`}>
      <**ThemeProvider**>
        <Toggle />
        <List />
      </**ThemeProvider**>
    </div>
  )
}
```