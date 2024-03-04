---

layout: post
title:  "Axios 통신을 위한 Custom Hook 사용"
date:   2024-03-03 01:38:04 +0900

---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}


## Custom Hook이란?  
- 정규 함수와 같지만 안에 상태를 포함할 수 있는 로직 함수
- 기존의 React Hook과 다른 Hook을 동시에 사용할 수 있다.

React에서 useState or useEffect와 같은 Hook을 사용할 경우  
React 함수 컴포넌트에서만 호출이 가능하다.  
즉 일반적인 JavaScript 함수에서 호출이 불가능하다.  
{:.note}

## Custom Hook이 필요한 이유  
위에서 본 것처럼 React Hook을 사용하기 위해서는 조건이 있다.  
React 함수 컴포넌트에서만 호출이 가능하다는 조건이다.  
따라서 일반적인 JavaScript 함수에서는 호출이 불가능하다.

~~~js
const MainComponent = () => {
    const [number,setNumber] = useState(0);

    return (
        <h1> Main Component</h1>
    )
};
~~~
위의 코드처럼 React 함수형 컴포넌트에서는 useState() Hook을 호출하여  
상태를 관리할 수 있다.

~~~js
const MainComponent = () => {
    
    const setNumberStateHandler = () => {
        const [number,setNumber] = useState(0);
    }

    return (
        <h1> Main Component</h1>
    )
};
~~~
그러나 위의 코드에서처럼 일반적인 JS 함수에서 useState()를 호출한다면 오류를 경험하게 된다.  

그렇다면 위의 조건대로 함수형 컴포넌트에서 사용하면 되는것 아닌가?  
Side Project를 진행하면서 함수형 컴포넌트 외에도 Hook을 사용해야 하는 경우를 접하게 되었다.  

## Project에서의 사용 이유
- Axios
- Spring boot
- React  


사이드 프로젝트를 진행하면서 위의 개발 요건으로 개발을 진행하게 되었다.  
Jwt인증을 구현하게 되면서 jwt token을 관리할 방법을 고민하였다.  
나는 useContext() Hook을 사용하였고, 이에 따라 jwt 인증 방식으로  
Spring boot 백앤드로 요청과 함께 Token을 보내야만 했다.  

~~~js
login({ email, password })
        .then(response => {
           return response.data
        }).then(
            responseData => {
                const {id,name,email,password} = responseData.MEMBER;
                const {accessToken, grantType, refreshToken, accessTokenExpiresIn} = responseData.TOKEN;

                localStorage.setItem("accessToken",accessToken); // LocalStorage에 accessToken 저장
                loginCtx.loginUser(id,name,email,password);
                tokenCtx.setUserToken(grantType,accessToken,refreshToken,accessTokenExpiresIn);
                navigate("/NaHC/main");
            }
        )
~~~
위 코드는 LoginPage의 JS 코드 중 일부이다.  
login은 Axios를 이용한 요청 메소드이다.  
그리고 백앤드 서버에서 Token 및 Login 사용자의 정보를 받고 useContext 저장소에 저장한다.  
이 과정까지는 Custom Hook의 필요성을 못느꼈다.  
이제 Token과 함께 Axios 요청을 보내는 코드를 살펴보면 다음과 같다.  

~~~js
export const editName = ({ email, newName, grantType, accessToken }) => {
    return memberApiClient.post('/editName',null,{
        params : {
            email : email,
            newName : newName
        },
        withCredentials: true,
        headers: { Authorization:`${grantType} ${accessToken}`}
    }) 
}
~~~
Axios 이용한 서버로의 요청 코드로 grantType과 AccessToken을 통해   
백앤드 서버에서는 검증을 진행한다. 하지만 해당 editName() 이라는 메소드는  
JS 함수로 해당 함수안에서 useContext()를 통해 저장소를 가져올 수 없다.  
따라서 editName이라는 메소드를 사용하는 JSX File에서 useContext()를 통해 저장소를 가져오고  
해당 메소드를 호출해야 한다.  

