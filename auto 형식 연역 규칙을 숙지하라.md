auto 형식 연역 규칙을 숙지하라
============================
기본적으로 템플릿 형식 영역과 auto 형식 영역 사이에는 직접적인 대응 관계가 존재한다.

* 포인터, 참조,보편참조,포인터도 참조도 아님 모두 **템플릿과 똑같이 동작**한다.

```c++
auto x = 27;			//x의 형식은 int
const auto& rx = x;		//rx의 형식은 const int&
auto&& uref1 = x;		//uref1의 형식은 int&
auto&& uref2 = rx;		//uref2의 형식은 const int&
auto&& uref3 = 27;		//uref3의 형식은 int&& 
```

배열과 함수 이름이 포인터로 붕괴되는 방식도 똑같이 일어난다.

이제 템플릿과 다른 경우를 살펴보자.

* **중괄호 초기치**에서 차이를 보인다.

```c++
auto x = { 11, 23, 4};		// x의 형식은
							// std::initializer_list<int>
template<typename T>		// x의 선언에 해당하는 매개변수
void f(T param);			// 선언을 가진 템플릿
							//
f({ 11, 23, 4});			// 오류! T에 대한 형식을 연역할 수 없음.
```

T의 형식이 제대로 연역되게 하려면...

```c++
template<typename T>
void f(stdd::initializer_list<T> param);
							//
f({ 11, 23, 4});			// T는 int로 연역되고,
							// param의 형식은 stdd::initializer_list<int>로 연역됨.
```

정리하면, auto는 중괄호 초기치가 std::initializer_list 를 나타낸다고 가정하지만 템플릿 형식 연역은 그렇지 않다.

C++14에서는 알아둬야 할 것이 더 남아있다. 그것은 **auto 반환형식**과 **람다의 매개변수 형식 명세에 auto**를 사용하는 경우이다.

그러한 용법들에는 **auto 형식** 연역이 아니라 **템플릿 형식**연역의 규칙들이 적용된다.

```c++
auto createInitList()
{
	reuturn { 1, 2, 3 };	// 오류! { 1, 2, 3 }의 형식을 연역할 수 없음.
}
```

```c++
std::vector<int> v;
...
auto resetV = 
	[&v](const auto& newValue) { v = newValue };	// C++14
...
resetV({ 1, 2, 3});		// 오류! { 1, 2, 3 }의 형식을 연역할 수 없음
```
