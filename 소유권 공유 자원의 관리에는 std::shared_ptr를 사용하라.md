---


---

<h1 id="소유권-공유-자원의-관리에는-stdshared_ptr을-사용하라">소유권 공유 자원의 관리에는 std::shared_ptr을 사용하라</h1>
<p><code>std::shared_ptr</code> 은 공유된 소유권을 통해 관리한다. 객체를 가르키던 마지막 std::shared_ptr이 더 이상 가리키지 않게되면 객체는 자동으로 파괴된다. 이는 참조 횟수를 통해 이루어진다. 생성자에서 참조 횟수를 증가시키고 소멸자에서 감소시킨다. 기존의 std::shared_ptr을 이용해 새 std::shared_pt에 이동 생성, 이동 배정시 참조 횟수 증가 없음.</p>
<p>참조 횟수 관리는 다음 과 같은 특징이 있다.</p>
<ul>
<li>std::shared_ptr의 크기는 생 포인터의 두 배이다.</li>
<li>참조 횟수를 담을 메모리를 반드시 동적으로 할당해야 한다.</li>
<li>참조 횟수의 증가와 감소는 원자적 연산이어야 한다. (다중 스레드때문에)</li>
</ul>
<p>커스텀 삭제자의 형식이 std::unique_ptr과 다름</p>
<pre class=" language-cpp"><code class="prism  language-cpp">std<span class="token operator">::</span>unique<span class="token operator">&lt;</span>Widget<span class="token punctuation">,</span> <span class="token keyword">decltype</span><span class="token punctuation">(</span>loggingDel<span class="token punctuation">)</span><span class="token operator">&gt;</span> <span class="token function">upw</span><span class="token punctuation">(</span><span class="token keyword">new</span> Widget<span class="token punctuation">,</span> loggingDel<span class="token punctuation">)</span><span class="token punctuation">;</span>

