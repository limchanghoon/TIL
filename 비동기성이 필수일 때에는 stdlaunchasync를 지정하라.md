# 비동기성이 필수일 때에는 std::launch::async를 지정하라

**std::async** 호출은 **시동 방침**을 지정해 함수를 실행할 수 있다.

- **std::launch::async** : 함수는 비동기적으로, 즉 다른 스레드에서 실행된다.
- **std::launch::deferred** : get이나 wait가 호출될 때에만 실행될 수 있다. (지연된 후 get이나 wait이 호출되면 함수를 동기적으로 실행한다.)

**기본 시동 방침**은 위의 둘을 OR한 것이다. 그래서 **기본 시동 방침**을 사용하면 다음같은 문제가 있다.

- 함수가 **지연 실행**될 수도 있으므로, 함수가 **비동기**적으로 실행될지 예측할 수 없다.
- 함수가 **get**이나 **wait**을 호출하는 스레드와는 다른 스레드에서 실행될지 예측할 수 없다.
- **get**이나 **wait** 호출을 보장할 수 없기 때문에, 함수가 반드시 실행될 것인지 예측할 수 없다.

함수에 `thread_locl 변수`같은 **스레드 지역 저장소(TLS)**를 읽거나 쓰는 코드가 있으면, 그 코드가 어떤 스레드의 지역 변수에 접근할지 예측할 수 없다. 

또한, 만료 기간이 있는 wait기반 루프에서는 **지연된 과제**에 **wait_for**나 **wait_until**을 호출하면 **std::future_status::deferred**라는 값을 반환해 무한 루프에 빠진다.

```c++
auto fut = std::asysnc(f);
while (fut.wait_for(100ms) != std::future_status::ready)
{
    ...
}
```

여기서 해결책으로는 다음과 같다.

```c++
auto fut = std::asysnc(f);
if (fut.wait_for(0s) == std::future_status::deferred)
{
    ...	// fut에 wait이나 get을 적용해서 f를 동기적으로 호출한다.
} else{
    while (fut.wait_for(100ms) != std::future_status::ready)
    {
        ...
    }
    ...
}
```

즉 다음 조건들이 모두 성립할 때에만 기본 시동 방침은 적합하다.

- get이나 wait을 호출하는 스레드와 반드시 **동시적**으로 실행되어야 하는 것은 아닐 때
- 여러 스레드 중 어떤 스레드의 **thread_local** 변수들을 읽고 쓰는지가 중요하지 않을 때
- 미래(future) 객체에 대해 **get**이나 **wait**이 반드시 호출된다는 보장이 있거나 실행되지 않아도 괜찮을 때
- 과제가 지연되는 경우가 **wait_for**이나  **wait_until**에 반영된 경우

 위 조건 중에서 하나라도 부합하지 않은 경우 비동기로 수행되도록 변경해야 한다. 방법은 시동방침을 이용하는 것이다.

```c++
auto fut = std::async(std::launch::async,f);
```

 만약 함수를 만들어서 구현하고 싶다면 다음과 같이 작성할 수 있다.

```c++
// C++11
template<typename F, typename... Ts>
inline
std::future<typename std::result_of<F(Ts...)>::type>
reallyAsync(F&& f, Ts&&... params)
{
  return std::async(std::launch::async,
                    std::forward<F>(f),
                    std::forward<Ts>(params)...);
}

// C++14
template<typename F, typename... Ts>
inline
auto
reallyAsync(F&& f, Ts&&... params)
{
  return std::async(std::launch::async,
                    std::forward<F>(f),
                    std::forward<Ts>(params)...);
}
```

