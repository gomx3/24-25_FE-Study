# HOOK 패턴

Hooks패턴은 React 16.8버전에 추가된 기능입니다. Hooks가 디자인 패턴이 아닐순 있지만, 여러 전통적인 디자인 패턴들을 모두 Hooks로 변경할 수 있는 등 Hooks는 앱에서 아주 중요한 역할을 합니다.

### 1. 사용이유

- React의 상태와 생명 주기 함수들을 ES2015의 클래스를 사용하지 않고 쓸 수 있게 해주기 때문입니다 .
    
    ```jsx
    // Class를 사용했을 때 
    
    class Example extends React.Component {
      constructor(props) {
        super(props);
        this.state = {
          count: 0
        };
      }
      render() {
        return (
          <div>
            <p>You clicked {this.state.count} times</p>
            <button onClick={() => this.setState({ count: this.state.count + 1 })}>
              Click me
            </button>
          </div>
        );
      }
    }
    
    ```
    
    ```jsx
    // Hook을 사용했을 때 
    
    import React, { useState } from 'react';
    function Example() {
      const [count, setCount] = useState(0);
      return (
        <div>
          <p>You clicked {count} times</p>
          <button onClick={() => setCount(count + 1)}>
           Click me
          </button>
        </div>
      );
    }
    ```
    

### 2. 클래스 컴포너트에 대해 알아보자

- Hooks가 추가되기 전, React에서 상태와 생명 주기 함수를 사용하려면 아래의 예시와 같이 클래스 컴포넌트를 꼭 사용해야했습니다.
    
    ```jsx
    class MyComponent extends React.Component {
      /* Adding state and binding custom methods */
      constructor() {
        super()
        this.state = { ... }
    
        this.customMethodOne = this.customMethodOne.bind(this)
        this.customMethodTwo = this.customMethodTwo.bind(this)
      }
    
      /* Lifecycle Methods */
      componentDidMount() { ...}
      componentWillUnmount() { ... }
    
      /* Custom methods */
      customMethodOne() { ... }
      customMethodTwo() { ... }
    
      render() { return { ... }}
    ```
    
    - 클래스 컴포넌트는 생성자에서 상태를 선언한뒤, **componentDidMount와 componentWillUnmout** 같은 생명 주기 메서들들을 생명 주기를 기준으로 사이드 이펙트를 발생시켰습니다. 그 외에 추가 동작을 위한 메서드들을 선언할 수 있었습니다.

### 2.1 ES2015 클래스란

- 앞서 말했듯이, hooks가 추가되기 전에는 상태/생명 주기 메서드를 쓰려면 클래스 컴포넌트로 만들어야 했기 때문에 해당 기능을 쓰기 위해서는 종종 함수형 컴포넌트를 클래스형 컴포넌트로 리펙토링 해야만 했습니다.

 

- ‘div’로 만든 버튼을 예로 들어보겠습니다.
    
    ```jsx
    function Button() {
      return <div className="btn">disabled</div>
    }
    ```
    
    - 사용자가 버튼을 클릭했을 때에는 ‘enabled’로 보이도록 바뀌는 버튼을 만들기 위해 CSS를 추가하고 싶다면,
        - 버튼이 enabled인지 disabled인지 상태를 유지해야 합니다.
        - 따라서, 위와 같은 함수형 컴포넌트를 클래스형 컴포넌트로 리팩토링하고 상태를 가지도록 리팩토링해야 합니다.
        
        ```jsx
        export default class Button extends React.Component {
          constructor() {
            super()
            this.state = { enabled: false }
          }
        
          render() {
            const { enabled } = this.state
            const btnText = enabled ? 'enabled' : 'disabled'
        
            return (
              <div
                className={`btn enabled-${enabled}`}
                onClick={() => this.setState({ enabled: !enabled })}
              >
                {btnText}
              </div>
            )
          }
        }
        ```
        
        - 위 예제는 리팩토링 과정이 간단하지만, 실무에서 사용되는 컴포넌트를 이렇게 리펙토링 하기는 매우 까다롭습니다. 리팩토링 과정에서 동작을 의도치 않게 변경하지 않으려면 ES2015클래스에 대해 알고 있어야 하기 때문입니다.

