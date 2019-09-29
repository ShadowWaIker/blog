---
title: 详解ip2long和long2ip（PHP）
author: ShadoWalker
type: post
date: 2018-10-31T14:34:43+00:00
url: /?p=328
categories:
  - 转载

---
偶然在开源中国看到了这篇文章，写的很不错所以转载收藏之。

转载自：https://my.oschina.net/goal/blog/198049

&nbsp;

在开发中，经常需要将IP地址转成整型进行保存，这样不仅有利于做索引，并且原本需要15个字节的存储空间，转换后只需4个字节就能存储了。但是很多人对于ip2long的结果有时候是负数并不理解，本文将详细解释这一点。因为ip2long只支持IPv4，所以本文也是基于IPv4来描述和编码的。

## 右移 {#h2_1}

### 逻辑右移 {#h3_2}

右移多少位，则在高位补多少位0。

### 算术右移 {#h3_3}

对无符号数做算术右移和逻辑右移的结果是相同的。但是对一个有符号数做算术右移，则右移多少位，即在高位补多少位1。

### 注意事项 {#h3_4}

对于C来说，只提供了>>右移运算符，究竟是逻辑右移还是算术右移这取决于编译器的行为，因此一般只提倡对无符号数进行位操作。

## IPv4地址是如何表示的 {#h2_5}

IPv4使用无符号32位地址，因此最多有2的32次方减1(4294967295)个地址。一般的书写法为用4个小数点分开的十进制数，记为：A.B.C.D，比如：157.23.56.90。

## IPv4地址转换成无符号整型 {#h2_6}

![][1]

IPv4地址的每一个十进制数都为无符号的字节，因此范围在0~255，将IPv4地址转成无符号整型其实就是将每个十进制数放在对应的8位上组成一个4字节的无符号整型。依上图表示：157在高8位，90在低8位，23和56在中间对应的8位上。来看一个C实现的例子：

<pre class="brush:cpp;toolbar: true; auto-links: false;"><code class="hljs cpp">&lt;span class="hljs-meta">#&lt;span class="hljs-meta-keyword">include&lt;/span> &lt;span class="hljs-meta-string">&lt;stdio.h&gt;&lt;/span>&lt;/span>

&lt;span class="hljs-function">&lt;span class="hljs-keyword">int&lt;/span> &lt;span class="hljs-title">main&lt;/span>&lt;span class="hljs-params">(&lt;span class="hljs-keyword">int&lt;/span> argc, &lt;span class="hljs-keyword">char&lt;/span>** argv)&lt;/span>
&lt;/span>{
	&lt;span class="hljs-keyword">unsigned&lt;/span> &lt;span class="hljs-keyword">int&lt;/span> ip_long = (&lt;span class="hljs-number">157&lt;/span> &lt;&lt; &lt;span class="hljs-number">24&lt;/span>) | (&lt;span class="hljs-number">23&lt;/span> &lt;&lt; &lt;span class="hljs-number">16&lt;/span>) | (&lt;span class="hljs-number">56&lt;/span> &lt;&lt; &lt;span class="hljs-number">8&lt;/span>) | &lt;span class="hljs-number">90&lt;/span>;
	&lt;span class="hljs-built_in">printf&lt;/span>(&lt;span class="hljs-string">"%u\n"&lt;/span>, ip_long);
	&lt;span class="hljs-built_in">printf&lt;/span>(&lt;span class="hljs-string">"%d\n"&lt;/span>, ip_long);

	&lt;span class="hljs-keyword">return&lt;/span> &lt;span class="hljs-number">0&lt;/span>;
}</code></pre>

<pre class="brush:shell;toolbar: true; auto-links: false;"><code class="hljs shell">&lt;span class="hljs-meta">$&lt;/span>&lt;span class="bash"> gcc -o ip2long main.c&lt;/span>
&lt;span class="hljs-meta">$&lt;/span>&lt;span class="bash"> ./ip2long&lt;/span>
2635544666
-1659422630</code></pre>

可以看到，即使ip\_long声明为无符号整型，在输出时也需要指明%u来格式化输出为无符号整型。这是因为157大于127(二进制为01111111)，也就是说如果157(8位)用二进制来表示，最高位必然是1。当将157放在一个4字节整型的高8位时，导致这个4字节整型的最高位为1。虽然ip\_long定义为无符号整型，但printf函数并不知道，因此需要指明无符号格式化字符。如果最高位为0，则使用%d就可以了，来看另一个例子：

