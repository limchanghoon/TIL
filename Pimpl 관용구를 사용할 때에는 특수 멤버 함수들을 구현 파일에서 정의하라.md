---


---

<h1 id="pimpl-관용구를-사용할-때에는-특수-멤버-함수들을-구현-파일에서-정의하라">Pimpl 관용구를 사용할 때에는 특수 멤버 함수들을 구현 파일에서 정의하라</h1>
<p><code>Pimpl 관용구</code>는 너무 긴 빌드 시간을 줄이는 방법이다. 기본적인 방식은 클래스의 자료 멤버들을 구현 클래스를 가리키는 포인터로 대체하고, 자료 멤버들을 구현 클래스로 옮기고, 포인터를 통해서 그 자료 멤버들에 간접적으로 접근하는 기법이다.</p>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token keyword">class</span> <span class="token class-name">Widget</span> <span class="token punctuation">{</span>			<span class="token comment">// 헤더 "Widget.h" 안에서</span>
<span class="token keyword">public</span><span class="token operator">:</span>
	<span class="token function">Widget</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token operator">~</span><span class="token function">Widget</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>				<span class="token comment">// 선언만 해둔다</span>
	<span class="token comment">//</span>
	<span class="token function">Widget</span><span class="token punctuation">(</span>Widget<span class="token operator">&amp;&amp;</span> rhs<span class="token punctuation">)</span><span class="token punctuation">;</span>				<span class="token comment">// 이동 연산</span>
	Widget<span class="token operator">&amp;</span> <span class="token keyword">operator</span><span class="token operator">=</span><span class="token punctuation">(</span>Widget<span class="token operator">&amp;&amp;</span> rhs<span class="token punctuation">)</span><span class="token punctuation">;</span>	<span class="token comment">// 선언만 해둔다</span>
	<span class="token comment">//</span>
	<span class="token function">Widget</span><span class="token punctuation">(</span><span class="token keyword">const</span> Widget<span class="token operator">&amp;</span> rhs<span class="token punctuation">)</span><span class="token punctuation">;</span>				<span class="token comment">// 복사 연산</span>
	Widget<span class="token operator">&amp;</span> <span class="token keyword">operator</span><span class="token operator">=</span><span class="token punctuation">(</span><span class="token keyword">const</span> Widget<span class="token operator">&amp;</span> rhs<span class="token punctuation">)</span><span class="token punctuation">;</span>	<span class="token comment">// 선언만 해둔다</span>
	<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
<span class="token keyword">private</span><span class="token operator">:</span>
	<span class="token keyword">struct</span> Impl<span class="token punctuation">;</span>
	std<span class="token operator">::</span>unique_ptr<span class="token operator">&lt;</span>Impl<span class="token operator">&gt;</span> pImpl<span class="token punctuation">;</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>
</code></pre>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">"widget.h"</span>		</span><span class="token comment">// "widget.cpp" 파일 안에서</span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">"gadget.h"</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;string&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;vector&gt;</span></span>

