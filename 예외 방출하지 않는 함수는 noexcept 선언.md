---


---

<h1 id="예외를-방출하지-않을-함수는-noexcept로-선언하라">예외를 방출하지 않을 함수는 noexcept로 선언하라</h1>
<p>throw의 대체로써 <strong>noexcept</strong>으로 선언하면 예외를 방출하지 않는다. 그래서 예외 방출을 위한 스택 되감기에 따른 스택을 확보할 필요가 없어서 성능 향상에 기여함.  noexcept는 예외를 방출하냐 안하냐 라는 이분법적인 정보이다.</p>
<p>std::vector::push_back은 예외 방출 가능성이 있을 때 이동생성자가 호출되면 미정의 행동을 유발할 수 있다. 그래서 예외 방출하지 않는 것이 확실할 때만 이동생성자를 호출한다. (예외 방출 가능성이 있으면 복상생성자가 호출됨 그러면 성능적으로 뒤떨어짐) 이는 noexcept로 선언되어 있는지 여부에 따라 달라짐. (std::vector::push_back의 경우 받는 값이 오른값인지 왼값인지에 따라 달라짐)</p>
<p>noexcept안의 절이 noexcept인지에 의존하는 <strong>조건부 noexcept</strong>가 있다. 에를 들어 표준 라이브러리에 있는 배열에 대한 swap에 대한 선언들이다.</p>
<pre class=" language-c"><code class="prism ++ language-c">template <span class="token operator">&lt;</span>class T<span class="token punctuation">,</span> size_t N<span class="token operator">&gt;</span>
<span class="token keyword">void</span> <span class="token function">swap</span><span class="token punctuation">(</span><span class="token function">T</span> <span class="token punctuation">(</span><span class="token operator">&amp;</span>a<span class="token punctuation">)</span> <span class="token punctuation">[</span>N<span class="token punctuation">]</span><span class="token punctuation">,</span>
   		  <span class="token function">T</span> <span class="token punctuation">(</span><span class="token operator">&amp;</span>b<span class="token punctuation">)</span> <span class="token punctuation">[</span>N<span class="token punctuation">]</span><span class="token punctuation">)</span> <span class="token function">noexcept</span><span class="token punctuation">(</span><span class="token function">noexcept</span><span class="token punctuation">(</span><span class="token function">swap</span><span class="token punctuation">(</span><span class="token operator">*</span>a<span class="token punctuation">,</span> <span class="token operator">*</span>b<span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<p>noexcept안의 noexcept에 있는 swap은 사용자 정의 swap이다. 결과적으로 사용자 정의 swap이 noexcept인지 아닌지에 따라 위 함수의 noexcept여부가 결정된다.</p>
<p>noexcept를 사용하다 나중에 제거하면 클라이언트 코드가 깨질 위험이 생긴다. 따라서 함수의 구현이 예외를 방출하지 않는다는 것을 오랫동안 유지할 확신이 있어야만 noexcept로 선언해야한다.</p>
<p><strong>예외에 중립적</strong>인 함수는 noexcept로 선언할 수 없다. 예외에 중립적인 함수란 직접 예외를 호출하지는 않지만 예외를 던지는 함수를 호출하는 함수이다.<br>
대부분의 함수가 이런 함수여서  noexcept가 지정되어 있지 않다.</p>

