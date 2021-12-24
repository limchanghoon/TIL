# std::bind보다 람다를 선호하라

결론적으로 말하자면, c++14부터는 std::bind는 사용하지말고 람다를 사용하라.

- std::bind를 사용하면 가독성이 매우 떨어진다.

인수를 받고싶으면 자리표를 사용해야고, 호출시점을 맞추다보면 코드가 장황해진다. 중복적재 함수를 지정하면 강제 함수 포인터 형식으로 캐스팅도 해줘야한다. 그러면 인라인화될 가능성도 낮아진다.

또한 작동방식의 명확성이 잘 안보인다. std::bind호출로 인수들을 복사할 때는 값으로 전달되고, std::bind로 얻은 바인드 객체의 호출에서 인수의 전달 방식은 참조로 전달된다. 하지만 람다의 접근 방식은 소스 코드 그대로이다.

즉, C++11에서의 제한된 경우를 제외하면 그냥 람다를 써라.

1. C++11에서 이동 갈무리의 흉내를 낼 때. 

   ```c++
   auto func = std::bind([](const std::vector& data){},std::move(data)) );
   ```

   

2. C++11에서 객체를 템플릿화된 함수 호출 연산자와 묶으려 할 때.

   ```c++
   class PolyWidget{
   public:
       template<typename T>
       void operator()(const T& param) const;
       ...
   };
   
   
   PolyWidget pw;
   auto boundPW = std::bind(pw,_1);
   
   boundPW(1930);	boundPW(nullptr);	boundPW("Rosebud");
   ```

   

