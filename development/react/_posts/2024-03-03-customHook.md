---

layout: post
title:  "Custom Hook"
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
그러나 위의 코드에서처럼 일반적인 JS 함수에서 useState()를 호출한다면  
오류를 경험하게 된다.

그렇다면 위의 조건대로 함수형 컴포넌트에서 사용하면 되는것 아닌가?  
Side Project를 진행하면서 함수형 컴포넌트 외에도 Hook을 사용해야 하는  
경우를 접하게 되었다.  

## Side Projec결t에서의 사용 이유
- Axios
- Spring boot
- React
사이드 프로젝트를 진행하면서 위의 개발 요건으로 개발을 진행하게 되었다.  
Jwt인증을 구현하게 되면서 jwt token을 관리할 방법을 고민하였다.  
나는 useContext() Hook을 사용하였고, 이에 따라 jwt 인증 방식으로  
Spring boot 백앤드로 요청과 함께 Token을 보내야만 했다.  

## Side Project에서의 사용 예시

