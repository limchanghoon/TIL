# 일반적 람다, auto&& 매개변수에는 decltype을 사용하라

## 일반적 람다

```c++
auto f = [](auto&& x){ return normalize( std::forward<decltype(x)>(x) ); };
```

decltype을 쓰는 이유는 std::forward</여기 부분!/> , < > 부분에 들어가는 것은 완벽 전달에선 보통 T 이다. 하지만 여기엔 T가 없다.  여기서 decltype을 사용하면 결과적으로 완벽 전달의 T를 사용하는 결과와 같다.