<pre class="brush:cpp;toolbar: true; auto-links: false;"><code class="hljs cpp">&lt;span class="hljs-meta">#&lt;span class="hljs-meta-keyword">include&lt;/span> &lt;span class="hljs-meta-string">&lt;stdio.h&gt;&lt;/span>&lt;/span>

&lt;span class="hljs-function">&lt;span class="hljs-keyword">int&lt;/span> &lt;span class="hljs-title">main&lt;/span>&lt;span class="hljs-params">(&lt;span class="hljs-keyword">int&lt;/span> argc, &lt;span class="hljs-keyword">char&lt;/span>** argv)&lt;/span>
&lt;/span>{
	&lt;span class="hljs-keyword">unsigned&lt;/span> &lt;span class="hljs-keyword">int&lt;/span> ip_long = (&lt;span class="hljs-number">120&lt;/span> &lt;&lt; &lt;span class="hljs-number">24&lt;/span>) | (&lt;span class="hljs-number">23&lt;/span> &lt;&lt; &lt;span class="hljs-number">16&lt;/span>) | (&lt;span class="hljs-number">56&lt;/span> &lt;&lt; &lt;span class="hljs-number">8&lt;/span>) | &lt;span class="hljs-number">90&lt;/span>;
	&lt;span class="hljs-built_in">printf&lt;/span>(&lt;span class="hljs-string">"%u\n"&lt;/span>, ip_long);
	&lt;span class="hljs-built_in">printf&lt;/span>(&lt;span class="hljs-string">"%d\n"&lt;/span>, ip_long);

	&lt;span class="hljs-keyword">return&lt;/span> &lt;span class="hljs-number">0&lt;/span>;
}</code></pre>

<pre class="brush:shell;toolbar: true; auto-links: false;"><code class="hljs shell">&lt;span class="hljs-meta">$&lt;/span>&lt;span class="bash"> gcc -o ip2long main.c&lt;/span>
&lt;span class="hljs-meta">$&lt;/span>&lt;span class="bash"> ./ip2long&lt;/span>
2014787674
2014787674</code></pre>

## PHP是如何做的 {#h2_7}

现在已经知道了为什么会出现负数。对于动态类型语言来说，数据类型一般是有符号的，所以需要我们自己转成无符号整型。PHP有内置函数ip2long来将IPv4地址转换成整型，也提供了类C的sprintf方法，因此很容易解决出现负数的问题：

<pre class="brush:php;toolbar: true; auto-links: false;"><code class="hljs perl">&lt;?php
echo &lt;span class="hljs-keyword">sprintf&lt;/span>(&lt;span class="hljs-string">"%u\n"&lt;/span>, ip2long(&lt;span class="hljs-string">"157.23.56.90"&lt;/span>));</code></pre>

<pre class="brush:shell;toolbar: true; auto-links: false;"><code class="hljs shell">&lt;span class="hljs-meta">$&lt;/span>&lt;span class="bash"> php -f test.php&lt;/span>
2635544666</code></pre>

## JavaScript是如何做的 {#h2_8}

JavaScript既没有提供ip2long方法，也没有提供类C的格式化函数。但JavaScript却同时提供了逻辑右移(>>>)和算术右移(>>)运算符，所以解决的方法也很简单，对结果再跟0做逻辑右移即可：

<pre class="brush:js;toolbar: true; auto-links: false;"><code class="hljs xml">&lt;span class="hljs-tag">&lt;&lt;span class="hljs-name">script&lt;/span> &lt;span class="hljs-attr">type&lt;/span>=&lt;span class="hljs-string">"text/javascript"&lt;/span>&gt;&lt;/span>&lt;span class="javascript">
&lt;span class="hljs-built_in">console&lt;/span>.log(((&lt;span class="hljs-number">157&lt;/span> &lt;&lt; &lt;span class="hljs-number">24&lt;/span>) | (&lt;span class="hljs-number">23&lt;/span> &lt;&lt; &lt;span class="hljs-number">16&lt;/span>) | (&lt;span class="hljs-number">56&lt;/span> &lt;&lt; &lt;span class="hljs-number">8&lt;/span>) | &lt;span class="hljs-number">90&lt;/span>) &gt;&gt;&gt; &lt;span class="hljs-number">0&lt;/span>);
&lt;/span>&lt;span class="hljs-tag">&lt;/&lt;span class="hljs-name">script&lt;/span>&gt;&lt;/span></code></pre>

