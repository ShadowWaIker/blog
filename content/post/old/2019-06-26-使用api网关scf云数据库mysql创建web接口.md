---
title: 20190625|使用API网关+SCF+云数据库MySQL创建WEB接口
author: ShadoWalker
type: post
date: 2019-06-25T16:28:47+00:00
tags:
  - 笔记
---

#### 1. 创建私有网络并进行配置

首先需要创建一个私有网络，因为在默认情况下云数据库MySQL只可以在私有网络中使用，这也是腾讯云官方的推荐使用方式。

在本文中，使用了默认的私有网络，因此跳过这一步骤。如果有特殊需求，可以先自行创建一个私有网络，注意私有网络所在的区域要与SCF及云数据库MySQL一致。

#### 2. 创建云数据库MySQL实例选择

略，这一步没有碰到什么坑，直接按照步骤选择套餐、购买、初始化并保存好数据库的密码即可。

#### 3. 使用SCF模板创建函数

打开SCF无服务器云函数模块，选择新建函数。

创建方式选择模板函数，在模板搜索中填写“SQL”并点击进行搜索。

这里选用“MySQL基础使用Demo”这一Demo函数用于测试，点击“下一步”，再点击“完成”后完成函数的创建。<figure class="wp-block-image">

<img src="/images/2019/06/图片-4.png" alt="" class="wp-image-388" srcset="/images/2019/06/图片-4.png 989w, /images/2019/06/图片-4-300x254.png 300w, /images/2019/06/图片-4-768x650.png 768w" sizes="(max-width: 989px) 100vw, 989px" /></figure> 

打开函数代码，填写MySQL账号信息，点击“保存”，这里**先不进行测试**。

\# 这里需要注意，**还要给函数的第73行增加一个缩进**，这一行的course.close()脱离了函数的作用域。会令数据库事务不能被正常关闭(?)，从而导致测试表被锁死无法再次操作。然后就会出现SCF函数第一次测试能够成功，后续再进行测试就会超时的情况。<figure class="wp-block-image">

<img src="/images/2019/06/图片-5.png" alt="" class="wp-image-389" srcset="/images/2019/06/图片-5.png 400w, /images/2019/06/图片-5-300x75.png 300w" sizes="(max-width: 400px) 100vw, 400px" /></figure> 

#### 4. 修改函数的网络设置并进行测试

<p class="has-text-color has-cyan-bluish-gray-color">
  因为SCF默认的网络设置无法访问内网的资源，所以需要修改函数的网络设置，使其可以访问内网的资源。但是在修改网络设置后函数将无法访问公网资源，后文中将会讲到如何恢复函数对公网的访问能力。
</p>

点击函数，打开函数详情，在函数配置面板点击编辑按钮。<figure class="wp-block-image">

<img src="/images/2019/06/图片-6.png" alt="" class="wp-image-390" srcset="/images/2019/06/图片-6.png 997w, /images/2019/06/图片-6-300x69.png 300w, /images/2019/06/图片-6-768x177.png 768w" sizes="(max-width: 997px) 100vw, 997px" /></figure> 

修改网络配置参数，设置为前文中创建的私有网络。<figure class="wp-block-image">

<img src="/images/2019/06/图片-7.png" alt="" class="wp-image-391" srcset="/images/2019/06/图片-7.png 529w, /images/2019/06/图片-7-300x54.png 300w" sizes="(max-width: 529px) 100vw, 529px" /></figure> 

返回代码选项卡，点击“测试”，进行函数的测试。<figure class="wp-block-image">

<img src="/images/2019/06/图片-8.png" alt="" class="wp-image-392" /></figure> 

成功连接到云数据库MySQL。 

#### 5. 配置API网关

略，这一步没有碰到什么坑，直接参考 [API网关使用指南][1] 即可。

<hr class="wp-block-separator is-style-wide" />

