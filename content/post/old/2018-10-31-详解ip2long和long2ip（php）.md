---
title: 详解ip2long和long2ip（PHP）
author: ShadoWalker
type: post
date: 2018-10-31T14:34:43+00:00
tags:
  - 转载
---

偶然在开源中国看到了这篇文章，写的很不错所以转载收藏之。

转载自：https://my.oschina.net/goal/blog/198049

&nbsp;

在开发中，经常需要将IP地址转成整型进行保存，这样不仅有利于做索引，并且原本需要15个字节的存储空间，转换后只需4个字节就能存储了。但是很多人对于ip2long的结果有时候是负数并不理解，本文将详细解释这一点。因为ip2long只支持IPv4，所以本文也是基于IPv4来描述和编码的。

## 右移

### 逻辑右移

右移多少位，则在高位补多少位0。

### 算术右移

对无符号数做算术右移和逻辑右移的结果是相同的。但是对一个有符号数做算术右移，则右移多少位，即在高位补多少位1。

### 注意事项

对于C来说，只提供了 `>>` 右移运算符，究竟是逻辑右移还是算术右移这取决于编译器的行为，因此一般只提倡对无符号数进行位操作。

## IPv4地址是如何表示的

IPv4使用无符号32位地址，因此最多有2的32次方减1(4294967295)个地址。一般的书写法为用4个小数点分开的十进制数，记为：A.B.C.D，比如：157.23.56.90。

## IPv4地址转换成无符号整型

![][1]

IPv4地址的每一个十进制数都为无符号的字节，因此范围在0~255，将IPv4地址转成无符号整型其实就是将每个十进制数放在对应的8位上组成一个4字节的无符号整型。依上图表示：157在高8位，90在低8位，23和56在中间对应的8位上。来看一个C实现的例子：

```go-html-template
#include <stdio.h>

int main(int argc, char** argv)
{
	unsigned int ip_long = (157 << 24) | (23 << 16) | (56 << 8) | 90;
	printf("%u\n", ip_long);
	printf("%d\n", ip_long);

	return 0;
}
```

```go-html-template
$ gcc -o ip2long main.c
$ ./ip2long
2635544666
-1659422630
```

可以看到，即使ip\_long声明为无符号整型，在输出时也需要指明%u来格式化输出为无符号整型。这是因为157大于127(二进制为01111111)，也就是说如果157(8位)用二进制来表示，最高位必然是1。当将157放在一个4字节整型的高8位时，导致这个4字节整型的最高位为1。虽然ip\_long定义为无符号整型，但printf函数并不知道，因此需要指明无符号格式化字符。如果最高位为0，则使用%d就可以了，来看另一个例子：

```go-html-template
#include <stdio.h>

int main(int argc, char** argv)
{
	unsigned int ip_long = (120 << 24) | (23 << 16) | (56 << 8) | 90;
	printf("%u\n", ip_long);
	printf("%d\n", ip_long);

	return 0;
}
```

```go-html-template
$ gcc -o ip2long main.c
$ ./ip2long
2014787674
2014787674
```

## PHP是如何做的

现在已经知道了为什么会出现负数。对于动态类型语言来说，数据类型一般是有符号的，所以需要我们自己转成无符号整型。PHP有内置函数ip2long来将IPv4地址转换成整型，也提供了类C的sprintf方法，因此很容易解决出现负数的问题：

```go-html-template
<?php
	echo sprintf("%u\n", ip2long("157.23.56.90"));
```

```go-html-template
$ php -f test.php
2635544666
```

## JavaScript是如何做的

JavaScript既没有提供ip2long方法，也没有提供类C的格式化函数。但JavaScript却同时提供了逻辑右移(>>>)和算术右移(>>)运算符，所以解决的方法也很简单，对结果再跟0做逻辑右移即可：

```go-html-template
<script type="text/javascript">
	console.log(((157 << 24) | (23 << 16) | (56 << 8) | 90) >>> 0);
</script>
```

```go-html-template
2635544666
```

## PHP和JavaScript的long2ip的实现

有了前面的知识，long2ip的实现就很简单了。只须从ip2long的结果中取出每8位形成的十进制数，再用点(.)连接就可以了。之前的例子都是用IP(157.23.56.90)来举例的，它的ip2long的结果是：2635544666。

### PHP的long2ip的实现

```go-html-template
<?php
	$ip_long = 2635544666;
	echo long2ip($ip_long) . "\n";
```

```go-html-template
php -f test.php
157.23.56.90
```

以上代码是由PHP的内置函数long2ip来实现的。但是对于想通过移位来自己实现的童鞋来说，可能没有那么简单。因为PHP的>>运算符是算术右移运算符，所以如果最高位是1的话，右移的结果是在高位补1，这跟结果不符。但是我们可以用另一种思路去解决：保存最高位(符号位)，然后将最高位置0，之后再将高8位的最高位置1(这取决于之前保存的符号位)。代码实现如下：

```go-html-template
<?php
	$ip_long = 2635544666;
	// 保存最高位(符号位)
	$msb = 0;
	if ($ip_long & 0x80000000)
	{
		$msb = 1;
	}

	// 将最高位(符号位)置0变成无符号数
	$uip_long = $ip_long & 0x7fffffff;
	$ip1 = $uip_long >> 24;
	if ($msb == 1)
	{
		$ip1 |= 0x80;
	}

	$ip2 = ($uip_long >> 16) & 0xff; // 跟0xff做与运算的目的是取低8位
	$ip3 = ($uip_long >> 8) & 0xff;
	$ip4 = $uip_long & 0xff;
	echo $ip1 . '.' . $ip2 . '.' . $ip3 . '.' . $ip4 . "\n";
```

```go-html-template
$ php -f test.php
157.23.56.90
```

虽然以上代码能得到正确的结果，但是并不推荐这样做。因为以上代码是假设PHP中的数据类型是32位的。这样将会有移植性问题。我们可以跟0xff做与运算来取得低8位，这样做的好处是兼容性好。代码如下：

```go-html-template
<?php
	$ip_long = 2635544666;
	$ip1 = ($ip_long >> 24) & 0xff; // 跟0xff做与运算的目的是取低8位
	$ip2 = ($ip_long >> 16) & 0xff;
	$ip3 = ($ip_long >> 8) & 0xff;
	$ip4 = $ip_long & 0xff;
	echo $ip1 . '.' . $ip2 . '.' . $ip3 . '.' . $ip4 . "\n";
```

```go-html-template
$ php -f test.php
157.23.56.90
```

另外还可以通过pack和unpack方法来实现，但要注意的是IPv4应使用大端序。

### JavaScript的long2ip的实现

```go-html-template
<script type="text/javascript">
	var ip_long = 2635544666
	var ip1 = (ip_long >> 24) & 0xff
	var ip2 = (ip_long >> 16) & 0xff
	var ip3 = (ip_long >> 8) & 0xff
	var ip4 = ip_long & 0xff
	console.log(ip1 + "." + ip2 + "." + ip3 + "." + ip4);
</script>
```
`157.23.56.90`

知其然，而知其所以然。这样以后就不会为ip2long的结果是负数而感到惊讶了。

&nbsp;

 [1]: /images/2019/09/143546_x7Tg_182025.jpg