std<span class="token operator">::</span>shared_ptr<span class="token operator">&lt;</span>Widget<span class="token operator">&gt;</span> <span class="token function">spw</span><span class="token punctuation">(</span><span class="token keyword">new</span> Widget<span class="token punctuation">,</span> loggingDel<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<p>즉 다른 형식의 삭제자를 가지는 두 std::shared_ptr끼리는 같은 컨테이너에 담을 수 있고, 하나를 다른 하나에 배정할 수도 있다. 또한 커스텀 삭제자를 지정해도 객체의 크기가 변하지 않는다.</p>
<ul>
<li><code>std::make_shared</code>는 항상 제어 블록을 생성한다. 이 함수는 공유 포인터가 가리킬 객체를 새로 생성하므로, std::make_shared가 호출되는 시점에서 그 객체에 대한 제어 블록이 존재할 가능성은 전혀 없다.</li>
<li>고유 소유권 포인터(std::unique_ptr나 std::auto_ptr)로부터 std::shared_ptr 객체를 생성하면 제어 블록이 생성된다.</li>
<li>생 포인터로 std::shared_ptr 생성자를 호출하면 제어 블록이 생성된다.</li>
</ul>
<p>⇒ 하나의 생 포인터로 여러 개의 std::shared_ptr을 생성하지 말자! 앵간하면 사용하지말고 꼭 써야할 때는 생 포인터 변수를 거치지 말고 new의 결과를 직접 전달하자!</p>
<pre class=" language-cpp"><code class="prism  language-cpp">std<span class="token operator">::</span>shared_ptr<span class="token operator">&lt;</span>Widget<span class="token operator">&gt;</span> <span class="token function">spw1</span><span class="token punctuation">(</span><span class="token keyword">new</span> Widget<span class="token punctuation">,</span> loggingDel<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<p>⇒ 커스텀 삭제자를 지정하지 않을 경우 std::make_shared를 사용하자!</p>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token keyword">void</span> Widget<span class="token operator">::</span><span class="token function">process</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">{</span>
	<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
	processedWidgets<span class="token punctuation">.</span><span class="token function">emplace_back</span><span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">)</span><span class="token punctuation">;</span> 
	<span class="token comment">// emplace_back =&gt; 인수를 이용해 직접 객체를생성해 넣어줌 </span>
<span class="token punctuation">}</span>
</code></pre>
<p>위 코드에서 this(생 포인터)를 std::shared_ptr들의 컨테이너에 넘겨주면 새 제어 블럭이 생긴다.<br>
이를 방지하기 위해 사용하는 것이 <code>std::enable_shared_from_this</code>이다. 이것을 상속하면</p>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token keyword">class</span> <span class="token class-name">Widget</span><span class="token operator">:</span> <span class="token keyword">public</span> std<span class="token operator">::</span>enable_shared_from_this<span class="token operator">&lt;</span>Widget<span class="token operator">&gt;</span> <span class="token punctuation">{</span>
<span class="token keyword">public</span><span class="token operator">:</span>
	<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
	<span class="token keyword">void</span> <span class="token function">process</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
<span class="token punctuation">}</span>
</code></pre>
<p>이 템플릿의 형식 인수로는 파생할 클래스의 이름을 지정해야함. 이러고 나면</p>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token keyword">void</span> Widget<span class="token operator">::</span><span class="token function">process</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">{</span>
	<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
	processedWidgets<span class="token punctuation">.</span><span class="token function">emplace_back</span><span class="token punctuation">(</span><span class="token function">shared_from_this</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span> 
	<span class="token comment">// emplace_back =&gt; 인수를 이용해 직접 객체를생성해 넣어줌 </span>
<span class="token punctuation">}</span>
</code></pre>
<p>이는 현재 객체에 이미 제어 블록이 연관되어 있다고 가정한다. 그런 std::shared_ptr이 존재하지 않으면 함수의 행동은 정의되지 않는다.<br>
std::shared_ptr이 유효한 객체를 가리키기도 전에 클라이언트가 shared_from_this를 호출하는 것을 방지하기 위해, std::enable_shared_from_this을 상속받은 클래스는 자신의 생성자들을 private로 선언한다. 그리고 클라이언트가 객체르 생성할 수 있도록, std::shared_ptr을 돌려주는 팩터리 함수를 제공한다.</p>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token keyword">class</span> <span class="token class-name">Widget</span> <span class="token operator">:</span>  <span class="token keyword">public</span>  std<span class="token operator">::</span>enable_shared_from_this<span class="token operator">&lt;</span>Widget<span class="token operator">&gt;</span> <span class="token punctuation">{</span> 
<span class="token keyword">public</span><span class="token operator">:</span> 
	<span class="token comment">// 팩터리 함수; 인수들을 전용 생성자에 완벽하게 전달한다.  </span>
	<span class="token keyword">template</span> <span class="token operator">&lt;</span><span class="token keyword">typename</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span> Ts<span class="token operator">&gt;</span> 
	<span class="token keyword">static</span>  std<span class="token operator">::</span>shared_ptr<span class="token operator">&lt;</span>Widget<span class="token operator">&gt;</span> <span class="token function">create</span><span class="token punctuation">(</span>Ts<span class="token operator">&amp;&amp;</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span> params<span class="token punctuation">)</span><span class="token punctuation">;</span> 
	<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
	<span class="token keyword">private</span><span class="token operator">:</span>
	<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>					<span class="token comment">// 생성자들</span>
<span class="token punctuation">}</span>
</code></pre>
<p>크지 않은 비용을 치르는 대신 std::shared_ptr을 사용하면 동적 할당 자원의 수명이 자동으로 관리된다는 이득이 생긴다. (소유권 공유 객체)<br>
독점 소유권으로도 충분하다면, 심지어 반드시 충분하지는 않아도 충분할 가능성이 있다면, std::unique_ptr을 사용하자. std::unique_ptr은 생 포인터와 성능은 거의 같고, std::unique_ptr에서 std::shared_ptr로의 업그레이드는 쉽다. 하지만 그 역은 안된다.</p>

