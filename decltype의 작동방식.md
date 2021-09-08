---


---

<h1 id="decltype의-작동-방식을-숙지하라">decltype의 작동 방식을 숙지하라</h1>
<p><strong>decltype</strong>은 항상 변수나 표현식의 형식을 아무 수정 없이 보고한다.</p>
<pre><code>const int i = 0;     //decltype(i)는 const int
Widget w;		     //decltype(w)는 Widget
vector&lt;int&gt; v;
if(v[0] == 0) ...    //decltype(v[0]) 은 int&amp;
</code></pre>
<p>decltype을 이용한 함수 반환 형식 표현도 가능하다.</p>
<pre class=" language-c"><code class="prism ++ language-c">template<span class="token operator">&lt;</span>typename Container<span class="token punctuation">,</span> typename Index<span class="token operator">&gt;</span>
<span class="token keyword">auto</span> <span class="token function">authAndAccess</span><span class="token punctuation">(</span>Container<span class="token operator">&amp;</span> c<span class="token punctuation">,</span> Index i<span class="token punctuation">)</span>
 <span class="token operator">-&gt;</span> <span class="token function">decltype</span><span class="token punctuation">(</span>c<span class="token punctuation">[</span>i<span class="token punctuation">]</span><span class="token punctuation">)</span>
 <span class="token punctuation">{</span>
     <span class="token function">authenticateUser</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
     <span class="token keyword">return</span> c<span class="token punctuation">[</span>i<span class="token punctuation">]</span><span class="token punctuation">;</span>
 <span class="token punctuation">}</span>
</code></pre>
<pre class=" language-c"><code class="prism ++ language-c">template<span class="token operator">&lt;</span>typename Container<span class="token punctuation">,</span> typename Index<span class="token operator">&gt;</span>
<span class="token function">decltype</span><span class="token punctuation">(</span><span class="token keyword">auto</span><span class="token punctuation">)</span> <span class="token function">authAndAccess</span><span class="token punctuation">(</span>Container<span class="token operator">&amp;</span> c<span class="token punctuation">,</span> Index i<span class="token punctuation">)</span>
 <span class="token punctuation">{</span>
     <span class="token function">authenticateUser</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
     <span class="token keyword">return</span> c<span class="token punctuation">[</span>i<span class="token punctuation">]</span><span class="token punctuation">;</span>
 <span class="token punctuation">}</span>
</code></pre>
<p>함수 반환 형식 auto에 decltype 사용하지 않는다면<br>
형식 연역에 의하여  c[i]로부터 연역. (int&amp; 에서 참조성을 무시해 int가 됨 ,즉 오른값이 됨.)</p>