<pre class="brush:js;toolbar: true; auto-links: false;"><code class="hljs">2635544666</code></pre>

## PHP和JavaScript的long2ip的实现 {#h2_9}

有了前面的知识，long2ip的实现就很简单了。只须从ip2long的结果中取出每8位形成的十进制数，再用点(.)连接就可以了。之前的例子都是用IP(157.23.56.90)来举例的，它的ip2long的结果是：2635544666。

### PHP的long2ip的实现 {#h3_10}

<pre class="brush:php;toolbar: true; auto-links: false;"><code class="hljs xml">&lt;span class="php">&lt;span class="hljs-meta">&lt;?php&lt;/span>
$ip_long = &lt;span class="hljs-number">2635544666&lt;/span>;
&lt;span class="hljs-keyword">echo&lt;/span> long2ip($ip_long) . &lt;span class="hljs-string">"\n"&lt;/span>;&lt;/span></code></pre>

<pre class="brush:shell;toolbar: true; auto-links: false;"><code class="hljs css">&lt;span class="hljs-selector-tag">php&lt;/span> &lt;span class="hljs-selector-tag">-f&lt;/span> &lt;span class="hljs-selector-tag">test&lt;/span>&lt;span class="hljs-selector-class">.php&lt;/span>
157&lt;span class="hljs-selector-class">.23&lt;/span>&lt;span class="hljs-selector-class">.56&lt;/span>&lt;span class="hljs-selector-class">.90&lt;/span></code></pre>

以上代码是由PHP的内置函数long2ip来实现的。但是对于想通过移位来自己实现的童鞋来说，可能没有那么简单。因为PHP的>>运算符是算术右移运算符，所以如果最高位是1的话，右移的结果是在高位补1，这跟结果不符。但是我们可以用另一种思路去解决：保存最高位(符号位)，然后将最高位置0，之后再将高8位的最高位置1(这取决于之前保存的符号位)。代码实现如下：

<pre class="brush:php;toolbar: true; auto-links: false;"><code class="hljs xml">&lt;span class="php">&lt;span class="hljs-meta">&lt;?php&lt;/span>
$ip_long = &lt;span class="hljs-number">2635544666&lt;/span>;
&lt;span class="hljs-comment">// 保存最高位(符号位)&lt;/span>
$msb = &lt;span class="hljs-number">0&lt;/span>;
&lt;span class="hljs-keyword">if&lt;/span> ($ip_long & &lt;span class="hljs-number">0x80000000&lt;/span>)
{
	$msb = &lt;span class="hljs-number">1&lt;/span>;
}

&lt;span class="hljs-comment">// 将最高位(符号位)置0变成无符号数&lt;/span>
$uip_long = $ip_long & &lt;span class="hljs-number">0x7fffffff&lt;/span>;
$ip1 = $uip_long &gt;&gt; &lt;span class="hljs-number">24&lt;/span>;
&lt;span class="hljs-keyword">if&lt;/span> ($msb == &lt;span class="hljs-number">1&lt;/span>)
{
	$ip1 |= &lt;span class="hljs-number">0x80&lt;/span>;
}

$ip2 = ($uip_long &gt;&gt; &lt;span class="hljs-number">16&lt;/span>) & &lt;span class="hljs-number">0xff&lt;/span>; &lt;span class="hljs-comment">// 跟0xff做与运算的目的是取低8位&lt;/span>
$ip3 = ($uip_long &gt;&gt; &lt;span class="hljs-number">8&lt;/span>) & &lt;span class="hljs-number">0xff&lt;/span>;
$ip4 = $uip_long & &lt;span class="hljs-number">0xff&lt;/span>;
&lt;span class="hljs-keyword">echo&lt;/span> $ip1 . &lt;span class="hljs-string">'.'&lt;/span> . $ip2 . &lt;span class="hljs-string">'.'&lt;/span> . $ip3 . &lt;span class="hljs-string">'.'&lt;/span> . $ip4 . &lt;span class="hljs-string">"\n"&lt;/span>;&lt;/span></code></pre>

<pre class="brush:shell;toolbar: true; auto-links: false;"><code class="hljs shell">&lt;span class="hljs-meta">$&lt;/span>&lt;span class="bash"> php -f test.php&lt;/span>
157.23.56.90</code></pre>

