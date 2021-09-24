---


---

<h1 id="iterator보다-const_iterator를-선호하라">iterator보다 const_iterator를 선호하라</h1>
<p><strong>const_iterator</strong>는 const를 가리키는 포인터의 STL 버전이다. const_iterator는 수정하면 안되는 값들을 가리킨다. 반복자가 가리키는 것을 수정할 필요가 없을 때에는 항상 const_iterator를 사용하는 것이 바람직하다.<br>
C++98에서는 iterator로 부터 const_iterator를 얻기는 쉽지 않았다. <strong>static_cast(vetor.begin())</strong> 같이 캐스팅을 이용해 정적 캐스팅한다. 다소 작위적인 왜곡이 있다. 그뒤 삽입(삭제) 위치를 지정하려면 이걸 다시 iterator로 캐스팅해야한다. 이 과정에서 오류가 나올 수 있다. 그래서 c++98에선 사용하기 까다로웠다.<br>
하지만 c++11에서는 모든 것이 달라졌다. 컨테이너 멤버 함수 cbeing과 cend는 const_iterator를 돌려준다. insert와 erase 같은 STL 멤버 함수들은 실제로 const_iterator를 사용한다.<br>
템플릿 라이브러리 코드를 작성할 때 비멤버 함수로서 제공해야 하는 컨테이너들과 컨테이너 비슷한 자료구조들이 존재할 수 있다. (물론 필요 없는 함수도 존재함)그럴 때 다음과 같이 하나의 템플릿으로 일반화할 수 있다.</p>
<pre class=" language-c"><code class="prism ++ language-c">template <span class="token operator">&lt;</span>typename C<span class="token punctuation">,</span> typename V<span class="token operator">&gt;</span>
<span class="token keyword">void</span> <span class="token function">findAndInsert</span><span class="token punctuation">(</span>C<span class="token operator">&amp;</span> container<span class="token punctuation">,</span>
				   <span class="token keyword">const</span> V<span class="token operator">&amp;</span> targetVal<span class="token punctuation">,</span>
				   <span class="token keyword">const</span> V<span class="token operator">&amp;</span> inserVal<span class="token punctuation">)</span>
<span class="token punctuation">{</span>
	using std<span class="token punctuation">:</span><span class="token punctuation">:</span>cbegin<span class="token punctuation">;</span>
	using std<span class="token punctuation">:</span><span class="token punctuation">:</span>cend<span class="token punctuation">;</span>
	<span class="token keyword">auto</span> it <span class="token operator">=</span> std<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">find</span><span class="token punctuation">(</span><span class="token function">cbegin</span><span class="token punctuation">(</span>container<span class="token punctuation">)</span><span class="token punctuation">,</span>		<span class="token comment">// 비멤버 cbegin</span>
						<span class="token function">cend</span><span class="token punctuation">(</span>contatiner<span class="token punctuation">)</span><span class="token punctuation">,</span>		<span class="token comment">// 비멤버 cend</span>
						targetVal<span class="token punctuation">)</span><span class="token punctuation">;</span>
	
	<span class="token comment">//container.insert(it, inserVal);</span>
<span class="token punctuation">}</span>
</code></pre>
<p>C++11에서 비멤버 cbegin을 제공하지 않는다면, 직접 구현하면 된다.</p>
<pre class=" language-c"><code class="prism ++ language-c">templat <span class="token operator">&lt;</span>class C<span class="token operator">&gt;</span>
<span class="token keyword">auto</span> <span class="token function">cbegin</span><span class="token punctuation">(</span><span class="token keyword">const</span> C<span class="token operator">&amp;</span> container<span class="token punctuation">)</span> <span class="token operator">-&gt;</span> <span class="token function">decltype</span><span class="token punctuation">(</span>std<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">begin</span><span class="token punctuation">(</span>container<span class="token punctuation">)</span><span class="token punctuation">)</span>
<span class="token punctuation">{</span>
	<span class="token keyword">return</span> std<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">begin</span><span class="token punctuation">(</span>container<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>

