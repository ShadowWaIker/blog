---
title: 20160420|CentOS(Linux)中设置Apache不敏感大小写
author: ShadoWalker
type: post
date: 2016-04-21T02:52:43+00:00
url: /?p=33
categories:
  - 笔记

---
<p class="p1">
  用过Linux系统的都知道Linux是区分大小写的，在Linux服务器上部署网站，如果URL大小写敏感，对用户和搜索引擎来说是非常不友好的。
</p>

<p class="p1">
  解决Linux服务器URL大小写问题：
</p>

<p class="p1">
  <strong>1、</strong>查看系统有无mod_speling.so模块，路径：/usr/local/apache2/modules，如果没有按以下方法生成；
</p>

<p class="p1">
  1.1、下载一个与当前使用的apache一样版本的安装包
</p>

<p class="p1">
  1.2、解压安装包，然后进入其目录
</p>

<div class="dp-highlighter">
  <ol class="dp-xml" start="1">
    <li class="alt">
      # cd /tmp/httpd-2.2.22/modules/mappers/
    </li>
    <li class="">
      # ls
    </li>
  </ol>
</div>

<p class="p1">
  此目录有个mod_speling.c文件
</p>

<p class="p1">
  1.3、生成模块
</p>

<div class="dp-highlighter">
  <ol class="dp-xml" start="1">
    <li class="alt">
      # /usr/local/apache/bin/apxs -c -i -a mod_speling.c
    </li>
  </ol>
</div>

<p class="p1">
  注：usr/local/apache/为我生产环境的apache目录，完成上述后会在/usr/local/apache/modules/目录下多一个mod_speling.so模块文件
</p>

<p class="p1">
  <strong>2、</strong>加载此模块
</p>

<div class="dp-highlighter">
  <ol class="dp-xml" start="1">
    <li class="alt">
      # vi /usr/local/apache2/conf/httpd.conf
    </li>
  </ol>
</div>

添加如下行：

<div class="dp-highlighter">
  <ol class="dp-xml" start="1">
    <li class="alt">
      LoadModule speling_module modules/mod_speling.so
    </li>
    <li class="">
      CheckSpelling on
    </li>
  </ol>
</div>

<p class="p1">
  保存修改，退出
</p>

<p class="p1">
  <strong>3、</strong>重启httpd服务。
</p>

<p class="p1">
  # service httpd restart
</p>

<p class="p1">
  完成。
</p>

&nbsp;