### 2.2 재설계

- 여러 컴포넌트에서 코드를 공유하기 위해 HOC패턴이나 Render Prop 패턴을 사용하지만, 나중에 이런 패턴들을 도입하려고 할때는 구조를 재설계해야 할 수도 있습니다.
- 또한, 컴포넌트의 크기가 클수록 앱을 재구성하기가 까다로우며, 컴포넌트를 많이 래핑하다보면 Wrapper Hell이라는 안티 패턴이 나타날 수 있습니다.
    - **Wrapper Hell**은 React나 다른 컴포넌트 기반 프레임워크에서 발생하는 안티패턴으로, 컴포넌트 구조가 너무 깊게 중첩되면서 코드의 가독성과 유지보수성이 떨어지는 상황을 말합니다.  특히 HOC, Context API를 많이 사용할 경우에 발생합니다.
        
        ```jsx
        <WrapperOne>
          <WrapperTwo>
            <WrapperThree>
              <WrapperFour>
                <WrapperFive>
                  <Component>
                    <h1>Finally in the component!</h1>
                  </Component>
                </WrapperFive>
              </WrapperFour>
            </WrapperThree>
          </WrapperTwo>
        </WrapperOne>
        ```
        
        - 이런 Wrapper Hell은 앱 내에서 데이터가 어떻게 흘러가는지 파악하기 어렵게 만들 뿐더러, 어떤 동작이 이뤄지고 있는지도 알기 어렵게 만듭니다.

### 2.3 복잡도

- 클래스 컴포넌트에 로직을 추가할수록 컴포넌트의 크기는 빠르게 증가합니다. 그럴수록 로직들은 서로 얽히고 분리하기 점점 어려워져서 디버깅과 성능 최척화에도 어려움을 갖게 합니다.
    - 아래의 예제 코드에서도 생명주기 메서드들이 꽤 많은 코드의 중복을 만들어내는 것을 볼 수 있습니다.
    
    ```jsx
    import React from 'react'
    import './styles.css'
    
    import { Count } from './Count'
    import { Width } from './Width'
    
    export default class Counter extends React.Component {
      constructor() {
        super()
        this.state = {
          count: 0,
          width: 0,
        }
      }
    
      componentDidMount() {
        this.handleResize()
        window.addEventListener('resize', this.handleResize)
      }
    
      componentWillUnmount() {
        window.removeEventListener('resize', this.handleResize)
      }
    
      increment = () => {
        this.setState(({ count }) => ({ count: count + 1 }))
      }
    
      decrement = () => {
        this.setState(({ count }) => ({ count: count - 1 }))
      }
    
      handleResize = () => {
        this.setState({ width: window.innerWidth })
      }
    
      render() {
        return (
          <div className="App">
            <Count
              count={this.state.count}
              increment={this.increment}
              decrement={this.decrement}
            />
            <div id="divider" />
            <Width width={this.state.width} />
          </div>
        )
      }
    }
    
           : Window width
           : Counter functionality
    ```
    
    - 복잡하게 얽힌 로직 뿐만 아니라 일부 로직은 생명주기 함수 내에서 중복되어있습니다. **componentDidMount와 componentWillUnmount** 양쪽에서 윈도우의 resize이벤트를 기준으로 동작을 커스터마이징 하고 있습니다.

### 3. Hooks

- 이렇게 개발자가 클래스 컴포넌트를 개발할 때 겪는 문제들을 해결하기 위해 React는 Hooks를 추가했습니다.
- Hooks는 컴포넌트의 **상태**와 **라이프사이클 메서드를** 관리할 수 있는 함수입니다. React Hooks은 다음의 항목들을 가능하게 합니다.
    
    <aside>
    💡 함수형 컴포넌트에 상태를 추가합니다.
    
    </aside>
    
    <aside>
    💡 **componentDidMount**  혹은 **componentWillUnmount** 와 같은 생명주기 메서드 없이도 컴포넌트의 생명주기를 관리할 수 있습니다.
    
    </aside>
    
    <aside>
    💡 앱 내에 상태를 가진 로직을 여러 컴포넌트에서 재사용할 수 있게 합니다.
    
    </aside>
    

