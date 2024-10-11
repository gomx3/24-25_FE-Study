## Proxy 패턴 활용 예제: 비밀번호 접근 제한 ! ! !

[참고] https://gist.github.com/superLipbalm/dea43695b66897d1b4d246b402aab320

 관리자 권한을 가지고 있는지 여부에 따라 서버로부터 받아 온 **사용자 프로필 정보에 대한 접근을 제한하는 기능**을 구현한다. 

- 관리자 권한을 가지고 있을 경우(**true**), 사용자의 비밀번호를 보여준다.
- 관리자 권한을 가지고 있지 않을 경우(**false**), 사용자의 비밀번호를 보여주지 않는다.

 비밀번호 접근 제한 기능을 사용자 데이터 객체에 직접 추가하지 않고, Proxy 객체를 따로 생성해 추가한다.

 기존 코드를 변경하지 않고 새로운 기능을 추가하는 기능(OCP)을 하는 Proxy 패턴의 활용 사례를 보여주면서, 기존 객체와 사용자의 요청 사이에 위치해 있기 때문에 보안 역할을 강화하는 특징을 보여주는 예제이다.

![image](https://github.com/user-attachments/assets/41f82ac7-c649-4c82-95d0-f00db524345c)

![image](https://github.com/user-attachments/assets/1934d306-bb2b-4e95-bef0-e34735d15732)

 사진과 같이 관리자는 사용자의 비밀번호를 볼 수 있고(왼쪽), 관리자가 아닌 경우 비밀번호 접근 제한이 거절된다(오른쪽).

- hooks/useSecureuserData.js
    
    ```jsx
    import { useEffect, useState } from 'react';
    
    const fetchUserData = () => {
      // 실제로는 API 호출을 통해 사용자 데이터를 받아 온다.
      console.log('Data fetching...');
    
      return {
        username: 'gildong123',
        password: 'mypassword',
      };
    };
    
    const useSecureUserData = () => {
      const [userData, setUserData] = useState(null);
    
      **const isUserAdmin = () => {
        // 관리자 권한을 true로 가정한다.
        // 실제로는 사용자의 권한을 확인하는 로직이 필요하다.
        return true;
      };**
    
      useEffect(() => {
        // ==== Proxy 객체 생성! ====
        const userProxy = new Proxy(fetchUserData(), {
          // 비밀번호 접근 핸들러 -> 관리자인지 확인하고 권한이 있는 경우에만 보여준다.
          get(target, prop) {
            if (prop === 'password' && !isUserAdmin()) {
              console.warn('Unauthorized access to password!');
              return null; // 권한이 없을 경우 null을 반환
            }
            //return target[prop];
            return Reflect.get(target, prop);
          },
        });
    
        setUserData(userProxy);
      }, []);
    
      return userData;
    };
    
    export default useSecureUserData;
    ```
    
- components/UserProfile.js
    
    ```jsx
    import React from 'react';
    import '../App.css';
    import useSecureUserData from '../hooks/useSecureUserData'
    
    const UserProfile = () => {
      const userData = useSecureUserData();
    
      if (!userData) {
        return <div>Loading user data...</div>;
      }
    
      return (
        <div className="user-profile">
          <p><strong>Username:</strong> {userData.username}</p>
          <p><strong>Password:</strong> 
    	      {userData.password ? userData.password : 'Access Denied'}
    	    </p>
        </div>
      );
    };
    
    export default UserProfile;
    ```
    
- App.js
    
    ```jsx
    import React from 'react';
    import './App.css';
    import UserProfile from './components/UserProfile';
    
    function App() {
      return (
        <div className="App">
          <h1>User Profile</h1>
          <UserProfile />
        </div>
      );
    }
    
    export default App;
    ```