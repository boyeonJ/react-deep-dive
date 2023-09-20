## Hook과 랜더링
지난번 포스팅 [Hook의 동작원리 파헤쳐보기(React 코드 까보기) 02 - 외부 주입 역할을 하는(의존성 관리) ReactSharedInternals.js와 shared 패키지](https://velog.io/@boyeon_jeong/Hook%EC%9D%98-%EB%8F%99%EC%9E%91%EC%9B%90%EB%A6%AC-%ED%8C%8C%ED%97%A4%EC%B3%90%EB%B3%B4%EA%B8%B0React-%EC%BD%94%EB%93%9C-%EA%B9%8C%EB%B3%B4%EA%B8%B0-02-%EC%99%B8%EB%B6%80-%EC%A3%BC%EC%9E%85-%EC%97%AD%ED%95%A0%EC%9D%84-%ED%95%98%EB%8A%94%EC%9D%98%EC%A1%B4%EC%84%B1-%EA%B4%80%EB%A6%AC-ReactSharedInternals.js%EC%99%80-shared-%ED%8C%A8%ED%82%A4%EC%A7%80) 에 이어서 hook 구현체 주입을 더 자세히 살펴보도록 합시다!

훅 구현체 주입은 reconciler/renderWithHooks()에서 이루어 집니다. 이 함수는 Render phase에서 실행됩니다. 이때 HooksDispatcherOnMount > mountState > mountWorkInProgressHook 함수가 순서대로 실행됩니다. 

![](https://velog.velcdn.com/images/boyeon_jeong/post/456185a8-bd94-4e80-9c94-7371a0bd1e32/image.png)

그리고 hook 자체는 workInProgressHook이라는 linked list에 저장됩니다. 
![](https://velog.velcdn.com/images/boyeon_jeong/post/3b3663b0-8faa-4072-9a2b-f69e3aceb7a1/image.png)


또한 각각 hook 내부의 status값은 hook.queue에 쌓입니다. 
![](https://velog.velcdn.com/images/boyeon_jeong/post/8c8c11ca-bfe4-44bd-aa05-af1115f1771f/image.png)


## 총정리
마지막으로 지금까지 분석한 내용을 총 정리하면 아래와 같습니다!
### 1. Core: ReactCurrentDispatcher 주입해줘
Core 부터 순서대로 hook으로 가보면 결국에는 ReactCurrentDispatcher.current에 {useState: ~, useEffect:~ } 와 같이 hook이 주입될 것이라고 예상할 수 있습니다.
![](https://velog.velcdn.com/images/boyeon_jeong/post/02dab6d9-652b-4b69-9e4c-14459d6a069e/image.png)

### 2. reconciler: 주입해줄께
reconciler/renderWithHooks()에서 위에서 말했던 hook 객체를 주입해줍니다. 
이제 useState, useEffect 등 호출하여 사용할 수 있는 상태가 됩니다.
> hook 객체 : {useState: ~, useEffect:~ }

![](https://velog.velcdn.com/images/boyeon_jeong/post/9518ca77-92b3-4294-8b8a-3a8f7834876b/image.png)

내부 코드는 아래와 같으며 두가지의 역할을 수행합니다.
1. **호출한 hook을 연결리스트에 저장**
2. 현재 hook의 값(state), dispatcher(set함수)를 return
![](https://velog.velcdn.com/images/boyeon_jeong/post/b0ee533c-1454-417b-823f-55a7d8141f8f/image.png)

연결리스트를 추가하는 부분을 자세히 살펴보면 firstWorkInProgressHook인 연결리스트의 head이며 workInProgressHook가 tail(이자 현재의 hook)입니다.
![](https://velog.velcdn.com/images/boyeon_jeong/post/568e82c5-a816-4c7f-9093-c68095db499f/image.png)

그래서 만약 실제 여러 hook을 호출했을경우 차례대로 연결리스트에 저장이 되어, 맨 처음 호출한 hook이 head인 firstWorkInProgressHook에 저장이 되고, 제일 마지막에 호출한 hook이 workInProgressHook에 저장됩니다.
![](https://velog.velcdn.com/images/boyeon_jeong/post/89554abc-072d-4c69-abf4-835bd10c3ac8/image.png)

> 하나의 hook에 여러번 set 해주는 경우에는 해당 hook의 큐에 저장합니다.

### reconciler: 훅과 컴포넌트와 매핑도 시켜줄께
마지막으로 위에서 reconciler가 hook구현체를 주입시켜주면서 훅과 컴포넌트도 매핑시켜주었습니다. 소스를 보면 workInProgress(fiber)의 memoizedState에 firstWorkInProgressHook를 연결시켜주었습니다. firstWorkInProgressHook는 위에서 설명했듯이 hook 연결리스트의 head이므로 이를 통해 훅과 컴포넌트와 매핑시켜줄수 있습니다.

![](https://velog.velcdn.com/images/boyeon_jeong/post/39683f9b-0e37-4d46-bb04-628bb8262599/image.png)


---