### 3.1 Hooks를 사용해 함수형 컴포넌트에 상태를 추가하는 방법

### **1. Sate Hook**

- 상태를 관리할 수 있도록 **useState** 훅을 제공합니다.
    - input요소 하나를 렌더링하고, 사용자의 타이핑을 상태에 업데이트하는 예제
        - 클래스 컴포넌트를 사용했을때
            
            ```jsx
            class Input extends React.Component {
              constructor() {
                super()
                this.state = { input: '' }
            
                this.handleInput = this.handleInput.bind(this)
              }
            
              handleInput(e) {
                this.setState({ input: e.target.value })
              }
            
              render() {
                ;<input onChange={handleInput} value={this.state.input} />
              }
            }
            ```
            
        - useState 훅을 사용했을 때
            
            ```jsx
            import React, { useState } from "react";
            
            export default function Input() {
              const [input, setInput] = useState("");
            
              return (
                <input
                  onChange={e => setInput(e.target.value)}
                  value={input}
                  placeholder="Type something..."
                />
              );
            }
            
               const [input, setInput] 
            =  const [상태의 현재 값, 상태를 업데이트 할 수 있는 함수]
            
            input == this.state.[value]
            setInput == this.setState
            
            ```
            

### **2. Effect Hook**

- useState를 통해 함수형 컴포넌트 내에서 상태를 다룰 수 있었다면, **useEffect** 훅을 사용하면 **컴포넌트의 생명주기**를 다룰 수 있습니다.
    - 예로, State hook section에서 다루었던 Input 예제를 활용해보겠습니다.사용자가 input요소에 포커스를 둔 채로 타이핑을 시작하면 이 값을 콘솔에 출력하려고 합니다.
    
    ```jsx
    import React, { useState, useEffect } from "react";
    
    export default function Input() {
      const [input, setInput] = useState("");
    
      useEffect(() => {
        console.log(`The user typed ${input}`);
      }, [input]);
    
      return (
        <input
          onChange={e => setInput(e.target.value)}
          value={input}
          placeholder="Type something..."
        />
      );
    }
    
    input을 useEffect hook의 의존 배열에 추가함으로써,
    useEffect가 input값을 지켜보도록하게 되는 것입니다.
    ```
    

### 3.1 Custom Hooks

- React가 제공하는 빌트인 훅들

:**`useState`**, **`useEffect`**, **`useReducer`**, **`useRef`**, **`useContext`**, **`useMemo`**, **`useImperativeHandle`**, **`useLayoutEffect`**, **`useDebugValue`**, **`useCallback'`**  을 이용하여 커스텀 훅을 직접 만들 수도 있습니다. 

*모든 훅이 `use` 로 시작하는 것을 볼 수 있습니다. [https://legacy.reactjs.org/docs/hooks-rules.html](https://legacy.reactjs.org/docs/hooks-rules.html)에 따라 모든 훅들은 `use` 로 시작해야 합니다.

- 예) 사용자 input에 타이핑할 때 hook의 인자로 넘어온 키를 받을 수 있도록 구현해보겠습니다.

```jsx
import React from "react";
import useKeyPress from "./useKeyPress";

export default function Input() {
  const [input, setInput] = React.useState("");
  const pressQ = useKeyPress("q");
  const pressW = useKeyPress("w");
  const pressL = useKeyPress("l");

  React.useEffect(() => {
    console.log(`The user pressed Q!`);
  }, [pressQ]);

  React.useEffect(() => {
    console.log(`The user pressed W!`);
  }, [pressW]);

  React.useEffect(() => {
    console.log(`The user pressed L!`);
  }, [pressL]);

  return (
    <input
      onChange={e => setInput(e.target.value)}
      value={input}
      placeholder="Type something..."
    />
  );
}

```

```jsx
import React from "react";

export default function useKeyPress(targetKey) {
  const [keyPressed, setKeyPressed] = React.useState(false);

  function handleDown({ key }) {
    if (key === targetKey) {
      setKeyPressed(true);
    }
  }

  React.useEffect(() => {
    window.addEventListener("keydown", handleDown);

    return () => {
      window.removeEventListener("keydown", handleDown);
    };
  }, []);

  return keyPressed;
}

```