그렇다면 이에 대한 문제점은 무엇일까?  
- 반복되는 코드  
- 유지보수의 문제    
- useContext() 저장소의 상태 갱신  

- 반복되는 코드 / 유지보수의 문제  
코드의 재사용성이다. 해당 프로젝트를 진행하면서 여러 백앤드 서버의 Rest API를 불러오게 되었다.  
하지만 많은 Rest API를 호출하면서 모든 호출하는 JSX 파일마다 useContext()의 저장소를 불러오고  
요청에 대한 에러처리 및 Refresh Token을 통한 재인증을 구현하는 것은 비효율적이다.  

- useContext() 저장소의 상태 갱신   
다음 문제는 useContext() 저장소의 상태 갱신 문제이다.  
이에 대한 문제는 상태 갱신에서 React의 렌더링 문제와 연관되며 아래의 포스팅을 참고하면 된다.  
[Custom Hook에서 useContext의 상태 갱신 문제][url]

~~~js
const tokenCtx = useContext(authContext);
const loginCtx = useContext(loginContext);

const tryGrantType = tokenCtx.grantType;
const tryAccessToken = tokenCtx.accessToken;
const tryRefreshToken = tokenCtx.refreshToken;
const tryAccessTokenExpiresIn = tokenCtx.accessTokenExpiresIn;

 try{
    
    // Axios 함수 호출
    const editNameResponse = await editName({
        grantType : tryGrantType,
        accessToken : tryAccessToken,
        email : loginCtx.email,
        newName : newName
    });

    // Axios 함수 호출하여 Data 획득
    const editNameResponseData = await editNameResponse.data;
    }catch(error){

        // AccessToken이 만료된 경우 서버에서 401로 응답한다.
        if (error.response.status === 401){

            try {
                const token = {
                    grantType : tryGrantType,
                    accessToken : tryAccessToken,
                    accessTokenExpiresIn : tryAccessTokenExpiresIn,
                    refreshToken : tryRefreshToken
                };

                // refreshToken을 통해 재인증 및 새로운 jwt Token 발급 함수
                const refreshTokenResponse = await refreshTokenProcess(token);
                const refreshTokenResponseData = await refreshTokenResponse.data;

                const { 
                    accessToken : newAccessToken,  
                    grantType : newGrantType, 
                    accessTokenExpiresIn : newaccessTokenExpiresIn, 
                    refreshToken : newRefreshToken 
                } = refreshTokenResponseData;

                tokenCtx.setUserToken(newGrantType,newAccessToken,newRefreshToken,newaccessTokenExpiresIn); 

                // 기존의 요청하였던 editName() Rest API를 재호출
                const newRefreshFunctionResponse = await editName({
                    grantType : newGrantType,
                    accessToken : newAccessToken,
                    email : loginCtx.email,
                    newName : newName
                });

                const editNameResponseData = await editNameResponse.data;

            }catch(error){

                // 만약 Refresh Token도 만료된 경우 재로그인을 요청
                if (error.response.status === 401 || error.response.status === 403){
                    Swal.fire({
                        icon: 'warning',                        
                        title: '세션 만료',         
                        html: `세션이 만료되었습니다.<br> 다시 로그인 해주세요.`
                    });
                    // 기존의 저장되어있던 저장소의 User,Token 정보 삭제
                    loginCtx.logoutUser();
                    tokenCtx.removeUserToken();
                    navigate('/');
                }
            }
            
        }
    } 
~~~
위의 코드는 Rest API를 호출하고 만약 AccessToken이 만료시 Refresh Token을 통해  
재인증을 하는 코드이다. 만약 모든 API 호출에 해당 코드처럼 에러처리 및 재인증을 포함한다면  
코드가 너무 길어져 가독성이 떨어지고 이는 유지보수도 힘들어진다.  
따라서 이 반복되는 코드를 새로운 함수로 생성하거나 Hook을 만들어 재사용을 할 필요가 있다.  

