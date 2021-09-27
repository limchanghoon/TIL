---


---

<h1 id="소유권-독점-자원의-관리에는-stdunique_ptr를-사용하라">소유권 독점 자원의 관리에는 std::unique_ptr를 사용하라</h1>
<p><code>std::unique_ptr</code>의 특징</p>
<ul>
<li>생 포인터와 같은 크기이고 거의 비슷한 일을 한다.(물론 예외도 있음)</li>
<li>독점적 소유권, 항상 자신이 가리키는 객체를 소유함(널이 아니면)</li>
<li>복사를 허용하지 않고, 이동 전용 형식임</li>
<li>소멸시 자신이 가리키는 자원을 파괴, 자원 파괴는 내부의 생 포인터에 delete를 적용함</li>
<li>std::unique_ptr에서 std::shared_ptr로의 변환이 쉽고 효율적임</li>
</ul>
<p><code>커스텀 삭제자</code>의 특징</p>
<ul>
<li>커스텀 삭제자를 사용할 때에는 그 형식을 std::unique_ptr의 둘째 형식 인수로 지정해야함</li>
<li>생 포인터(이를테면 new를 통해 얻은 포인터)를 std_unique_ptr에 배정하는 문장은 컴파일 되지않음. 이 때문에, new로 생성한 객체의 소유권을 pInv에 부여하기 위해 <code>reset</code>을 호출함</li>
</ul>
<pre class=" language-cpp"><code class="prism  language-cpp">pInv<span class="token punctuation">.</span><span class="token function">reset</span><span class="token punctuation">(</span><span class="token keyword">new</span> <span class="token function">Widget</span><span class="token punctuation">(</span>std<span class="token operator">::</span>forward<span class="token operator">&lt;</span>Ts<span class="token operator">&gt;</span><span class="token punctuation">(</span>params<span class="token punctuation">)</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<ul>
<li>파생 클래스의 객체를 파괴할 때 기반 클래스 포인터로서 delete됨. 이것이 제대로 작동하려면 기반 클래스의 소멸자가 가상 소멸자이어야 함</li>
<li>커스텀 삭제자를 함수 포인터로 사용하면 std::unique_ptr의 크기가 커짐. 이를 해결하려면 상태 없는 람다 표현식으로 구현하는 것이 이득인 경우가 많음</li>
</ul>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token comment">/// 상태 없는 람다 형태의 삭제자</span>
<span class="token keyword">auto</span> delInvmt <span class="token operator">=</span> <span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">(</span>Investment<span class="token operator">*</span> pInvestment<span class="token punctuation">)</span>
				<span class="token punctuation">{</span>
					<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
					<span class="token keyword">delete</span> pInvestment<span class="token punctuation">;</span>
				<span class="token punctuation">}</span>
</code></pre>

