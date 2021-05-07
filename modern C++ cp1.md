템블릿 형식 영역 규칙
============================
```c++
template<typename T>
void f(ParamType param);
```
##경우 1: ParamType이 포인터 또는 참조형식이지만 보편 참조는 아님
* T&

``` c++
template<typename T>
void f(T& param);		// param은 참조 형식
int x= 27;				// x는 int
const int cx = x;		// cx는 const int
const int& rx = x;		// rx는 const int인 x에 대한 참조
```

```c++
f(x);					// T는 int
						// param의 형식은 int&
						//
f(cx);					// T는 const int
						// param의 형식은 const int&
						//
f(rx);					// T는 const int
						// param의 형식은 const int&
```

* const T&

``` c++
template<typename T>
void f(const T& param);		// param은 const 참조 형식
int x= 27;				// x는 int
const int cx = x;		// cx는 const int
const int& rx = x;		// rx는 const int인 x에 대한 참조
```

```c++
f(x);					// T는 int
						// param의 형식은 int&
						//
f(cx);					// T는 int
						// param의 형식은 const int&
						//
f(rx);					// T는 int
						// param의 형식은 const int&
```

* T*


``` c++
template<typename T>
void f(T* param);		// param은 포인터 형식
int x= 27;				// x는 int
const int* px = x;		// px는 const int로서의 x를 가리키는 포인터
```

```c++
f(x);					// T는 int
						// param의 형식은 int*
						//
f(px);					// T는 const int
						// param의 형식은 const int*
```
##경우 2: ParamType이 보편참조임
* T&&

``` c++
template<typename T>
void f(T&& param);		// param은 보편 참조
int x= 27;				// x는 int
const int cx = x;		// cx는 const int
const int& rx = x;		// rx는 const int인 x에 대한 참조
```

```c++
f(x);					// x는 왼값 따라서 T는 int&
						// param의 형식은 int&
						//
f(cx);					// cx는 왼값 따라서 T는 const int&
						// param의 형식은 const int&
						//
f(rx);					// rx는 왼값 따라서 T는 const int&
						// param의 형식은 const int&
						//
f(27);					// 27은 오른값 따라서 T는 int
						// param의 형식은 int&&
```
##경우 3: ParamType이 포인터도 아니고 참조도 아님
* T

``` c++
template<typename T>
void f(T param);		// param은 값으로 전달
int x= 27;				// x는 int
const int cx = x;		// cx는 const int
const int& rx = x;		// rx는 const int인 x에 대한 참조
```

```c++
f(x);					// T와 param의 형식은 둘 다 int
						//
f(cx);					// T와 param의 형식은 둘 다 int
						//
f(rx);					// T와 param의 형식은 둘 다 int
						//
```

``` c++
template<typename T>
void f(T param);		// param은 값으로 전달
						//
const char* const prt = // ptr는 const 객체를 가르키는 const 포인터
 "Fun with pointers";
						//
f(ptr)					// T와 param의 형식은 const char* (const 문자열을 가리키는 수정 가능한 포인터)
```
##배열 인수
* 포인터 붕괴

```c++
const char name[] = "J. P. Briggs"; // name의 형식은
									// const char[13]
									//
const char* ptrToName = name;		// 배열이 포인터로 붕괴된다.
```

사실 name 자체는 const char[13] 형식의 배열이다. 하지만 포인터로 붕괴된다.
```c++
template<typename T>
void f(T param);					// 값 전달 
f(name);
```

```c++
f(name);							// name은 배열이지만 T는 const char*로 연역된다.
```

여기서 f가 인수를 참조로 받도록 바꿔보자

```c++
template<typename T>
void f(T& param);					// 참조 전달 매개변수가 있는 템플릿
f(name);
```

```c++
f(name);							// 배열을 f에 전달
```

여기서 T는 const char [13]으로 연역되고 param의 형식은 const char (&)[13]으로 연역된다.
##배열 인수
* 함수 포인터로의 붕괴

```c++
void someFunc(int,double);			// someFunc는 하나의 함수
									// 형식은 void(int,double)
									//
template<typename T>
void f1(T param);					// f1의 param은 값 전달 방식
									//
template<typename T>
void f2(T& param);					// f2의 param은 참조 전달 방식
									//
f1(someFunc);						// param은 함수 포인터로 연역됨
									// 형식은 void(*) (int,double)
									//
f2(someFunc);						// param은 함수 참조로 연역됨
									// 형식은 void(&) (int,double)
```