---

layout: post
title:  "useContext의 상태 갱신 문제"
date:   2024-03-03 01:38:04 +0900

---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## useContext의 사용 이유
일반적으로 React에서 데이터는 부모에서 자식에게 props를 통해 전달된다.  
그러나 application 전체에 데이터를 전달해야줘야 할 경우 props를 사용한다면  
props chain이 너무 길어지는 문제점이 발생한다.  
따라서 useContext는 이를 해결하기 위한 Hook으로 저장소같은 역할을 한다.  
따라서 명시적으로 props를 전달하지 않고도 전역 데이터를 사용할 수 있다.  

useContext를 통해 로그인한 유저의 정보를 저장하거나  
application에서 사용하는 전역 변수들을 저장소로서 저장하고 데이터 조회 및 변경도 가능하다.  

~~~js
// file: `login-context.js`
import React from "react";

const loginContext = React.createContext({
    id : 0,
    name : '',
    email : '',
    password : '',
    loginUser : (email,password) => {},
    logoutUser : (email,password) => {},
    editNameUser : (newName) => {},
    editEmailUser : (newEmail) => {},
    editPasswordUser : (newPassword) => {}
});

export default loginContext;
~~~
위의 코드는 Context 저장소를 만든 것이다.  
createContext()를 통해 저장소를 생성하고 이를 통해 React Project에서 전역적으로 사용가능하다.  

~~~js
// file: `LoginProvider.js`
import React, { useReducer } from "react";
import loginContext from './login-context';

const defaultLoginUser = {
    id : 0,
    name : '',
    email : '',
    password : '',
}

const loginReducer = (state,action) => {

    if (action.type === "LOGIN"){
        return {
            ...state,
            id : action.id,
            name : action.name,
            email : action.email,
            password : action.password
        }
    }
    ... another logic
}

const LoginProvider = (props) => {

    // useReducer를 통해 저장소의 데이터를 갱신
    const [userState,dispatchUserAction] = useReducer(loginReducer,defaultLoginUser);

    // 상태를 갱신하는 reducer 함수로 전달
    const loginHandler = (id,name,email,password) => {
        dispatchUserAction({
            type : 'LOGIN',
            id : id,
            name : name,
            email : email,
            password : password
        })
    };
    
    ... another logic 

    // 다른 Component에서 사용할 데이터가 저장된 저장소
    const userContext = {
        id : userState.id,
        name : userState.name,
        email : userState.email,
        password : userState.password,
        loginUser : loginHandler,
        logoutUser : logoutHandler,
        editNameUser : editNameHandler,
        editEmailUser : editEmailHandler,
        editPasswordUser : editPasswordHandler
    }

    return (
        // Provider를 통해 생성한 context를 하위 컴포넌트에게 전달한다.
        <loginContext.Provider value={userContext}>
            {props.children}
        </loginContext.Provider>
    );
}
export default LoginProvider;
~~~
위의 코드는 useContext를 사용하기 위한 제공자(Provider)를 생성한 것이다.  
해당 코드의 역할은 다음과 같다.  
- 저장소의 초기값을 설정  
- 저장소의 데이터를 변경하기 위한 함수를 작성 (useReducer 이용)
- Provider를 생성하여 자식 컴포넌트에서 해당 저장소의 데이터를 사용하도록 설정  

return문에서 생성한 저장소 객체인 loginContext에서 Provider를 추가하여  
생성한 context를 하위 컴포넌트에게 전달할 수 있도록 한다. 따라서 App.js를 다음과 같이 수정한다.  

~~~js
// file: 'App.js'
function App() {

  return (
    <LoginProvider>
          <RouterProvider 
            router={router}  
          >
          </RouterProvider>
    </LoginProvider>
  );

}
~~~
생성한 LoginProvider 객체로 App.js의 하위 컴포넌트를 감싸주면  
하위 컴포넌트 모든 곳에서 저장소의 데이터를 사용할 수 있다.  

## useContext의 상태 갱신
그렇다면 useContext를 이용한 저장소를 만들었다면 어떻게 상태를 갱신할까?  
위의 코드에서 보면 useReducer를 사용하여 저장소의 상태를 변경한다.  
즉 전역적으로 데이터를 사용하고 해당 데이터를 변경하는 것은 다른 React Hook인  
useReducer를 이용한다. 결국 데이터를 갱신하는 것은 useContext만의  
다른 방법이 있는 것이 아닌 React Hook을 통해 변경하는 것이다.  
따라서 React의 컴포넌트 렌더링 규칙을 따를 수 밖에 없다.  

하지만 렌더링 규칙을 따른다는 점에서 Project를 진행하면서 문제점을 겪게 되었다.  
Jwt인증을 구현하는 과정에서 백앤드 Spring boot와 통신할 때 상태 갱신의 문제점이 생겼다.  

## React에서의 상태 갱신
~~~js
// file: 'ExampleComponent.js'
import React, { useState } from 'react';

function ExampleComponent() {
  const [count, setCount] = useState(0);

  const handleIncrement = () => {
    setCount(count + 1);
    console.log('After setCount: ', count);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleIncrement}>Increment</button>
    </div>
  );
}
~~~
useState를 사용하여 상태를 갱신할 때, 상태 갱신은 비동기적으로 이루어진다.  
따라서 함수가 끝나고 해당 상태 갱신이 언제 처리될지 정확히 예측이 불가능하다.  
상태 갱신은 리액트가 컴포넌트를 다시 렌더링할 때 비동기적으로 처리되는 것이다.  
useState와 마찬가지로 useReducer도 동일한 상태 갱신의 알고리즘을 가진다.  

## Project에서의 해결법 (localStorage)

