# HOC 패턴

HOC 패턴은 `High Order Component` 로, 즉 고차컴포넌트라고 합니다. https://ko.legacy.reactjs.org/docs/higher-order-components.html 리액트 공식문서를 보면 `고차 컴포넌트는 컴포넌트를 가져와 새 컴포넌트를 반환하는 함수`라고 기술되어있습니다.

좀 더 쉽게 말하면 다른 컴포넌트를 받는 컴포넌트라고 할 수 있습니다. HOC는 인자로 넘긴 컴포넌트에게 추가되길 원하는 로직을 가지고 있고, 로직이 적용된 엘리먼트를 반환하게 되는 것입니다.

예)

```jsx
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

컴포넌트는 Props를 UI로 변환하는 반면에, **고차 컴포넌트**는 컴포넌트를 새로운 컴포넌트로 변환합니다.

### 1. 사용이유

- 종종 여러 컴포넌트에서 같은 로직을 사용해야하는 경우에 많이 쓰입니다. 그렇기 때문에 여러 컴포넌트에서 재사용하는 방법 중 하나입니다.

  - 예를 들면, 컴포넌트의 스타일 시트를 설정하는 경우, 권환을 요청하거나 전역 상태를 추가하는 경우가 될 수도 있습니다.

    - 예1) 여러 컴포넌트에게 동일한 스타일을 적용하고 싶을 때

      ```jsx
      function withStyles(Component) {
        return props => {
          const style = { padding: '0.2rem', margin: '1rem' }
          return <Component style={style} {...props} />
        }
      }

      const Button = () = <button>Click me!</button>
      const Text = () => <p>Hello World!</p>

      const StyledButton = withStyles(Button)
      const StyledText = withStyles(Text)

      ```

      => StyledButton와 StyledText, 이 두 컴포넌트는 모두withStyles HOC로부터 스탕일링 로직이 적용되었다고 할 수 있습니다.

### 2. 예시로 보는 HOC

### **[1. 원래 코드] 강아지 사진 목록을 API로부터 받아와 렌더링 하기**

```jsx
//index.js

import React from "react";
import { render } from "react-dom";
import DogImages from "./DogImages";
import "./styles.css";

function App() {
  return (
    <div className="App">
      <DogImages />
    </div>
  );
}

render(<App />, document.getElementById("root"));
```

```jsx
//DogImage.js

import React from "react";
import useDogImages from "./useDogImages";

export default function DogImages() {
  const dogs = useDogImages();

  return dogs.map((dog, i) => <img src={dog} key={i} alt="Dog" />);
}
```

```jsx
//useDogImage.js

import { useState, useEffect } from "react";

export default function useDogImages() {
  const [dogs, setDogs] = useState([]);

  useEffect(() => {
    async function fetchDogs() {
      const res = await fetch(
        "https://dog.ceo/api/breed/labrador/images/random/6"
      );
      const { message } = await res.json();
      setDogs(message);
    }

    fetchDogs();
  }, []);

  return dogs;
}
```

### [2. 개선 1 ] 데이터를 받아오는 중에는 “로딩 중…”이라는 메시지 띄우기

- withLodaer HOC만들기

  ```jsx
  //DogImage.js

  import React from "react";
  import withLoader from "./withLoader";

  function DogImages(props) {
    return props.data.message.map((dog, index) => (
      <img src={dog} alt="Dog" key={index} />
    ));
  }

  export default withLoader(
    DogImages,
    "https://dog.ceo/api/breed/labrador/images/random/6"
  );
  ```

  - DogImage.js 에서 더이상 DogImages 컴포넌트를 직접 export할 필요 없습니다
  - 대신 ,withLoader HOC로 감싸진 DogImages 컴포넌트를 export 하면 됩니다.

  ```jsx
  //withLoader.js

  import React, { useEffect, useState } from "react";

  export default function withLoader(Element, url) {
    return (props) => {
      const [data, setData] = useState(null);

      useEffect(() => {
        async function getData() {
          const res = await fetch(url);
          const data = await res.json();
          setData(data);
        }

        getData();
      }, []);

      if (!data) {
        return <div>Loading...</div>;
      }

      return <Element {...props} data={data} />;
    };
  }
  ```

  - withLoader HOC는 데이터를 prop으로 전달하고 있기 때문에 이것을 통해 강아지 사진 목록을 사용할 수 있습니다.

⇒ withLoader HOC는 컴포넌트와 url에서 받아오는 데이터에는 관여하지 않는다.

⇒ 컴포넌트가 유효하고 API엔드포인트도 정상인 경우 단순히 API호출을 통해 받아온 데이터를 넘길 뿐이다.

### [3. Composing] 이미지(DogImages) 컴포넌트에 마우스를 올리면 ‘호버링!’이라는 텍스트 박스 띄우기

- ‘hovering’이라는 prop를 제공하는 HOC를 만들어야 한다.

  ```jsx
  //DogImages.js

  import React from "react";
  import withLoader from "./withLoader";
  import withHover from "./withHover";

  function DogImages(props) {
    return (
      <div {...props}>
        {props.hovering && <div id="hover">Hovering!</div>}
        <div id="list">
          {props.data.message.map((dog, index) => (
            <img src={dog} alt="Dog" key={index} />
          ))}
        </div>
      </div>
    );
  }

  export default withHover(
    withLoader(DogImages, "https://dog.ceo/api/breed/labrador/images/random/6")
  );
  ```

  ```jsx
  //withHover.js

  import React, { useState } from "react";

  export default function withHover(Element) {
    return (props) => {
      const [hovering, setHover] = useState(false);

      return (
        <Element
          {...props}
          hovering={hovering}
          onMouseEnter={() => setHover(true)}
          onMouseLeave={() => setHover(false)}
        />
      );
    };
  }
  ```

  - DogImages element는 이제 withhover와 withLodaer에서 제공하는 prop을 사용할 수 있습니다

> HOC를 사용하는 유명 오픈소스 라이브러리에는 recompose 가 있다. 나중에 혹시 HOC가 훅으로 완전 대체가 가능해 진다면 이 라이브러리는 더 이상 사용되지 않을것이다. 이 글도 마찬가지이다.
> by) https://patterns-dev-kr.github.io/design-patterns/hoc-pattern/

### [4. Hooks] HOC 패턴을 React Hook으로 대체하기

- 위의 withHover HOC를 useHover hook으로 리펙토링 하기
  - 고차 컴포넌트를 사용하는 대신 엘리먼트에 mouseOver, mouseLeave 이벤트 핸들러를 추가할 것입니다.
  - HOC처럼 엘리먼트를 반환할 수 없으니 ref를 반환하여 이벤트 핸들러를 추가할 엘리먼트를 지정할 수 있습니다

```jsx
import React from "react";
import withLoader from "./withLoader";
import useHover from "./useHover";

