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
<pre><code>template&lt;typename Container, typename Index&gt;
auto authAndAccess(Container&amp; c, Index i)
 -&gt; decltype(c[i])
 {
     authenticateUser();
     return c[i];
 }
</code></pre>
<pre><code>template&lt;typename Container, typename Index&gt;
decltype(auto) authAndAccess(Container&amp; c, Index i)
 {
     authenticateUser();
     return c[i];
 }
</code></pre>
<p>함수 반환 형식 auto에 decltype 사용하지 않는다면<br>
형식 연역에 의하여  c[i]로부터 연역. (int&amp; 에서 참조성을 무시해 int가 됨 ,즉 오른값이 됨.)</p>

