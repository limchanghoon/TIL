---


---

<h1 id="범위-없는-enum보다-범위-있는-enum을-선호하라">범위 없는 enum보다 범위 있는 enum을 선호하라</h1>
<ul>
<li>범위있는 enum의 첫번째 장점 - 열거자들의 이름</li>
</ul>
<p><strong>범위 없는 enum</strong>은 enum이 속한 범위에 열거자들 이름이 속함.<br>
그래서 그 범위에 같은 이름이 있으면 안됨.</p>
<pre class=" language-c"><code class="prism ++ language-c"><span class="token keyword">enum</span> Color <span class="token punctuation">{</span> black<span class="token punctuation">,</span> white<span class="token punctuation">,</span> red <span class="token punctuation">}</span><span class="token punctuation">;</span>	<span class="token comment">// Color가 속한 범위에 속함</span>

<span class="token keyword">auto</span> white <span class="token operator">=</span> false<span class="token punctuation">;</span>					<span class="token comment">// 오류! 이 범위에 이미</span>
									<span class="token comment">// white가 선언되어 있음</span>
</code></pre>
<p><strong>범위 있는 enum</strong>( <strong>enum 클래스</strong>라고도 함)는 enum의 범위에 열거자들이 속함.</p>
<pre class=" language-c"><code class="prism ++ language-c"><span class="token keyword">enum</span> class Color <span class="token punctuation">{</span> black<span class="token punctuation">,</span> white<span class="token punctuation">,</span> red <span class="token punctuation">}</span><span class="token punctuation">;</span>	<span class="token comment">// Color의 범위에 속함</span>

<span class="token keyword">auto</span> white <span class="token operator">=</span> false<span class="token punctuation">;</span>			<span class="token comment">// OK; 이 범위에 다른 "white"는 없음</span>

Color c <span class="token operator">=</span> white<span class="token punctuation">;</span>			<span class="token comment">// 오류! 이 범위에 "white"라는</span>
							<span class="token comment">// 이름의 열거자가 없음</span>
Color c <span class="token operator">=</span> Color<span class="token punctuation">:</span><span class="token punctuation">:</span>white<span class="token punctuation">;</span>		<span class="token comment">// OK</span>

