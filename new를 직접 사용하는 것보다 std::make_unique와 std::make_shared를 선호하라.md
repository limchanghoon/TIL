---


---

<h1 id="new를-직접-사용하는-것보다-stdmake_unique와-stdmake_shared를-선호하라">new를 직접 사용하는 것보다 std::make_unique와 std::make_shared를 선호하라</h1>
<p>new로 직접 할당한 객체를 사용하면 자원누수의 위험이 있다.</p>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token function">processWidget</span><span class="token punctuation">(</span>std<span class="token operator">::</span>shared_ptr<span class="token operator">&lt;</span>Widget<span class="token operator">&gt;</span><span class="token punctuation">(</span><span class="token keyword">new</span> Widget<span class="token punctuation">)</span><span class="token punctuation">,</span> <span class="token function">compute</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token comment">// 자원 누수 위험이 있음</span>
</code></pre>
<p><code>"new Widget" 실행 -&gt; std::shared_ptr 생성자 실행</code>과정 사이에 compute()가 실행돼 예외를 던질 수 있다. 그러면 Widget 객체 누수가 발생함.<br>
<code>std::make_shared&lt;&gt;()</code> 을 사용하면 안전하다.</p>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token function">processWidget</span><span class="token punctuation">(</span>std<span class="token operator">::</span>make_shared<span class="token operator">&lt;</span>Widget<span class="token operator">&gt;</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">,</span> <span class="token function">compute</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token comment">// 자원 누수 위험이 없음</span>
</code></pre>
<p>또한 std::make_shared&lt;&gt;() 는 한 번의 할당만 함으로써 성능적 우위가 있다.<br>
new를 사용하면 객체를 위한 메모리 할당과 제어 블럭을 위한 메모리 할당, 총 2번의 할당이 일어난다.</p>
<ul>
<li>new를 사용해야하는 경우</li>
</ul>
<ol>
<li>커스텀 삭제자를 지정해야 하는 경우</li>
<li>중괄호 초기치 사용하려는 경우(이건 구지 중괄호를 안쓰면 될거같다. 또한 auto initList = { 10, 20} 이런 객체를 만들어 make 함수에 넣어주면 된다.)<br>
------------------std::unique_ptr은 여기까지 해당----------------------1&amp;2</li>
<li>make함수로 생성하면 std::shared_ptr이 전부 파괴될지라도 std::weak_ptr이 남아있으면 객체가 차지하던 메모리는 해제되지 않는다. std::weak_ptr이 다 파괴될 때 까지.(new는 std::shared_ptr이 전부 파괴되면 객체가 차지하던 메모리 해제됨)</li>
</ol>

