### 오버로딩

일반적으로 JavaScript에서 언어적으로 오버로딩을 지원하지 않는다.

아래와 같이 매개변수가 다른 함수 A를 호출할 경우 자바스크립트에서는 제일 아래 선언한

함수가 같은 이름의 함수를 덮어씌운다.

```jsx
function A(number){
	console.log(1);
}
function A(number, type) {
	console.log(2);
}

A(1);
// 출력: 2
```

하지만, JavaScript 에서도 arguments를 활용해서 오버로딩과 유사하게 동작하도록 개발을 할 수 있다.

아래와 같이 넘겨받는 arguments의 합을 for문을 통해 처리하도록 할 수 있다.

```jsx
function sum() {
    var result = 0;
    for (var i = 0; i < arguments.length; i++) {
        result += arguments[i];
    }
    return result;
}

console.log(sum(10, 20)); // 30
console.log(sum(10, 20, 30)); // 60
console.log(sum(10, 20, 30, 40)); // 100
console.log(sum(10, 20, 30, 40, 50)); // 150
```
