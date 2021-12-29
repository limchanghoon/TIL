# 단발성 사건 통신에는 void 미래 객체를 고려하라

> 비동기적 스레드A에서 다른 비동기적 스레드B로의 단계적 처리를 위해 통신을 할 때, 접근 방식은 무엇이 있을까?

## 조건 변수 사용

조건 변수 `std::condition_variable`을 사용해 **검출 과제**에서 **반응 과제**로의 통신을 한다.

```c++
std::condition_variable cv;		// 사건을 위한 조건 변수
std::mutex m;					// cv와 함께 사용할 뮤텍스
```

 ```c++
 {								// 검출 과제
     ...
     cv.notify_one();			// 반응 과제에게 알린다.(반응 과제가 여러개면 notify_all)
 }
 ```

```c++
{								// 반응 과제
    std::unique_lock<std::mutex> lk(m);
    cv.wait(lk);				// 통지를 기다린다.
    ...
}
```

 위 코드는 약간의 잘못된 점이 있다. 바로 **뮤텍스**를 사용한다는 점이다.**뮤텍스**는 공유 자료에 대한 접근을 제어하는 데 사용된다.하지만  **검출 과제**와 **반응 과제**가 동시에 같은 변수에 접근하는 일은 일어나지 않는다. 따라서 **뮤텍스**를 사용할 필요가 전혀 없다. 그런데 이보다 더 큰 문제는, 만일 반응 과제가 ***wait*** 을 하기 전에 ***notify*** 를 하게 되면 반응 과제는 무한히 기다리게 된다. 또한 만일 `가짜 기상`이 일어나는 경우에 ***wait*** 이 풀려버리게 된다.

- 가짜 기상(spurious wakeup) : 조건 변수를 기다리는 코드가 조건 변수가 통지되지 않았는데도 깨어나는 현상

## 부울 플래그 (bool flag)

flag라는 부울값을 폴링(주기적 점검)을 통해 참인지 거짓인지 검사한다. 사건 발생시 부울 값이 바뀌며 반응한다.

```c++
std::atomic<bool> flag(false);
// 검출 과제
{
  ...
  flag = true;
}
// 반응 과제
{
  ...
  while (!flag)
  ...
}
```

하지만 **폴링**으로 인한 **자원 낭비**가 단점이다.

## **부울 플래그**와 **조건 변수**를 함께 사용하는 것도 흔하다.

> 이 경우 std::atomic<bool> 대신 그냥 bool로 충분하다.

```c++
std::condition_variable cv;		// 사건을 위한 조건 변수
std::mutex m;					// cv와 함께 사용할 뮤텍스

bool flag(false);
...
{								// 검출 과제
    {
        std::lock_guard<std::mutex> g(m);	// g의 생성자에서 m을 잠근다.
        flag = true;			// 반응 과제에게 알린다.
    }							// g의 소멸자에서 m을 푼다.
    cv.notify_one();			// 반응 과제에게 알린다.(반응 과제가 여러개면 notify_all)
}

{								// 반응 과제
    std::unique_lock<std::mutex> lk(m);
    cv.wait(lk,[] {return flag;});			// 통지를 기다린다. 가짜 기상 방지를 위해 람다 사용
    ...
}
```

잘 작동하지만 mutex와 bool변수 모두 사용해야한다. 또한 **반응 과제**에서 **wait**이 먼저 호출해야하는 것은 변함없다.

## 미래 객체와 std::promise 사용

```c++
std::promise<void> p;		// void로 선언했기 때문에 자료 전송 없이 연결만 함.

// 검출 과제
{
  ...
  p.set_value();
}

// 반응 과제
{
  ...
  p.get_future().wait();
  ...
}
```

공유 상태를 동적으로 할당하기 때문에, 힙 기반 할당 및 해제 비용이 발생한다.

**std::promise**와 **미래 객체**사이의 통신은 `단발성`이다.

```c++
std::promise<void> p;

// 반응 과제
void react();

// 검출 과제
void detect()
{
  ThreadRAII tr(
    std::thread([]
                {
                  p.get_future().wait();	// 미래 객체 설정까지 t 유보
                  react();
                }),
    ThreadRAII::DtorAction::join
  );
  ...										// 사건을 검출함. 만약 여기서 예외 발생시
      										// tr의 소멸자의 즉, join이 안됨. 즉 이 함수는
      										// 무한히 대기한다.
  
  p.set_value();							// t의 유보를 푼다.
  ...
}
```

### 다음 예시는 반응 과제가 여러개일 경우

```c++
std::promise<void> p;

void detect()
{
  auto sf = p.get_furue().share();
  
  std::vector<std:;thread> vt;
  
  for (int i = 0; i < threadsToRun; ++i) {
    vt.emplace_back([sf]{ sf.wait();
                          react(); });
  }
  
  ...
  p.set_value();
  ...
  
  for (auto& t : vt) {
    t.join();
  }
}
```