- 키 입력 감지 로직을 Input컴포넌트 안에 넣는 대신 useKeyPress 훅을 활용해 여러 컴포넌트에 기능을 제공할 수 있게 하였

### 4. 정리하며

-Hook의 최대 장점은 개발자들이 만든 Hook들을 서로 공유할 수 있다는 점입니다. 이미 구현하여 공유되고 있는 Hook들이 많습니다.

[GitHub - streamich/react-use: React Hooks — 👍](https://github.com/streamich/react-use)

[Collection of React Hooks](https://nikgraf.github.io/react-hooks/)

[useHooks – The React Hooks Library](https://usehooks.com/)

- 위에서 클래스 컴포넌트를 이용해 구현했던 코드를 hook으로 리펙토링 해보겠습니다.

```jsx
import React from 'react'
import './styles.css'

import { Count } from './Count'
import { Width } from './Width'

export default class Counter extends React.Component {
  constructor() {
    super()
    this.state = {
      count: 0,
      width: 0,
    }
  }

  componentDidMount() {
    this.handleResize()
    window.addEventListener('resize', this.handleResize)
  }

  componentWillUnmount() {
    window.removeEventListener('resize', this.handleResize)
  }

  increment = () => {
    this.setState(({ count }) => ({ count: count + 1 }))
  }

  decrement = () => {
    this.setState(({ count }) => ({ count: count - 1 }))
  }

  handleResize = () => {
    this.setState({ width: window.innerWidth })
  }

  render() {
    return (
      <div className="App">
        <Count
          count={this.state.count}
          increment={this.increment}
          decrement={this.decrement}
        />
        <div id="divider" />
        <Width width={this.state.width} />
      </div>
    )
  }
}

       : Window width
       : Counter functionality
```

```jsx
import React, { useState, useEffect } from "react";
import "./styles.css";

import { Count } from "./Count";
import { Width } from "./Width";

function useCounter() {
  const [count, setCount] = useState(0);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);

  return { count, increment, decrement };
}

function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    window.addEventListener("resize", handleResize);
    return () => window.addEventListener("resize", handleResize);
  });

  return width;
}

export default function App() {
  const counter = useCounter();
  const width = useWindowWidth();

  return (
    <div className="App">
      <Count
        count={counter.count}
        increment={counter.increment}
        decrement={counter.decrement}
      />
      <div id="divider" />
      <Width width={width} />
    </div>
  );
}

       : Window width
       : Counter functionality
```

- 형광펜만 봐도 hook을 사용했을때 컴포넌트가 훨씬 명확하고 작은 조각으로 분리되어 가독성도 좋고, 재사용하기도 훨씬 수월한 것을 확인할 수 있습니다.

### **일반적인 훅에 대한 요약**

- **useState**

:  함수형 컴포넌트 내에서 **상태를 관리하려 할 때** 클래스형 컴포넌트로 변경하지 않아도 이를 가능케 하는 훅입니다. 다른 훅들에 비해 사용법이 단순합니다.

- **useEffect**

: 코드를 **컴포넌트의 생명 주기에** **실행하기 위한 훅**입니다. 컴포넌트 함수 본문에서 변경을 하거나 뭔가를 구독하거나 타이머를 만들거나 로깅을 하는 등의 사이드 이펙트를 발생시키는것은 금지하고 있습니다. 허용될 경우 버그가 생기거나 모델과 뷰가 일치되지 않는 상황이 생길 수 있기 때문입니다. useEffect는 이런 사이드 이펙트가 발생하는것을 예방하고 UI가 부드럽게 동작하도록 한다. **`componentDidMount`**, **`componentDidUpdate`**, **`componentWillUnmount`**를 합친 것과 유사합니다.

- **useContext**

: 컨텍스트 객체를 받아서 provider에 넘겼던 값을 반환합니다. React 의 Context API와 함께 동작하여 앱 전반적으로 공유하는 데이터를 prop drilling 없이 사용할 수 있게 해줍니다.

- 주의해야 할 점은 **`useContext`**의 인자로는 컨텍스트 객체를 넘겨야 하고, **`useContext`**를 사용하는 컴포넌트는 컨텍스트 업데이트 시 마다 리렌더링 된다는 점입니다.
- **useReducer**

: **`setState`**대신 사용할 수 있으며 특히 이전 상태에서 다음 상태로 변경될 때 복잡한 상태 로직과 파생되는 변수들을 만들어 내야 하는 경우 더 선호됩니다. **`reducer`** 함수와 초기 값을 인자로 받아 초기값과 **`dispatch`**함수를 반환합니다. **`useReducer`** 는 또한 컴포넌트의 복잡한 상태에서 깊이 선언된 값을 변경할 때의 성능에 최적화되어 있습니다.

### **Hook의 장점과 단점**

- **장점**
    - 코드가 간결해집니다. 생명주기와 얽히지 않으며 코드들은 관심사와 기능에 따라 분류할 수 있기 때문입니다.
    
    ```jsx
    //Class 컴포넌트를 사용했을 때
    class TweetSearchResults extends React.Component {
      constructor(props) {
        super(props)
        this.state = {
          filterText: '',
          inThisLocation: false,
        }
    
        this.handleFilterTextChange = this.handleFilterTextChange.bind(this)
        this.handleInThisLocationChange = this.handleInThisLocationChange.bind(this)
      }
    
      handleFilterTextChange(filterText) {
        this.setState({
          filterText: filterText,
        })
      }
    
      handleInThisLocationChange(inThisLocation) {
        this.setState({
          inThisLocation: inThisLocation,
        })
      }
    
      render() {
        return (
          <div>
            <SearchBar
              filterText={this.state.filterText}
              inThisLocation={this.state.inThisLocation}
              onFilterTextChange={this.handleFilterTextChange}
              onInThisLocationChange={this.handleInThisLocationChange}
            />
            <TweetList
              tweets={this.props.tweets}
              filterText={this.state.filterText}
              inThisLocation={this.state.inThisLocation}
            />
          </div>
        )
      }
    }
    ```
    
    ```jsx
    //Hookd을 사용했을 때
    const TweetSearchResults = ({ tweets }) => {
      const [filterText, setFilterText] = useState('')
      const [inThisLocation, setInThisLocation] = useState(false)
      return (
        <div>
          <SearchBar
            filterText={filterText}
            inThisLocation={inThisLocation}
            setFilterText={setFilterText}
            setInThisLocation={setInThisLocation}
          />
          <TweetList
            tweets={tweets}
            filterText={filterText}
            inThisLocation={inThisLocation}
          />
        </div>
      )
    }
    ```
    
    - 복잡한 컴포넌트를 단순화 해줍니다.
        
        : 자바스크립트의 클래스는 관리하기 힘들고 hot reloading과 함께 쓰기 어려우며 minifiy도 잘 되지 않습니다. 또한, 상태 로직이 있는 자바스크립트의 클래스를 사용하는 것은 여러 단계의 상속 구현을 유도하고 전체적인 복잡도를 빠르게 증가시키며 에러를 발생하기 쉽게 만들지만, 훅은 상태를 만드는 것 뿐만 아니라 React의 기능들을 클래스 없이 작성할 수 있습니다. Hook을 사용하면 상태를 가진 로직을 계속 재작성하지 않고 재사용할 수 있습니다. 에러를 발생시킬 확률을 낮추며 일반 함수를 조립해서 쓸 수 있게 해 줍니다.
        
        훅이 구현되기 전에는 React에서 뷰가 없는 로직을 추출해내기가 어려웠지만, 이는 HOC패턴이나 render prop패턴을 사용할 때 복잡도를 증가시키는 원인이 되었습니다. 그렇지만 훅이 추가되면서 상태가 있는 로직을 언제든 자바스크립트 함수로 분리할 수 있게 되었습니다.
        
- **단점**
    - 규칙에 따라 작성해야 합니다. 정적 분석기 플러그인을 사용하지 않으면 어떤 훅이 규칙을 어기고 있는지 알기 어렵습니다.
    - 올바르게 사용하기 위해 익숙하게 쓸 줄 알아야 합니다
    - 잘못 쓸 수 있습니다 (예를 들어 useCallback, useMemo)