<span class="token keyword">auto</span> c <span class="token operator">=</span> Color<span class="token punctuation">:</span><span class="token punctuation">:</span>white<span class="token punctuation">;</span>		<span class="token comment">// OK</span>
</code></pre>
<ul>
<li>또 다른 장점은 범위 있는 enum은 그 열거자들의 형식이 훨씬 강력하게 적용된다는 것이다.</li>
</ul>
<p><strong>범위없는 enum</strong>의 열거자는 암묵적으로 <strong>정수 형식으로 변환된다</strong>.  하지만  <strong>범위있는 enum</strong>의 열거자는 암묵적으로 <strong>다른 형식으로 변환되지 않는다</strong>.<br>
만약 어떤 이유로 범위있는 enum의 열거자를 다른 형식으로 변환하고 싶으면 캐스팅을 사용하면 된다.</p>
<pre class=" language-c"><code class="prism ++ language-c"><span class="token comment">//위의 코드와 이어짐</span>
<span class="token keyword">if</span><span class="token punctuation">(</span>static_cast<span class="token operator">&lt;</span><span class="token keyword">double</span><span class="token operator">&gt;</span><span class="token punctuation">(</span>c<span class="token punctuation">)</span> <span class="token operator">&lt;</span> <span class="token number">14.5</span><span class="token punctuation">)</span><span class="token punctuation">{</span>		<span class="token comment">//이상한 코드지만 어쨋든 유효함</span>
<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
</code></pre>
<ul>
<li>세번째 장점으로는 전방선언이 가능하다는 것이다.</li>
</ul>
<pre class=" language-c"><code class="prism ++ language-c"><span class="token keyword">enum</span> Color<span class="token punctuation">;</span>				<span class="token comment">// 오류!</span>

<span class="token keyword">enum</span> class Color<span class="token punctuation">;</span>		<span class="token comment">// OK!</span>
</code></pre>
<p>전방선언은 컴파일과 관련해 큰 이점을 준다.</p>
<pre class=" language-c"><code class="prism ++ language-c"><span class="token keyword">enum</span> class Status<span class="token punctuation">;</span>							<span class="token comment">// 전방 선언</span>
<span class="token keyword">void</span> <span class="token function">continueProcessing</span><span class="token punctuation">(</span>Status s<span class="token punctuation">)</span><span class="token punctuation">;</span>			<span class="token comment">// 전방 선언된 enum 사용</span>
</code></pre>
<p>이때 Status의 정의가 바뀌어도, 이 선언들을 담은 헤더파일은 다시 컴파일할 필요가 없다.<br>
더욱이 Status가 수정되고 그것이 continueProcessing의 행동에 영향을 주지 않는다면 continueProcessing의 구현 역시 다시 컴파일할 필요가 없다.<br>
전방 선언이 가능하려면 enum이 쓰이기 전에 그 크기를 컴파일러가 알아야한다.<br>
<strong>범위 있는 enum</strong>의 바탕 형식은 기본적으로 int이다. 그 기본 형식이 마음에 들지 않는다면 다른  형식을 명시적으로 지정하면 된다.<br>
이 원리를 바탕으로하면 <strong>범위 없는 enum</strong>도 전방 선언이 가능하다.</p>
<pre class=" language-c"><code class="prism ++ language-c"><span class="token keyword">enum</span> class Status<span class="token punctuation">:</span> std<span class="token punctuation">:</span><span class="token punctuation">:</span>unit32_t<span class="token punctuation">;</span>	<span class="token comment">// 범위 있는 enum의 바탕 형식은 std::unit32_t</span>

<span class="token keyword">enum</span> Color<span class="token punctuation">:</span> std<span class="token punctuation">:</span><span class="token punctuation">:</span>unit8_t<span class="token punctuation">;</span>			<span class="token comment">// 범위 없는 enum의 전방 선언 </span>
									<span class="token comment">// 바탕 형식은 std::unit8_t</span>
<span class="token keyword">enum</span> class Status<span class="token punctuation">:</span> std<span class="token punctuation">:</span><span class="token punctuation">:</span>unit32_t<span class="token punctuation">;</span> <span class="token punctuation">{</span> good <span class="token operator">=</span><span class="token number">0</span><span class="token punctuation">,</span>
									<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
									indeterminate <span class="token operator">=</span> <span class="token number">0xFFFFFFFF</span>
									<span class="token punctuation">}</span><span class="token punctuation">;</span>	<span class="token comment">//정의할 때에도 바탕 형식을 지정할 수 있다.</span>
</code></pre>
<p>하지만 <strong>범위 없는 enum</strong>이 유요한 상황도 적어도 하나는 존재한다. 바로 C++11의 std::tuple 안에 있는 필드들을 지칭할 때이다.</p>
<pre class=" language-c"><code class="prism ++ language-c">using UserInfo <span class="token operator">=</span> 					
	 std<span class="token punctuation">:</span><span class="token punctuation">:</span>tuple<span class="token operator">&lt;</span>std<span class="token punctuation">:</span><span class="token punctuation">:</span>string<span class="token punctuation">,</span>		<span class="token comment">// 사용자 이름</span>
    			std<span class="token punctuation">:</span><span class="token punctuation">:</span>string<span class="token punctuation">,</span>		<span class="token comment">// 이메일 주소</span>
				std<span class="token punctuation">:</span><span class="token punctuation">:</span>string<span class="token operator">&gt;</span> <span class="token punctuation">;</span>		<span class="token comment">// 평판치</span>
</code></pre>
<p>나중에 필드를 이용할 때 어느 필드가 사용자 이름인지 평판치인지를 알 수 없다.  이때 <strong>범위 없는 enum</strong>을 이용해 필드 이름을 필드 번호에 연관해 사용할 수 있다.</p>
<pre class=" language-c"><code class="prism ++ language-c"><span class="token keyword">enum</span> UserUnfoFields <span class="token punctuation">{</span> uiName<span class="token punctuation">,</span> uiEmail<span class="token punctuation">,</span> uiReputation <span class="token punctuation">}</span><span class="token punctuation">;</span>
UserInfo uInfo<span class="token punctuation">;</span>
<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
<span class="token keyword">auto</span> val <span class="token operator">=</span> std<span class="token punctuation">:</span><span class="token punctuation">:</span>get<span class="token operator">&lt;</span>uiEmail<span class="token operator">&gt;</span><span class="token punctuation">(</span>uInfo<span class="token punctuation">)</span><span class="token punctuation">;</span>	<span class="token comment">// 이메일 필드의 값을 얻음이 명확함.</span>
</code></pre>
<p>이것이 가능한 이유는 UserUnfoFields 에서 std::size_t으로 암묵적 변환 덕분이다.</p>
<pre class=" language-c"><code class="prism ++ language-c"><span class="token comment">// 범위 있는 enum 사용할 시</span>
<span class="token keyword">auto</span> val <span class="token operator">=</span>
	std<span class="token punctuation">:</span><span class="token punctuation">:</span>get<span class="token operator">&lt;</span>static_cast<span class="token operator">&lt;</span>std<span class="token punctuation">:</span><span class="token punctuation">:</span>size_t<span class="token operator">&gt;</span><span class="token punctuation">(</span>UserInfoFields<span class="token punctuation">:</span><span class="token punctuation">:</span>uiEmail<span class="token punctuation">)</span><span class="token operator">&gt;</span>
		<span class="token punctuation">(</span>uInfo<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<p>열거자 하나를 받아서 그에 해당하는  std::size_t 값을 돌려주는 함수를 작성해서 사용도 가능하다. constexpr 함수를 이용해 컴파일 도중에 결과를 산출한다.</p>
<pre class=" language-c"><code class="prism ++ language-c">template<span class="token operator">&lt;</span>typename E<span class="token operator">&gt;</span>
constexpr typename std<span class="token punctuation">:</span><span class="token punctuation">:</span>underlying_type<span class="token operator">&lt;</span>E<span class="token operator">&gt;</span><span class="token punctuation">:</span><span class="token punctuation">:</span>type
	<span class="token function">toUType</span><span class="token punctuation">(</span>E enumerator<span class="token punctuation">)</span> noexcept
	<span class="token punctuation">{</span>
		<span class="token keyword">return</span> static_cast<span class="token operator">&lt;</span>typename
							std<span class="token punctuation">:</span><span class="token punctuation">:</span>underlying_type<span class="token operator">&lt;</span>E<span class="token operator">&gt;</span><span class="token punctuation">:</span><span class="token punctuation">:</span>type<span class="token operator">&gt;</span><span class="token punctuation">(</span>enumerator<span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token punctuation">}</span>
</code></pre>
<p>toUtype을 이용하면 튜플의 한 필드에 당금과 같이 접근할 수 있다.</p>
<pre class=" language-c"><code class="prism ++ language-c"><span class="token keyword">auto</span> val <span class="token operator">=</span> std<span class="token punctuation">:</span><span class="token punctuation">:</span>get<span class="token operator">&lt;</span><span class="token function">toUtype</span><span class="token punctuation">(</span>UserInfoFields<span class="token punctuation">:</span><span class="token punctuation">:</span>uiEmail<span class="token punctuation">)</span><span class="token operator">&gt;</span><span class="token punctuation">(</span>uInfO<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>

