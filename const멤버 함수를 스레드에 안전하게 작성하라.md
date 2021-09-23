---


---

<h1 id="const-멤버-함수를-스레드에-안전하게-작성하라">const 멤버 함수를 스레드에 안전하게 작성하라</h1>
<p><code>const</code> 함수 내부에서는 멤버 변수들의 값을 바꾸는 것이 불가능하다. 하지만, 만약에 멤버 변수를 <code>mutable</code> 로 선언하였다면 const 함수에서도 이들 값을 바꿀 수 있다.</p>
<pre class=" language-c"><code class="prism ++ language-c">class Widget <span class="token punctuation">{</span>
public<span class="token punctuation">:</span>
	<span class="token keyword">double</span> <span class="token function">roots</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token keyword">const</span> <span class="token punctuation">{</span>
	<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
	<span class="token punctuation">}</span>
private<span class="token punctuation">:</span>
	mutable <span class="token keyword">int</span> rootVals <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
Widget p<span class="token punctuation">;</span>
<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
<span class="token comment">/*--- 스레드1 ---*/</span>				<span class="token comment">/*--- 스레드2 ---*/</span>
<span class="token comment">// roots() 는 const 멤버함수로 const 멤버 변수를 변경하고 받아옴</span>
<span class="token keyword">auto</span> root1 <span class="token operator">=</span> p<span class="token punctuation">.</span><span class="token function">roots</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>			<span class="token keyword">auto</span> root2 <span class="token operator">=</span> p<span class="token punctuation">.</span><span class="token function">roots</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<p>같은 메모리를 동기화 없이 일고 쓰려해 스레드1과 스레드2는 자료 경쟁에 빠진다.<br>
이런 경우 통상적인 동기화 수단인 뮤텍스를 사용한다.</p>
<pre class=" language-c"><code class="prism ++ language-c">class Widget <span class="token punctuation">{</span>
public<span class="token punctuation">:</span>
	<span class="token keyword">double</span> <span class="token function">roots</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token keyword">const</span> <span class="token punctuation">{</span>
	std<span class="token punctuation">:</span><span class="token punctuation">:</span>lock_guard<span class="token operator">&lt;</span>std<span class="token punctuation">:</span><span class="token punctuation">:</span>mutex<span class="token operator">&gt;</span> <span class="token function">g</span><span class="token punctuation">(</span>m<span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
	<span class="token punctuation">}</span>
private<span class="token punctuation">:</span>
	mutable std<span class="token punctuation">:</span><span class="token punctuation">:</span>mutex m<span class="token punctuation">;</span>
	mutable <span class="token keyword">int</span> rootVals <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
Widget p<span class="token punctuation">;</span>
</code></pre>
<p><code>std::mutex</code> 형식의 객체 m은 mutable 선언되었다. 그 이유는 m을 잠그고 푸는 멤버 함수들은 비const이지만 roots안에서는 m이 const객체로 간주되므로 이렇게 해야 한다.<br>
<code>std::mutex</code>는 이동과 복사가 불가능하다. 그래서 m을 추가한 Widget도 이동과 복사 능력도 사라진다.(뒤에 나오는 <code>std::atomic</code> 도 마찬가지로 이동과 복사가 불가능)<br>
멤버 함수의 호출 횟수를 세고 싶다면 뮤텍스를 도입하는 것 외에 <code>std::atomic</code>을 사용해 비용을 줄일 수 있다.</p>
<pre class=" language-c"><code class="prism ++ language-c">class Point<span class="token punctuation">{</span>
public<span class="token punctuation">:</span>
	<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
	<span class="token keyword">double</span> <span class="token function">distanceFromOrigin</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token keyword">const</span> noexcept
	<span class="token punctuation">{</span>
		<span class="token operator">++</span>callCount			<span class="token comment">//원자적 증가</span>
		<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
	<span class="token punctuation">}</span>
private<span class="token punctuation">:</span>
	mutable std<span class="token punctuation">:</span><span class="token punctuation">:</span>atomic<span class="token operator">&lt;</span><span class="token keyword">unsigned</span><span class="token operator">&gt;</span> <span class="token function">callCount</span><span class="token punctuation">(</span><span class="token number">0</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<p>밑에는 <code>std::atomic</code>을 두개 이상 같이 사용한 상황이다.</p>
<pre class=" language-c"><code class="prism ++ language-c">class Widget<span class="token punctuation">{</span>
public<span class="token punctuation">:</span>
	<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
	<span class="token keyword">int</span> <span class="token function">magicValue</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token keyword">const</span>
	<span class="token punctuation">{</span>
		<span class="token keyword">if</span><span class="token punctuation">(</span>cacheValid<span class="token punctuation">)</span> <span class="token keyword">return</span> cacheValue<span class="token punctuation">;</span>
		<span class="token keyword">else</span><span class="token punctuation">{</span>
		<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
			cacheValue <span class="token operator">=</span> val1 <span class="token operator">+</span> val2<span class="token punctuation">;</span>
			cacheValid <span class="token operator">=</span> true<span class="token punctuation">;</span>
			<span class="token keyword">return</span> cachedValue<span class="token punctuation">;</span>
		<span class="token punctuation">}</span>
	<span class="token punctuation">}</span>
private<span class="token punctuation">:</span>
	mutable std<span class="token punctuation">:</span><span class="token punctuation">:</span>atomic<span class="token operator">&lt;</span>bool<span class="token operator">&gt;</span> <span class="token function">cacheValid</span><span class="token punctuation">(</span>false<span class="token punctuation">)</span><span class="token punctuation">;</span>
	mutable std<span class="token punctuation">:</span><span class="token punctuation">:</span>atomic<span class="token operator">&lt;</span><span class="token keyword">int</span><span class="token operator">&gt;</span> cachedValue<span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<ol>
<li>1번 스레드에서 cacheValid를 false로 관측하고 cacheValue에 값을 배정함</li>
<li>그 시점에 2번 스레드가 magicValue호출 역시 cacheValid를 false로 관측 비싼 비용을 들여 1번 스레드와 같이 cacheValue에 값을 배정함<br>
이렇게 쓸 데 없는 연산을 하게 된다.<br>
동기화가 필요한 변수 하나 또는 메모리 장소 하나에 대해서는 <code>std::atomic</code>을 사용하는 것이 적합하지만, 둘 이상의 변수나 메모리 장소를 하나의 단위로서 조작해야 할 때에는 <code>뮤텍스</code>을 사용하는 것이 바람직하다.</li>
</ol>

