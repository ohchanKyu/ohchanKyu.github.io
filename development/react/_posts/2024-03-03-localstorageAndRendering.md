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
- ![Full-image](/assets/img/useContext/state_update2.png){:.lead width="200" height="100" loading="lazy"}
- ![Full-image](/assets/img/useContext/state_update.png){:.lead width="200" height="100" loading="lazy"}
위의 사진은 ExampleComponent.js 파일을 실제 구현하였을때 나타나는 현상이다.  

useState를 사용하여 상태를 갱신할 때, 상태 갱신은 비동기적으로 이루어진다.  
따라서 함수가 끝나고 해당 상태 갱신이 언제 처리될지 정확히 예측이 불가능하다.  
상태 갱신은 리액트가 컴포넌트를 다시 렌더링할 때 비동기적으로 처리되는 것이다.  
위의 결과처럼 console에 찍히는 값과 실제 렌더링 후에 나타는 값이 차이가 나는 것을 볼 수 있다.  
useState와 마찬가지로 useReducer도 동일한 상태 갱신의 알고리즘을 가진다.  

- Project에서의 코드는 다음과 같다.
~~~js
// file: 'example function'
const exampleFunction =  async (address) => {

        const localeResponseData = await functionHandler(getLocaleByAddress,{
            address
        });
      
        const cafeResponseData =  await functionHandler(getPlaceLocation,{
            x : localeResponseData.x,
            y : localeResponseData.y,
            placeName : cafePlaceName,
        }); 

        const fetchDatas = await fetchplaceItemHandler({
            placeDatas : cafeResponseData, 
            originX : localeResponseData.x,
            originY : localeResponseData.y,
        })
    }
~~~
- functionHandler는 Custom Hook으로 백앤드의 REST API를 호출한다.  
- Custom Hook에서는 useContext의 데이터를 변경한다.  
위의 코드를 보면 비동기적으로 REST API 호출을 하나의 함수에서 여러번 호출한다.  
Custom Hook에서 useContext를 사용하여 데이터를 갱신하는데  
위에서 설명한 것처럼 상태 갱신은 컴포넌트를 재랜더링할때 비동기적으로 처리된다.  
따라서 첫번째 비동기 통신에서 useContext의 값을 변경하여도 두번째 비동기 통신에서 값이  
변경되지 않는다. 이는 JWT 인증 방식에서 문제가 되었다.  

## Project에서의 해결법 - localStorage
~~~js
// file: 'customHook.js'
const useAuthFunction = () => {

    const tokenCtx = useContext(authContext);
    const loginCtx = useContext(loginContext);
    const navigate = useNavigate();

    const authFunctionHandler = async (userFunction, parameter) => {
        
        const tryGrantType = tokenCtx.grantType;
        const tryAccessToken = localStorage.getItem("accessToken");
        const tryRefreshToken = tokenCtx.refreshToken;
        const tryAccessTokenExpiresIn = tokenCtx.accessTokenExpiresIn;

        try{

            const functionResponse = await userFunction({
                grantType : tryGrantType,
                accessToken : tryAccessToken,
                ...parameter
            });
            const functionResponseData = await functionResponse.data;
            return functionResponseData;

        }catch(error){
            
            if (error.response.status === 401){

                try {
                    const token = {
                        grantType : tryGrantType,
                        accessToken : tryAccessToken,
                        accessTokenExpiresIn : tryAccessTokenExpiresIn,
                        refreshToken : tryRefreshToken
                    };
    
                    const refreshTokenResponse = await refreshTokenProcess(token);
                    const refreshTokenResponseData = await refreshTokenResponse.data;
    
                    const { 
                        accessToken : newAccessToken,  
                        grantType : newGrantType, 
                        accessTokenExpiresIn : newaccessTokenExpiresIn, 
                        refreshToken : newRefreshToken 
                    } = refreshTokenResponseData;
    
                    localStorage.setItem("accessToken",newAccessToken);
                    tokenCtx.setUserToken(newGrantType,newAccessToken,newRefreshToken,newaccessTokenExpiresIn); 
    
                    const newRefreshFunctionResponse = await userFunction({
                        grantType : newGrantType,
                        accessToken : newAccessToken,
                        ...parameter
                    });
    
                    const newRefreshFunctionResponseData = await newRefreshFunctionResponse.data;
                    return newRefreshFunctionResponseData;
                }catch(error){
                    if (error.response.status === 401 || error.response.status === 403){
                        Swal.fire({
                            icon: 'warning',                        
                            title: '세션 만료',         
                            html: `세션이 만료되었습니다.<br> 다시 로그인 해주세요.`
                        });
                        loginCtx.logoutUser();
                        tokenCtx.removeUserToken();
                        localStorage.removeItem("accessToken");
                        navigate('/');
                    }
                }
                
            }
        } 
    };
    return authFunctionHandler;
};

