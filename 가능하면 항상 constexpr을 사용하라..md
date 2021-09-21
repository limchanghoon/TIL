---


---

<h1 id="가능하면-항상-constexpr을-사용하라">가능하면 항상 constexpr을 사용하라</h1>
<p>constexpr가 적용된 객체는 실제로 const이고 컴파일 시점에서 그 값은 알려진다.<br>
컴파일 시점에 알려진 값은 일기 전용 메모리에 배치될 수 있다. 이는 C++에서 <strong>정수 상수 표현식</strong>이 요구되는 문맥에서 사용될 수 있다.</p>
<p><strong>정수 상수 표현식</strong> : 배열 크기나 정수 템플릿 인수(std::array 객체의 길이), 열거자 값, 정합 지정자를 지정하는 등의 여러 문맥</p>
<p>const와 constexpr는 동일한 보장을 제공하지 않는다.</p>
<pre class=" language-c"><code class="prism ++ language-c"><span class="token keyword">int</span> sz<span class="token punctuation">;</span>
<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
constexpr <span class="token keyword">auto</span> arraySize1 <span class="token operator">=</span> sz<span class="token punctuation">;</span>			<span class="token comment">// 오류! sz값이 컴파일 도중에</span>
										<span class="token comment">// 알려지지 않음</span>
<span class="token keyword">const</span> <span class="token keyword">auto</span> arraySize2 <span class="token operator">=</span> sz<span class="token punctuation">;</span>				<span class="token comment">// OK</span>
										<span class="token comment">// sz의 const 복사본</span>
</code></pre>
<p>constexpr함수는 컴파일 시점 상수를 인수로 해서 호출된 경우에는 컴파일 시점 상수를 산출한다. 실행시점이 되어서야 알려지는 값으로 호출하면 실행시점 값을 산출한다. 이는 컴파일 시점 상수를 위한 버전과 다른 모든 값을 위한 버전으로 나누어서 구현할 필요가 없어지는 장점이 있다.</p>
<p>C++11에서 constexpr함수는 실행 가능 문장이 많아야 하나이어야 하고, 보통의 경우 그 문장은 return 문일 수밖에 없다. 이를 삼항 연산자와 재귀를 사용하면 어는 정도 요령있게 사용할 수 있다.</p>
<pre class=" language-c"><code class="prism ++ language-c">	constexpr <span class="token keyword">int</span> <span class="token function">pow</span><span class="token punctuation">(</span><span class="token keyword">int</span> base<span class="token punctuation">,</span> <span class="token keyword">int</span> exp<span class="token punctuation">)</span> noexcept	<span class="token comment">//C++11</span>
	<span class="token punctuation">{</span>
	<span class="token keyword">return</span> <span class="token punctuation">(</span>exp <span class="token operator">==</span> <span class="token number">0</span> <span class="token operator">?</span> <span class="token number">1</span> <span class="token punctuation">:</span> base <span class="token operator">*</span> <span class="token function">pow</span><span class="token punctuation">(</span>base<span class="token punctuation">,</span> exp <span class="token operator">-</span><span class="token number">1</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token punctuation">}</span>
</code></pre>
<p>C++14에서는 이런 제약들이 없다.<br>
constexpr 함수는 반드시 리터럴 형식들을 받고 돌려주어야 한다. (컴파일 도중에 값을 결정할 수 있는 형식). C++11에서 void를 제외한 모든 내장 형식이 리터럴 형식에 해당한다.<br>
컴파일 시점에서 수행하는 계산들이 많을수록  소프트웨어는 빨라지고 컴파일 시간은 길어진다.<br>
constexpr는 객체나 함수의 인터페이스의 일부라는 점을 명심하자.<br>
나중에 constexpr를 제거하면 갑자기 컴파일이 되지 않는 클라이언트가 아주 많을 수 있다.</p>

