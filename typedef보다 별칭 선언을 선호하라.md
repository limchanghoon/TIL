---


---

<h1 id="typedef보다-별칭-선언을-선호하라">typedef보다 별칭 선언을 선호하라</h1>
<p>형식의 타이핑(타자수)이 많으면 별칭 선언을 사용하자.<br>
물론 typedef라는 것이 존재한다. 별칭 선언은 현대적인 typedef라고 생각하면 된다.</p>
<pre class=" language-c"><code class="prism ++ language-c">using UPtrMapSS <span class="token operator">=</span> 
	std<span class="token punctuation">:</span>unique_ptr<span class="token operator">&lt;</span>std<span class="token punctuation">:</span><span class="token punctuation">:</span>unordered_map<span class="token operator">&lt;</span>std<span class="token punctuation">:</span><span class="token punctuation">:</span>string<span class="token punctuation">,</span> std<span class="token punctuation">:</span><span class="token punctuation">:</span>string<span class="token operator">&gt;&gt;</span><span class="token punctuation">;</span>

<span class="token comment">//함수 포인터</span>

<span class="token keyword">typedef</span> <span class="token keyword">void</span> <span class="token punctuation">(</span><span class="token operator">*</span>FP<span class="token punctuation">)</span> <span class="token punctuation">(</span><span class="token keyword">int</span><span class="token punctuation">,</span> <span class="token keyword">const</span> std<span class="token punctuation">:</span><span class="token punctuation">:</span>string<span class="token operator">&amp;</span><span class="token punctuation">)</span><span class="token punctuation">;</span>	<span class="token comment">// typedef</span>

using FP <span class="token operator">=</span> <span class="token keyword">void</span> <span class="token punctuation">(</span><span class="token operator">*</span><span class="token punctuation">)</span><span class="token punctuation">(</span><span class="token keyword">int</span><span class="token punctuation">,</span> <span class="token keyword">const</span> std<span class="token punctuation">:</span><span class="token punctuation">:</span>string<span class="token operator">&amp;</span><span class="token punctuation">)</span><span class="token punctuation">;</span>	<span class="token comment">// 별칭 선언</span>
</code></pre>
<p>typedef 보다 나은 별칭 선언의 장점은 템플릿이다.  자타수가 줄어든다!<br>
<strong>별칭 템플릿</strong></p>
<pre class=" language-c"><code class="prism ++ language-c">template<span class="token operator">&lt;</span>typename T<span class="token operator">&gt;</span>		<span class="token comment">// 별칭 선언 사용</span>
using MyAllocList <span class="token operator">=</span> std<span class="token punctuation">:</span><span class="token punctuation">:</span>list<span class="token operator">&lt;</span>T<span class="token punctuation">,</span> MyAlloc<span class="token operator">&lt;</span>T<span class="token operator">&gt;&gt;</span><span class="token punctuation">;</span>

MyAllocList<span class="token operator">&lt;</span>Widget<span class="token operator">&gt;</span> lw<span class="token punctuation">;</span>

template<span class="token operator">&lt;</span>typename T<span class="token operator">&gt;</span>		<span class="token comment">// typedef 사용</span>
<span class="token keyword">struct</span> MyAllocList <span class="token punctuation">{</span>
	<span class="token keyword">typedef</span> std<span class="token punctuation">:</span><span class="token punctuation">:</span>list<span class="token operator">&lt;</span>T<span class="token punctuation">,</span> MyAlloc<span class="token operator">&lt;</span>T<span class="token operator">&gt;&gt;</span> type<span class="token punctuation">;</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>

MyAllocList<span class="token operator">&lt;</span>Widget<span class="token operator">&gt;</span><span class="token punctuation">:</span><span class="token punctuation">:</span>type lw<span class="token punctuation">;</span>
</code></pre>

