---
title: 20170510|CentOS7中Python连接ORACLE数据库的配置
author: ShadoWalker
type: post
date: 2017-05-10T08:15:12+00:00
url: /?p=87
categories:
  - 笔记

---
  1. 安装Python 
      * yum install -y python
  2. 安装Oracle Client 
      * 前往ORACLE官网下载rpm安装包并上传至服务器
      * rpm -ivh oracle-instantclient-basic-10.2.0.5-1.x86_64.rpm
      * rpm -ivh oracle-instantclient-devel-10.2.0.5-1.x86_64.rpm
  3. 测试 
      * python
      * >>> import cx_Oracle
  
        >>> conn=cx\_Oracle.connect('zfxfzb/zfsoft\_xzsy@192.168.100.13/zfxfzb')
  
        >>> print conn.version
  
        10.2.0.1.0
  
        >>> conn.close()
  4. 如果import cx_Oracle出错 
      * export LD\_LIBRARY\_PATH=/usr/lib/oracle/11.2/client64:/usr/lib/oracle/11.2/client64/lib
  5. 参考文献 
      * http://www.cnblogs.com/ylqmf/archive/2012/04/16/2451841.html