<p class="has-text-color has-vivid-red-color">
  TIPS： <strong>如果您开发的WEB接口无需访问公网，则无需阅读以下内容。</strong>
</p>

如果您编写的WEB接口需要访问公网（例如调用微信公众平台的接口等），则需要为私有网络配置NAT网关，因为如果在SCF云函数的网络配置中设置了私有网络后，SCF将无法访问公网资源，只有为私有网络配置了NAT网关后SCF才能恢复对公网的访问。

#### 6. 配置NAT网关

打开私有网络模块，在左侧的菜单栏找到NAT网关并点击。点击新建按钮，根据实际需求设置好参数后点击创建按钮完成网关的创建。<figure class="wp-block-image">

<img src="/images/2019/06/图片-3.png" alt="" class="wp-image-386" srcset="/images/2019/06/图片-3.png 887w, /images/2019/06/图片-3-300x241.png 300w, /images/2019/06/图片-3-768x618.png 768w" sizes="(max-width: 887px) 100vw, 887px" /></figure> 

接下来还需要设置路由表。否则即使创建好了NAT网关后，私有网络内的资源还是不能直接访问公网。

点击页面左侧的“路由表”，点击“新建”， “目的端”填写“0.0.0.0/0”、“下一跳类型”选择“NAT网关”、“下一跳”选择刚才创建的NAT网关、备注可留空。<figure class="wp-block-image">

<img src="/images/2019/06/图片-2-1024x679.png" alt="" class="wp-image-385" srcset="/images/2019/06/图片-2-1024x679.png 1024w, /images/2019/06/图片-2-300x199.png 300w, /images/2019/06/图片-2-768x509.png 768w, /images/2019/06/图片-2.png 1108w" sizes="(max-width: 1024px) 100vw, 1024px" /></figure> 

点击“创建”完成配置。到了这一步，私有网络内的资源即可通过这个NAT网关来访问公网的资源了。

可参考：<https://cloud.tencent.com/document/product/583/19704> 。

TIPS：SCF如何配置才能同时访问云数据库MySQL和公网资源是我咨询了“腾讯云SCF用户群”里的群友后才得知的，另外据这位群友说，~~NAT网关的使用时长的计费是按使用时间来累计计算的（即：累计使用够1H后收取1H的费用。而不是只要是在这一小时有使用就会收取这1H的费用，过了几个小时后再次使用又会收取1H的费用。还是比较的划算的。）~~，特别在这里感谢这位xixi兄嘚，2333.

#### 7. 修改SCF函数进行并进行测试

打开前文中使用Demo模板创建的函数，增加以下代码。

<pre class="wp-block-code"><code>    import requests
    print('访问baidu网站 获取Response对象\n')
    r = requests.get("http://www.baidu.com")
    print(r.status_code)
    print(r.encoding)
    print(r.apparent_encoding)
    print('将对象编码转换成UTF-8编码并打印出来')
    r.encoding = 'utf-8'
    print(r.text)</code></pre><figure class="wp-block-image">

<img src="/images/2019/06/图片-1.png" alt="" class="wp-image-384" srcset="/images/2019/06/图片-1.png 657w, /images/2019/06/图片-1-300x159.png 300w" sizes="(max-width: 657px) 100vw, 657px" /></figure> 

点击“保存并测试”，测试是否可以访问公网资源。<figure class="wp-block-image">

<img src="/images/2019/06/图片-1024x333.png" alt="" class="wp-image-383" srcset="/images/2019/06/图片-1024x333.png 1024w, /images/2019/06/图片-300x98.png 300w, /images/2019/06/图片-768x250.png 768w, /images/2019/06/图片.png 1331w" sizes="(max-width: 1024px) 100vw, 1024px" /></figure> 

成功连接到数据库，并可访问公网资源。

## 补充

> 20190930: 经过实测， NAT 网关只要开启后即收取 `0.50元/小时` 的费用。

&nbsp;
 [1]: https://cloud.tencent.com/document/product/628