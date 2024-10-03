### HOC 예시 더 알아보기

```jsx
import React from "react";

const withLoading = (Component) => {
  return function WithLoadingComponent({ isLoading, ...props }) {
    if (isLoading) {
      return <div>Loading...</div>;
    }
    return <Component {...props} />;
  };
};

export default withLoading;
```

```jsx
import React from "react";

const ComponentToEnhance = ({ data }) => {
  return (
    <div>
      {data.map((item) => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
};

export default ComponentToEnhance;
```

```jsx
import React from "react";
import withLoading from "./withLoading";
import ComponentToEnhance from "./ComponentToEnhance";

const EnhancedComponent = withLoading(ComponentToEnhance);

const App = () => {
  const data = [
    { id: 1, name: "Item 1" },
    { id: 2, name: "Item 2" },
  ];
  const isLoading = false;

  return <EnhancedComponent isLoading={isLoading} data={data} />;
};

export default App;
```

- withLoading을 다른 컴포넌트에서 재사용할 수 있다.
- ComponentToEnhance는 본연의 기능에만 집중할 수 있기 때문에, withLodaing과의 역할이 분리된다.

⇒ 공통된 로직을 여러 컴포넌트에 적용할 수 있고, 컴포넌트 간의 역할을 명확히 분리하여 더 효율적이고 유지보수가 쉬운 코드를 작성할 수 있도록 도운다.