export default useAuthFunction;
~~~
위의 코드가 Custom Hook을 제작한 것으로 JWT 인증을 위해 Header Authorization을  
추가하고 REST API를 호출하는 것이다. 기존의 useContext에 저장되어있던 accessToken을  
사용하여 인증을 시도하고, 만약 만료되었다면 RefreshToken을 통해 인증 허가를 받고  
accessToken을 재발급 받도록 한다. 알고리즘을 요약하면 다음과 같다. 

- 기존의 useContext Hook을 사용해 accessToken을 가져온다.  
- 이를 통해 Header 권한 설정 후 REST API를 호출한다.  
- 만약 accessToken이 만료되었다면 useContext에 있는 refreshToken을 통해  
  새로운 토큰을 재발급받는다.
- 새로운 토큰을 useContext에 저장하고 원래 요청하였던 REST API를 재호출하여
  데이터를 얻는다.  

위의 절차를 보면 문제점이 없어보인다. 하지만 React의 렌더링 주기를 생각해볼때 문제점이 발생한다.  
위의 코드에서 context의 값을 변경하고 테스트한 결과이다.

~~~js
// file: 'customHook.js 일부'
 try{
        const functionResponse = await userFunction({
            grantType : tryGrantType,
            accessToken : tryAccessToken,
            ...parameter
        });
        const demoString = "Test String"
        loginCtx.editEmailUser(demoString);
        console.log("Demo String : "+demoString);
        console.log("Context : "+ loginCtx.email);
        
        const functionResponseData = await functionResponse.data;
        return functionResponseData;

    }catch(error){
        ...code
    }
~~~
Demo String을 하나 만들고 Test를 진행한다. console.log()로 찍은 결과는 다음과 같다.  
- ![Full-image](/assets/img/useContext/error.png){:.lead width="200" height="100" loading="lazy"}

위의 상황은 example function처럼 하나의 함수에서 여러개의 REST API를 호출할 때 문제가 된다.  
상태 갱신은 리액트가 컴포넌트를 다시 렌더링할 때 비동기적으로 처리된다.  
따라서 컴포넌트가 다시 렌더링 되기 전에 REST API를 호출하므로 useContext에는  
재발급 받은 accessToken이 아닌 만료된 accessToken이 저장되어 있다.  
따라서 알고리즘의 흐름대로 인증이 이루어지는 것이 아닌 게속 만료된 accessToken으로  
REST API를 호출하게 된다.  
 
그렇다면 이를 해결할 수 있는 방법은 무엇일까?
- React Hook으로는 해결이 불가능하다.  
해당 문제는 React 상태 갱신의 시스템으로 인한 알고리즘 오류로 상태를 통해  
해결하는 것이 아닌 다른 방법이 필요하다.

- React Hook이 아닌 localStorage를 이용한다.  

## localStorage는 무엇일까?
웹 애플리케이션을 개발하다보면 데이터를 어딘가에는 저장해야한다.  
따라서 대부분 서버(DB)나 클라우드 플랫폼을 통해 데이터를 저장한다.  
하지만 중요한 데이터가 아니거나 혹은 Client 측에서만 저장해야 하는 데이터만  
존재할 수 있다. 이럴때 사용하는 것이 localStorage와 sessionStorage이다.  
이 둘을 묶어 Web Storage라고 부른다. 위의 JWT 인증 같은 경우가 Web Storage를  
이용해야 하는 경우이다. 물론 useContext를 통해 Token 관리를 진행하였지만  
렌더링 문제로 Web Storage를 이용하게 되었다. Web Storage는 React의 렌더링 주기와  
상관없이 브라우저에 데이터가 저장되므로 바로 key와 value값으로 갱신된 값을  
저장하고 가져올 수 있다.  

### local Storage vs Session Storage  
그렇다면 local Storage와 Session Storage의 차이는 무엇일까?  