그러나 주의할 점은 useContext()를 사용하므로 일반적인 JS 함수를 사용하지 못한다.  
따라서 Custom Hook을 만들기로 결정하게 되었다.  


## Project에서의 사용 예시

~~~js
const useAuthFunction = () => {

    const tokenCtx = useContext(authContext);
    const loginCtx = useContext(loginContext);
    const navigate = useNavigate();

    const authFunctionHandler = async (userFunction, parameter) => {
        
        const tryGrantType = tokenCtx.grantType;
        // AccessToken은 localStorage를 사용
        // 위의 Custom Hook에서 useContext의 상태 갱신 문제 포스팅을 참고
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
    
                    // 새로 발급받은 AccessToken을 localStorage에 저장
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
위의 코드는 Side Project에서 사용한 Custom Hook이다.  
use~라는 이름으로 시작하는 Component로 만들어 Custom Hook을 명시하고 JSX 코드를  
return 하는 것이 아닌 JS 함수를 return하여 사용할 수 있도록 한다.  
이를 통해 useContext라는 React Hook을 사용할 뿐만 아니라 반복되는 코드 없이  
Custom Hook에서 return하는 함수를 통해 매개변수로 함수와 변수들을 모은 객체를  
넘겨주어 REST API를 호출할 수 있도록 한다.  

이처럼 Custom Hook을 만들어서 유지보수를 용이하게 만들 수 있다.  
Custom Hook을 사용해야 하는 이유는 다음과 같이 정리가능하다.  
- 함수를 통해 반복되는 로직을 사용해야 하는 경우
- 일반적인 JS 함수에서 React Hook을 사용해야 하는 경우  
- JSX 코드가 아닌 React Hook을 사용하여 특정 Data를 return하고 싶은 경우

## Custom Hook을 만들때의 주의점
그러나 Custom Hook을 만들때의 주의사항이 몇개 존재한다.  
- React Project 디렉토리에서 src -> hooks 디렉토리를 생성하여 파일을 생성해야한다.  
- Custom Hook의 파일명은 반드시 use로 시작해야 한다.  
- 최상위 컴포넌트에서 Custom Hook을 호출헤야한다.  
- 분기문(if, for)에서 Hook을 호출해서는 안된다.  

~~~js
const HookTestComponent = () => {

    const [sampleData,setSampleData] = useState(0);
    // useTest() is Custom Hook sample
    // Correct Example
    const testData = useTest(sampleData);
    setSampleData(testData);
}
~~~
위의 예시는 올바른 예시로 최상위 컴포넌트에서  
useTest라는 Custom Hook을 사용하는 예시이다.

~~~js
const ErrorHookComponent = (props) => {

    const [sampleData,setSampleData] = useState(null);
    // useTest() is Custom Hook sample
    // Error Example
    if (props.item){
        const testData = useTest(props.item);
        setNumber(sampleData);
    }
}
~~~
위의 예시는 잘못된 예시로 분기문(if) 안에서 Hook을 호출하는 예시이다.  
만약 함수형 컴포넌트 렌더링 초기에 해당 Hook을 사용할 필요가 있다면  
아래와 같이 Custom Hook에서 useEffect()를 사용해야 한다.

~~~js
const CorrectHookComponent = (props) => {
    
    // Correct Example
    const [sampleData,setSampleData] = useState(null);
    
    const testDate = useTest(props.item);
    setSampleData(testData);
}

const useTest = (item) => {

    const [fetchData,setFetchData] = useState(null);

    useEffect(() => {
        if (item) {
            ... Your Logic
            setFetchData(...Your Fetch Data);
        }
    },[item])

    return fetchData;
}
~~~
[url]: 2024-03-03-localstorageAndRendering.md
