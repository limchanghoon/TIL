# 오른값 참조에는 std::move를, 보편 참조에는 std::forward를 사용하라

```c++
class Widget{
    Widget(Widget&& rhs) 	// rhs는 오른값 참조
        : name(std::move(rhs.name)),p(std::move(rhs.p))
        {...}
    ...
}

class Widget2{
    template<typname T>
    void setName(T&& newName)		// newName은 보편 참조
    { name = std::forward<T>(newName); }
    ...
}
```

오른값 참조 -> 오른값으로의 `무조건 캐스팅`을 적용해야 한다.(std::move를 통해서)

보편 참조 -> 오른값으로의`조건부 캐스팅`을 적용해야 한다.(std::forward를 통해서)

`반환값 최적화` : 복사 버전에서, 만일 `지역 변수 w`를 함수의 반환값 을 위해 마련한 메모리 안에 생성한다면 w의 복사를 피할 수 있다. (=> 이 경우 std::move() 사용하지마!)

결과를 `값 전달 방식으로 반환`하는 함수의 어떤 `지역 객체의 복사(또는 이동)을 제거`하려면,

1)  그 지역 객체의 형식이 함수의 반환 형식과 같아야 한다.

2)  그 지역 객체가 바로 함수의 반환값이어야 한다.  (값 전달 방식 반환)

   ```c++
   Widget makeWidget()		// 복사 버전
   {
       Widget w;
       ...
       return w;			// 복사 제거로 인해 복사를 피함.
   }						// 복사 제거를 수행하지 않을 때에는 w를 오른값으로 취급
   						// return std::move(w); 로 취급
   ```

```c++
Widget makeWidget(Widget w)		// 함수의 반환값과 형식이 같은,
{								// 값 전달 방식의 매개변수
    ...
    return w;					// w를 오른값으로 취급한다
}								// return std::move(w); 로 취급
```

