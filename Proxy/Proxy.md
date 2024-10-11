# 2. Proxy 패턴

이름: 김소원
상태: 완료
회차: 1회차

## Proxy란?

 기본적으로 *‘대리인’* 이라는 의미로, 어떤 객체를 직접 다루는 것이 아니라 Proxy를 통해 **대신** 객체에 접근하는 것을 말한다.

 여기 홍길동이라는 사람을 나타내는 객체 `person`이 있다.

```jsx
const person = {
	name: 'Hong Gil Dong',
	age: 42,
	nationality: 'Korean',
};
```

 이 객체를 직접 참조해서 다루지 않고, 이 객체를 대신하는 Proxy 객체를 통해 접근할 것이다. Proxy는 `Proxy` 인스턴스를 만들어 쉽게 객체를 생성할 수 있다.

```jsx
const personProxy = new Proxy(person, {})
```

## Proxy 객체 핸들러 추가

 Proxy 클래스의 **두 번째 인자는** **핸들러**를 의미한다. 객체를 다루는 특정 동작을 정의할 수 있다. 대표적으로 `get`이나 `set`과 같은 메소드를 추가할 수 있다. 

![image](https://github.com/user-attachments/assets/034b3e0d-d678-4a86-af6f-1c6ba9311981)

- `get`: 프로퍼티에 접근할 때
    
     해당 키와 값에 대한 메시지를 콘솔에 출력할 수 있다.
    

![image](https://github.com/user-attachments/assets/a6cc0c0b-f1be-447b-98ab-0c306063e603)

- `set`: 프로퍼티를 수정할 때
    
     변경 전과 변경 후의 값을 콘솔로 확인할 수 있다.
    

```jsx
const person = {
  name: 'Hong Gil Dong',
  age: 42,
  nationality: 'Korean',
};

const personProxy = new Proxy(person, {
  get: (obj, prop) => {
    console.log(`${prop}의 값은 ${obj[prop]}입니다.`);
  },
    
  set: (obj, prop, value) => {
    console.log(`${prop}가 ${obj[prop]}에서 ${value}로 변경되었습니다.`);
    obj[prop] = value;
    return true;
  }
});

personProxy.name;     // name 값은 Hong Gil Dong입니다.
personProxy.age = 43; // age가 42에서 43로 변경되었습니다.
```

 코드를 실행하면 콘솔에 이렇게 출력된다.

![image](https://github.com/user-attachments/assets/865a65b5-1b23-4b2b-be27-82c0c13f0519)

- `name` prop에 접근할 때, Proxy 객체가 콘솔에 값을 대신 출력한다
- `age` prop에 접근할 때, Proxy 객체가 변경 이전 값과 이후 값을 콘솔에 대신 출력한다

<aside>
💡

일반적으로 Proxy는 **인터페이스 역할**을 하는 클래스를 의미한다!

</aside>

## 유효성 검사 구현

 사용자는 `person` 객체의 `age` 프로퍼티를 문자열로 수정할 수 없고, `name` 프로퍼티로 빈 문자열을 사용할 수 없다. Proxy는 이러한 **유효성 검사**를 구현할 때 유용하다. 또한 사용자가 객체에 존재하지 않는 프로퍼티에 접근하려고 하면, 알려 줄 수 있다.

 Proxy는 `person` 객체를 실수로 수정하는 것을 막아주어 데이터를 안전하게 관리할 수 있도록 돕는 역할을 한다.

 다음과 같이 유효하지 않은 값을 넣어보는 코드가 있을 때,

```jsx
const person = {
  name: 'Hong Gil Dong',
  age: 42,
  nationality: 'Korean',
};

const personProxy = new Proxy(person, {
  get: (obj, prop) => {
    if (!obj[prop]) {
      console.log(`음.. 이 프로퍼티는 존재하지 않습니다.`);
    } else {
      console.log(`${prop}의 값은 ${obj[prop]}입니다.`);
    }
  },
  
  set: (obj, prop, value) => {
    if (prop === "age" && typeof value !== "number") {
      console.log(`age의 값으로는 수치형 값만 사용할 수 있습니다.`);
    } else if (prop === "name" && value.length < 2) {
      console.log(`이름의 글자수가 모자랍니다.`);
    } else {
      console.log(`${prop}이/가 ${obj[prop]}에서 ${value}(으)로 변경되었습니다.`);
      obj[prop] = value;
    }
    return true;
  }
});

personProxy.nonExistentProperty; // 음.. 이 프로퍼티는 존재하지 않습니다.
personProxy.age = "44";          // age의 값으로는 수치형 값만 사용할 수 있습니다.
personProxy.name = "";           // 이름의 글자수가 모자랍니다.
```

 실행하면 콘솔에 이렇게 출력된다. 

![image](https://github.com/user-attachments/assets/7b8ead37-72d9-4319-822f-9d8b01b71f7a)

## Reflect 객체 활용

 위의 예제에서는 Proxy의 핸들러 안에서 괄호 표기를 사용하여 직접 프로퍼티를 수정하거나 읽었다. 하지만, JavaScript가 제공하는 **`Reflect` 빌트인 객체**를 Proxy와 함께 사용하면 대상 객체를 쉽게 조작할 수 있다. 

 `Reflect` 객체는 핸들러 객체와 같은 이름의 메소드를 가질 수 있다.

```jsx
const personProxy = new Proxy(person, {
  get: (obj, prop) => {
	  console.log(`${prop}의 값은 ${Reflect.get(obj, prop)}입니다.`);
  },
  set: (obj, prop, value) => {
    console.log(`${prop}이/가 ${obj[prop]}에서 ${value}(으)로 변경되었습니다.`);
    Reflect.set(obj, prop, value)
  },
})
```

 `obj[prop]`으로 프로퍼티를 직접 다루는 대신, **`Reflect.get()`** 혹은 **`Reflect.set()`**을 활용할 수 있다. 각 메소드는 핸들러의 메소드와 동일한 인자를 사용한다.

## Proxy 장/단점

- 장점
    1. 객체 지향 설계 5 원칙 SOLID 중 **OCP를 만족**한다: 
        
         원본 객체를 **변경하지 않고**, Proxy 객체의 핸들러를 사용해 새로운 메소드를 추가할 수 있다.
        
        <aside>
        💡
        
         OCP(Open-Closed Principle, 개방-폐쇄 원칙)은 ‘소프트웨어 요소는 … 확장에는 열려 있으나 변경에는 닫혀 있어야 한다.’는 원칙을 말한다. 이는 기존 코드를 변경하지 않고 새로운 기능을 추가할 수 있어야 함을 의미한다.
        
        </aside>
        
    2. 객체 지향 설계 5 원칙 SOLID 중 **SRP를 만족**한다:
        
         기존 객체는 원래의 기능에만 집중하고, 그 외 **부가적인 기능**을 제공하는 역할을 Proxy 객체에게 위임하여 다중 책임을 막을 수 있다.
        
        <aside>
        💡
        
        SRP(Single Responsibility Principle, 단일 책임 원칙)은 하나의 클래스나 모듈이 하나의 책임(역할)만을 가져야 함을 의미한다.
        
        </aside>
        
    3. 데이터 유효성 검사
    4. 기존 객체와 사용자의 요청 사이에 위치(인터페이스 역할)해 있기 때문에 보안 역할을 한다.

- 단점
    1. 코드의 복잡도가 증가하여 가독성이 떨어진다.
    2. 객체에 접근할 때 한 단계를 거치게 되므로 서비스의 응답이 느릴 수 있다.

> 결국 Proxy는 특정 객체에 대한 **접근**을 다루거나 **기능을 추가**하고 싶은데 기존 객체를 수정하기 어려운 상황일 때 사용할 수 있다. 
 하지만 너무 무겁게 사용했을 때 성능에 부정적인 영향을 가할 수 있으므로 성능 문제가 발생하지 않을 정도로만 사용하는 것이 중요하다.
> 
