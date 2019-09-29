---
title: 使用scp命令从linux服务器拷贝文件到windows
author: ShadoWalker
type: post
date: 2018-11-01T08:27:36+00:00
url: /?p=337
categories:
  - 转载

---
**以下内容转载自互联网，具体出处我实在是记不清了，原作者请给我留言。**

&nbsp;

1.首先，下载putty软件，并可以在目录中，找到pscp.exe文件，我们可以通过这个软件实现Windows和linux之间拷贝文件。
  
<img class="alignnone size-full wp-image-338" src="https://blog.tracewalker.com/wp-content/uploads/2018/11/1.png" alt="" width="600" height="352" srcset="/images/2018/11/1.png 600w, /images/2018/11/1-300x176.png 300w" sizes="(max-width: 600px) 100vw, 600px" />

2.首先，将pscp.exe的路径加入到系统环境变量Path中，这样我们就可以在Windows的命令行下使用pscp命令了。
  
<img class="alignnone size-full wp-image-339" src="https://blog.tracewalker.com/wp-content/uploads/2018/11/2.png" alt="" width="600" height="320" srcset="/images/2018/11/2.png 600w, /images/2018/11/2-300x160.png 300w" sizes="(max-width: 600px) 100vw, 600px" />

3.按下Windows键+R，输入cmd然后回车，既可进入命令行模式。
  
<img class="alignnone size-full wp-image-340" src="https://blog.tracewalker.com/wp-content/uploads/2018/11/3.png" alt="" width="523" height="295" srcset="/images/2018/11/3.png 523w, /images/2018/11/3-300x169.png 300w" sizes="(max-width: 523px) 100vw, 523px" />

4.此时使用pscp命令既可以拷贝文件到远端的Linux系统中，或者从远端的Linux系统中拷贝文件到当前路径，该命令使用方法类似于Linux下的scp命令。
  
<img class="alignnone size-full wp-image-341" src="https://blog.tracewalker.com/wp-content/uploads/2018/11/4.png" alt="" width="600" height="210" srcset="/images/2018/11/4.png 600w, /images/2018/11/4-300x105.png 300w" sizes="(max-width: 600px) 100vw, 600px" />

5.输入正确的密码，就可以完成拷贝了。
  
<img class="alignnone size-full wp-image-342" src="https://blog.tracewalker.com/wp-content/uploads/2018/11/5.png" alt="" width="600" height="71" srcset="/images/2018/11/5.png 600w, /images/2018/11/5-300x36.png 300w" sizes="(max-width: 600px) 100vw, 600px" />

&nbsp;

Linux下scp命令使用教程：
  
1.首先我们启动两台ubuntu系统的设备，并且确保两台设备都开启了ssh远程登录，且两台设备能互相通信。然后我们介绍第一条命令将本地的目录上传的远程服务器目录上。执行命令"scp -r /opt/test root@192.168.2.105:/opt"。本条命令意思为将本地的目录/opt/test上传到远程192.168.2.105的opt目录下。然后根据提示输入root的密码，等待即可上传完毕。
  
<img class="alignnone size-full wp-image-343" src="https://blog.tracewalker.com/wp-content/uploads/2018/11/1-1.png" alt="" width="600" height="157" srcset="/images/2018/11/1-1.png 600w, /images/2018/11/1-1-300x79.png 300w" sizes="(max-width: 600px) 100vw, 600px" />

2.接下来我们登录远程服务器进行查看验证，可以看到目录内的文件已经全部拷贝过来。
  
<img class="alignnone size-full wp-image-344" src="https://blog.tracewalker.com/wp-content/uploads/2018/11/2-1.png" alt="" width="600" height="179" srcset="/images/2018/11/2-1.png 600w, /images/2018/11/2-1-300x90.png 300w" sizes="(max-width: 600px) 100vw, 600px" />

3.下面我们介绍将本地的文件上传到远程服务器上。执行命令"scp /root/node-v4.2.1-linux-x64.tar.gz root@192.168.2.105:/opt/test"。意思为将本地文件node-v4.2.1-linux-x64.tar.gz上传到服务器/opt/test目录下。
  
<img class="alignnone size-full wp-image-345" src="https://blog.tracewalker.com/wp-content/uploads/2018/11/3-1.png" alt="" width="600" height="73" srcset="/images/2018/11/3-1.png 600w, /images/2018/11/3-1-300x37.png 300w" sizes="(max-width: 600px) 100vw, 600px" />

4.下面我们再次验证是否真正的上传成功。登录远程服务器进行查看，可以看到已经拷贝成功。
  
<img class="alignnone size-full wp-image-346" src="https://blog.tracewalker.com/wp-content/uploads/2018/11/4-1.png" alt="" width="600" height="115" srcset="/images/2018/11/4-1.png 600w, /images/2018/11/4-1-300x58.png 300w" sizes="(max-width: 600px) 100vw, 600px" />

5.下面我们介绍如何将远程服务器的目录，拷贝到本地。执行命令"scp -r root@192.168.2.105:/root/rules /opt"。意思为将远程服务器上/root/rules目录内的内容拷贝到本地的opt目录下。
  
<img class="alignnone size-full wp-image-347" src="https://blog.tracewalker.com/wp-content/uploads/2018/11/5-1.png" alt="" width="600" height="205" srcset="/images/2018/11/5-1.png 600w, /images/2018/11/5-1-300x103.png 300w" sizes="(max-width: 600px) 100vw, 600px" />

6.接下来还是对拷贝结果的验证，我们进入/opt目录下，可以看到rules目录以及目录下的文件都拷贝过来了。
  
<img class="alignnone size-full wp-image-348" src="https://blog.tracewalker.com/wp-content/uploads/2018/11/6.png" alt="" width="600" height="105" srcset="/images/2018/11/6.png 600w, /images/2018/11/6-300x53.png 300w" sizes="(max-width: 600px) 100vw, 600px" />

7.下面我们介绍如何将远程服务器上的文件拷贝的本地。我们执行命令"scp -P 22 root@192.168.2.105:/root/filters.bpf /opt/"。意思为将远程服务器上/root目录下的filters.bpf文件拷贝到本地的opt目录下。
  
<img class="alignnone size-full wp-image-349" src="https://blog.tracewalker.com/wp-content/uploads/2018/11/7.png" alt="" width="600" height="54" srcset="/images/2018/11/7.png 600w, /images/2018/11/7-300x27.png 300w" sizes="(max-width: 600px) 100vw, 600px" />

8.下面还是验证是否拷贝过来，我们进入opt目录，可以查看到filters.bpf已经拷贝过来。
  
<img class="alignnone size-full wp-image-350" src="https://blog.tracewalker.com/wp-content/uploads/2018/11/8.png" alt="" width="600" height="122" srcset="/images/2018/11/8.png 600w, /images/2018/11/8-300x61.png 300w" sizes="(max-width: 600px) 100vw, 600px" />