<span class="token keyword">struct</span> Widget<span class="token operator">::</span>Impl<span class="token punctuation">{</span>
	std<span class="token operator">::</span>string name<span class="token punctuation">;</span>
	std<span class="token operator">::</span>vector<span class="token operator">&lt;</span><span class="token keyword">double</span><span class="token operator">&gt;</span> data<span class="token punctuation">;</span>
	Gadget g1<span class="token punctuation">,</span>g2<span class="token punctuation">,</span>g3<span class="token punctuation">;</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>

Widget<span class="token operator">::</span><span class="token function">Widget</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">:</span> <span class="token function">pImpl</span><span class="token punctuation">(</span>std<span class="token operator">::</span>make_unique<span class="token operator">&lt;</span>Impl<span class="token operator">&gt;</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token comment">// 구성원 초기화</span>
<span class="token punctuation">{</span><span class="token punctuation">}</span>

Widget<span class="token operator">::</span><span class="token operator">~</span><span class="token function">Widget</span><span class="token punctuation">(</span><span class="token punctuation">)</span>	<span class="token comment">// ~Widget의 정의</span>
<span class="token punctuation">{</span><span class="token punctuation">}</span>
<span class="token comment">//Widget::~Widget() = default; // 기본 기능으로 충분하다 강조함</span>

Widget<span class="token operator">::</span><span class="token function">Widget</span><span class="token punctuation">(</span>Widget<span class="token operator">&amp;&amp;</span> rhs<span class="token punctuation">)</span> <span class="token operator">=</span> <span class="token keyword">default</span><span class="token punctuation">;</span>
Widget<span class="token operator">&amp;</span> Widget<span class="token operator">::</span><span class="token keyword">operator</span><span class="token operator">=</span><span class="token punctuation">(</span>Widget<span class="token operator">&amp;&amp;</span> rhs<span class="token punctuation">)</span> <span class="token operator">=</span> <span class="token keyword">default</span><span class="token punctuation">;</span>

Widget<span class="token operator">::</span><span class="token function">Widget</span><span class="token punctuation">(</span><span class="token keyword">const</span> Widget<span class="token operator">&amp;</span> rhs<span class="token punctuation">)</span> <span class="token operator">:</span> <span class="token function">pImpl</span><span class="token punctuation">(</span><span class="token keyword">nullptr</span><span class="token punctuation">)</span>	<span class="token comment">// 복사 생성자</span>
<span class="token punctuation">{</span> <span class="token keyword">if</span> <span class="token punctuation">(</span>rhs<span class="token punctuation">.</span>pImpl<span class="token punctuation">)</span> pImpl <span class="token operator">=</span> std<span class="token operator">::</span>make_unique<span class="token operator">&lt;</span>Impl<span class="token operator">&gt;</span><span class="token punctuation">(</span><span class="token operator">*</span>rhs<span class="token punctuation">.</span>pImpl<span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token punctuation">}</span>
Widget<span class="token operator">&amp;</span> Widget<span class="token operator">::</span><span class="token keyword">operator</span><span class="token operator">=</span><span class="token punctuation">(</span><span class="token keyword">const</span> Widget<span class="token operator">&amp;</span> rhs<span class="token punctuation">)</span>	<span class="token comment">// 복사 배정 연산자</span>
<span class="token punctuation">{</span>
	<span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span>rhs<span class="token punctuation">.</span>pImpl<span class="token punctuation">)</span> pImpl<span class="token punctuation">.</span><span class="token function">reset</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token keyword">else</span> <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span>pImpl<span class="token punctuation">)</span> pImpl <span class="token operator">=</span> std<span class="token operator">::</span>make_unique<span class="token operator">&lt;</span>Impl<span class="token operator">&gt;</span><span class="token punctuation">(</span><span class="token operator">*</span>rhs<span class="token punctuation">.</span>pImpl<span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token keyword">else</span> <span class="token operator">*</span>pImpl <span class="token operator">=</span> <span class="token operator">*</span>rhs<span class="token punctuation">.</span>pImpl<span class="token punctuation">;</span>
	<span class="token keyword">return</span> <span class="token operator">*</span><span class="token keyword">this</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<p>스마트 포인터를 사용했기 때문에 직접 소멸자에서 포인터를 삭제해줄 필요가 없다. 하지만 소멸자가 없으면 기본 소멸자가 호출되고 그러면 <code>불완전 형식</code>(선언만 하고 정의를 하지 않은)인 Widget::Impl에 의해  오류가 발생한다. 그래서 소스코드에서 Widget::Impl의 정의 이후에 소멸자의 본문(정의)를 보게하면 된다.<br>
또한 소멸자를 정의했기 때문에 이동 연산들은 자동으로 선언되지 않는다.그래서 역시 헤더에서 선언만하고 소스파일에서 정의한다.<br>
Gadget 또한 복사가능한 형식이라면 3법칙에 의해 (소멸자 또는 이동 연산이 있으니)  복사연산도 직접 정의해줘야 한다.<br>
여기까지는 <code>std::unique_ptr</code>을 사용할 때 이야기였다. <code>std::shared_ptr</code>을 사용할 때는 과연 어떨까?</p>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token keyword">class</span> <span class="token class-name">Widget</span> <span class="token punctuation">{</span>			<span class="token comment">// 헤더 "Widget.h" 안에서</span>
<span class="token keyword">public</span><span class="token operator">:</span>
	<span class="token function">Widget</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>		<span class="token comment">// 쇼멸자나 이동 연산들의 선언이 전혀 없음</span>
<span class="token keyword">private</span><span class="token operator">:</span>
	<span class="token keyword">struct</span> Impl<span class="token punctuation">;</span>
	std<span class="token operator">::</span>shared_ptr<span class="token operator">&lt;</span>Impl<span class="token operator">&gt;</span> pImpl<span class="token punctuation">;</span>
</code></pre>
<p><code>std::unique_ptr</code>에서 삭제자의 형식은 해당 스마트 포인터 형식의 일부이고,  <code>std::shared_ptr</code>은 아니다. 그래서 후자는 컴파일러가 작성한 특수 멤버 함수들이 쓰이는 시점에서 피지칭 형식들이 완전한 형식이어야 한다는 요구조건이 사라진다.</p>

