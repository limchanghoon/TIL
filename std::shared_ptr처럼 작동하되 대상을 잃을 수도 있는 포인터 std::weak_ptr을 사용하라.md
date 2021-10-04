---


---

<h1 id="stdshared_ptr처럼-작동하되-대상을-잃을-수도-있는-포인터가-필요하면-stdweak_ptr을-사용하라">std::shared_ptr처럼 작동하되 대상을 잃을 수도 있는 포인터가 필요하면 std::weak_ptr을 사용하라</h1>
<p><code>std::weak_ptr</code>은 std::shared_ptr처럼 작동하되 소유권 참조 횟수에는 참가하지 않는다. 대상을 잃은 std::waek_ptr은 만료된 상태를 가진다.</p>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token keyword">if</span> <span class="token punctuation">(</span>wpw<span class="token punctuation">.</span><span class="token function">expired</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span> <span class="token comment">// 만료를 직접 판정할 수 있음</span>
</code></pre>
<p>미정의 행동이 방지하려면 만료를 직접 판정하고 피지칭 객체에 대한 접근을 돌려주는 연산을 원자적 연산으로 수행하는 것이다.</p>
<pre class=" language-cpp"><code class="prism  language-cpp">std<span class="token operator">::</span>shared_ptr<span class="token operator">&lt;</span>Widget<span class="token operator">&gt;</span> spw1 <span class="token operator">=</span> wpw<span class="token punctuation">.</span><span class="token function">lock</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>	<span class="token comment">// 만료됐으면 널 배정</span>
<span class="token keyword">auto</span> spw2 <span class="token operator">=</span> wpw<span class="token punctuation">.</span><span class="token function">lock</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

std<span class="token operator">::</span>shared_ptr<span class="token operator">&lt;</span>Widget<span class="token operator">&gt;</span> <span class="token function">spw3</span><span class="token punctuation">(</span>wpw<span class="token punctuation">)</span><span class="token punctuation">;</span>		<span class="token comment">// 만료됐으면 예외 발생</span>
</code></pre>
<hr>
<p>주어진 고유 ID에 해당하는 읽기 전용 객체를 가리키는 똑똑한 포인터를 돌려주는 팩터리 함수가 있다고 하자. 호출 결과를 캐싱하고, 더 이상 쓰이지 않는 객체는 캐시에서 삭제한다.</p>
<ul>
<li>캐시에 있는 포인터들은 자신이 대상을 잃었음을 검출할 수 있어야 한다.</li>
<li>팩토리 함수가 돌려준 객체를 클라이언트가 다 사용하고 나면 그 객체는 파괴되며, 그러면 해당 캐시 항목은 대상을 잃게 될 것이기 때문이다.</li>
<li>따라서 캐시에 저장할 포인터는 자신이 대상을 잃었음을 감지할 수 있는 포인터, 즉 std::weak_ptr이어야 한다.</li>
<li>이는 팩토리 함수의 반환 형식이 반드시 std::shared_ptr이어야 함을 뜻한다.</li>
<li>std::weak_ptr는 객체의 수명을 std::shared_ptr로 관리하는 경우에만 자신이 대상을 잃었음을 감지할 수 있기 때문이다.</li>
</ul>
<pre class=" language-cpp"><code class="prism  language-cpp">std<span class="token operator">::</span>shared_ptr<span class="token operator">&lt;</span><span class="token keyword">const</span> Widget<span class="token operator">&gt;</span> <span class="token function">fastLoadWidget</span><span class="token punctuation">(</span>WidgetID id<span class="token punctuation">)</span>
<span class="token punctuation">{</span>
   <span class="token keyword">static</span> std<span class="token operator">::</span>unoredered_map<span class="token operator">&lt;</span>WidgetId<span class="token punctuation">,</span>
		  	     std<span class="token operator">::</span>weak_ptr<span class="token operator">&lt;</span><span class="token keyword">const</span> Widget<span class="token operator">&gt;&gt;</span> cache<span class="token punctuation">;</span>

   <span class="token keyword">auto</span> objPtr <span class="token operator">=</span> cache<span class="token punctuation">[</span>id<span class="token punctuation">]</span><span class="token punctuation">.</span><span class="token function">lock</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>	<span class="token comment">// objPtr는 캐시에 있는 객체를 가리키는 std::shared_ptr</span>
					<span class="token comment">// 단 객체가 캐시에 없으면 널</span>

   <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span>objPtr<span class="token punctuation">)</span>				<span class="token comment">// 캐시에 없으면 적재하고 캐시에 저장</span>
   <span class="token punctuation">{</span>
      objPtr <span class="token operator">=</span> <span class="token function">loadWidget</span><span class="token punctuation">(</span>id<span class="token punctuation">)</span><span class="token punctuation">;</span>
	  cache<span class="token punctuation">[</span>id<span class="token punctuation">]</span> <span class="token operator">=</span> objPtr<span class="token punctuation">;</span>
   <span class="token punctuation">}</span>
   
   <span class="token keyword">return</span> objPtr<span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<ul>
<li>관찰자 설계 패턴</li>
</ul>
<p>A --------std::shared_ptr-------&gt; B &lt;-------std::shared_ptr------------C<br>
A와 C가 B의 소유권을 공유하고있다. 이때 B에서 다시 A를 가리키는 포인터가 필요하다고 하자. 그 포인터는 어떤 종류의 포인터이어야 할까?</p>
<ul>
<li><strong>생 포인터</strong> : A가 파괴되면 A를 가리키는 생 포인터는 대상을 잃고, 역참조시 미지의 행동이 발생</li>
<li><strong>std::shared_ptr</strong> : A와 B 서로를 가리키는 순환 고리가 생기고 둘 다 파괴하지 못하는 사실상 누수가 발생한다.</li>
<li><strong>std::weak_ptr</strong> : A가 파괴돼도 A를 가리키는 std::weak_ptr포인터는 그 사실을 알 수 있다. 또한 A와 B가 서로를 가리켜도 std::weak_ptr은 참조 횟수에 영향을 주지 않아 A는 파괴가능하다.</li>
</ul>