虽然以上代码能得到正确的结果，但是并不推荐这样做。因为以上代码是假设PHP中的数据类型是32位的。这样将会有移植性问题。我们可以跟0xff做与运算来取得低8位，这样做的好处是兼容性好。代码如下：

<pre class="brush:php;toolbar: true; auto-links: false;"><code class="hljs ruby">&lt;?php
$ip_long = &lt;span class="hljs-number">2635544666&lt;/span>;
$ip1 = ($ip_long &lt;span class="hljs-meta">&gt;&gt; &lt;/span>&lt;span class="hljs-number">24&lt;/span>) & &lt;span class="hljs-number">0xff&lt;/span>; &lt;span class="hljs-regexp">//&lt;/span> 跟&lt;span class="hljs-number">0xff&lt;/span>做与运算的目的是取低&lt;span class="hljs-number">8&lt;/span>位
$ip2 = ($ip_long &lt;span class="hljs-meta">&gt;&gt; &lt;/span>&lt;span class="hljs-number">16&lt;/span>) & &lt;span class="hljs-number">0xff&lt;/span>;
$ip3 = ($ip_long &gt;&gt; &lt;span class="hljs-number">8&lt;/span>) & &lt;span class="hljs-number">0xff&lt;/span>;
$ip4 = $ip_long & &lt;span class="hljs-number">0xff&lt;/span>;
echo $ip1 . &lt;span class="hljs-string">'.'&lt;/span> . $ip2 . &lt;span class="hljs-string">'.'&lt;/span> . $ip3 . &lt;span class="hljs-string">'.'&lt;/span> . $ip4 . &lt;span class="hljs-string">"\n"&lt;/span>;</code></pre>

<pre class="brush:shell;toolbar: true; auto-links: false;"><code class="hljs shell">&lt;span class="hljs-meta">$&lt;/span>&lt;span class="bash"> php -f test.php&lt;/span>
157.23.56.90</code></pre>

另外还可以通过pack和unpack方法来实现，但要注意的是IPv4应使用大端序。

### JavaScript的long2ip的实现 {#h3_11}

<pre class="brush:js;toolbar: true; auto-links: false;"><code class="hljs xml">&lt;span class="hljs-tag">&lt;&lt;span class="hljs-name">script&lt;/span> &lt;span class="hljs-attr">type&lt;/span>=&lt;span class="hljs-string">"text/javascript"&lt;/span>&gt;&lt;/span>&lt;span class="javascript">
&lt;span class="hljs-keyword">var&lt;/span> ip_long = &lt;span class="hljs-number">2635544666&lt;/span>;
&lt;span class="hljs-keyword">var&lt;/span> ip1 = (ip_long &gt;&gt; &lt;span class="hljs-number">24&lt;/span>) & &lt;span class="hljs-number">0xff&lt;/span>;
&lt;span class="hljs-keyword">var&lt;/span> ip2 = (ip_long &gt;&gt; &lt;span class="hljs-number">16&lt;/span>) & &lt;span class="hljs-number">0xff&lt;/span>;
&lt;span class="hljs-keyword">var&lt;/span> ip3 = (ip_long &gt;&gt; &lt;span class="hljs-number">8&lt;/span>) & &lt;span class="hljs-number">0xff&lt;/span>;
&lt;span class="hljs-keyword">var&lt;/span> ip4 = ip_long & &lt;span class="hljs-number">0xff&lt;/span>;
&lt;span class="hljs-built_in">console&lt;/span>.log(ip1 + &lt;span class="hljs-string">"."&lt;/span> + ip2 + &lt;span class="hljs-string">"."&lt;/span> + ip3 + &lt;span class="hljs-string">"."&lt;/span> + ip4);
&lt;/span>&lt;span class="hljs-tag">&lt;/&lt;span class="hljs-name">script&lt;/span>&gt;&lt;/span></code></pre>

<pre class="brush:js;toolbar: true; auto-links: false;"><code class="hljs css">157&lt;span class="hljs-selector-class">.23&lt;/span>&lt;span class="hljs-selector-class">.56&lt;/span>&lt;span class="hljs-selector-class">.90&lt;/span></code></pre>

知其然，而知其所以然。这样以后就不会为ip2long的结果是负数而感到惊讶了。

 [1]: http://static.oschina.net/uploads/space/2014/0209/143546_x7Tg_182025.jpg