function DogImages(props) {
  const [hoverRef, hovering] = useHover();

  return (
    <div ref={hoverRef} {...props}>
      {hovering && <div id="hover">Hovering!</div>}
      <div id="list">
        {props.data.message.map((dog, index) => (
          <img src={dog} alt="Dog" key={index} />
        ))}
      </div>
    </div>
  );
}

export default withLoader(
  DogImages,
  "https://dog.ceo/api/breed/labrador/images/random/6"
);
```

- DogImages 컴포넌트를 감싸는 대신 useHover hook을 직접 사용하여 기능을 구현할 수 있습니다.

```jsx
//useHover.js

import { useState, useRef, useEffect } from "react";

export default function useHover() {
  const [hovering, setHover] = useState(false);
  const ref = useRef(null);

  const handleMouseOver = () => setHover(true);
  const handleMouseOut = () => setHover(false);

  useEffect(() => {
    const node = ref.current;
    if (node) {
      node.addEventListener("mouseover", handleMouseOver);
      node.addEventListener("mouseout", handleMouseOut);

      return () => {
        node.removeEventListener("mouseover", handleMouseOver);
        node.removeEventListener("mouseout", handleMouseOut);
      };
    }
  }, [ref.current]);

  return [ref, hovering];
}
```

### 3. Hook과 HOC

- 일반적으로 React Hook은 HOC 패턴을 완전 대체할 수 없지만, 대부분의 경우에서 React Hook이 tree가 깊어지는 상황을 줄일 수 있습니다. HOC 패턴을 사용하면 컴포넌트의 tree가 깊어지는 경우가 있기 때문입니다.
  ```jsx
  <withAuth>
    <withLayout>
      <withLogging>
        <Component />
      </withLogging>
    </withLayout>
  </withAuth>
  ```
  - 컴포넌트 내에서 훅을 직접 사용하면 더 이상 컴포넌트를 래핑하지 않아도 됩니다
- HOC를 활용하면 동일한 로직을 한 군데 구현하여 여러 컴포넌트를 제공할 수 있습니다.
  - 활용사례
    - 앱 전반적으로 동일하며 커스터마이징 불가한 동작이 여러 컴포넌트에 필요한 경우
    - 컴포넌트가 커스텀 로직 추가 없이 단독으로 동작할 수 있어야 하는 경우
- Hook은 내부에서 특정한 동작을 추가할 수 있게 해줍니다
  - 하지만 HOC에 비해 버그를 발생시킬 확률이 높습니다
  - 활용사례
    - 공통 기능이 각 컴포넌트에서 쓰이기 전에 커스터마이징 되어야 하는 경우
    - 공통 기능이 앱 전반적으로 쓰이는 것이 아닌 하나나 혹은 몇개의 컴포넌트에서 요구되는 경우
    - 해당 기능이 기능을 쓰는 컴포넌트에게 여러 프로퍼티를 전달해야 하는 경우

### 4. HOC의 장점과 단점

- **장점**
  - 한 곳에 구현한 로직들을 여러 컴포넌트에서 재사용할 수 있습니다. 따라서 버그를 만들어 낼 확률도 줄일 수 있습니다.
  - 로직을 한 곳에서 관리하여 코드를 DRY하면서 관심사의 분리도 적용할 수 있게 됩니다.
- **단점**

  - HOC가 반환하는 컴포넌트에 전달하는 props의 이름이 겹칠 수 있습니다.

    ```jsx
    function withStyles(Component) {
      return props => {
        const style = { padding: '0.2rem', margin: '1rem' }
        return <Component style={style} {...props} />
      }
    }

    const Button = () = <button style={{ color: 'red' }}>Click me!</button>
    const StyledButton = withStyles(Button)
    ```

    - 이 경우 똑같은 style이라는 prop을 가지고 있기 때문에 덮어쓰게 될 것입니다. 따라서 HOC를 만들 땐 이런 상황을 고려하여 prop 병합을 통해 아래와 같이 해결할 수 있습니다.

    ```jsx
    function withStyles(Component) {
      return props => {
        const style = {
          padding: '0.2rem',
          margin: '1rem',
          ...props.style
        }

        return <Component style={style} {...props} />
      }
    }

    const Button = () = <button style={{ color: 'red' }}>Click me!</button>
    const StyledButton = withStyles(Button)
    ```

    - HOC를 여러번 조합하여 사용하는 경우 모든 prop이 안에서 병합되므로 어떤 HOC가 어떤 props에 관련이 있는지 파악하기 어렵습니다. 따라서 앱의 디버깅이나 규모를 키울 떄 방해가 될 수 있습니다.
