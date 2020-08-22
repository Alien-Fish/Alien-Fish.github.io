---
layout: post
title:  "Jekyll如何在Windows上安装"
date:   2019-11-02 14:31:01 +0800
categories: jekyll
tag: jekyll
---

* content
{:toc}


#在Windows上安装Jekyll.

##背景
最近想试试用Jekyll在Github搭建blog。选取网站模板，修改域名等等这些网上都有很详细的教程了，文末会附上链接，这里就不再赘述了。本文主要记录在Windows本地安装jekyll环境的过程，遇到的问题及如何解决的。
## 安装环境
### 1. 安装Ruby

在Windows上使用RubyInstaller安装比较方便，去Ruby官网下载最新版本的RubyInstaller。注意32位和64位版本的区分。

注意：勾选添加到PATH选项，以便在命令行中使用。  
![/styles/images/jekyll/2255514-dec066296c43f6a7.png]({{ '/styles/images/jekyll/2255514-dec066296c43f6a7.png' | prepend: site.baseurl  }})

安装完成界面：  
![/styles/images/jekyll/2255514-98eaec82dfc3c5cb.png]({{ '/styles/images/jekyll/2255514-98eaec82dfc3c5cb.png' | prepend: site.baseurl  }})

这里需要勾选安装msys2，后面安装gem和jekyll时会用到：  
![/styles/images/jekyll/2255514-7e92a90e54ebc783.png]({{ '/styles/images/jekyll/2255514-7e92a90e54ebc783.png' | prepend: site.baseurl  }})

安装msys2和development toolchain
### 2. 安装RubyGems

Windows中下载ZIP格式比较方便，下载后解压到任意路径。打开Windows的cmd界面，输入命令：
```bash
$ cd {unzip-path} // unzip-path表示解压的路径
$ ruby setup.rb
```

### 3. 安装Jekyll

在cmd中输入:
```bash
$ gem install jekyll
```

### 4. 安装jekyll-paginate

在cmd中输入：
```bash
$ gem install jekyll-paginate
```

### 5. 验证安装完成

在cmd中输入：
```bash
$ jekyll -v
```

输出版本说明安装完成（我的版本为3.7.3）：

jekyll 3.7.3

### 6. 开启本地实时预览

切换到仓库所在目录，在cmd中输入:
```bash
$ jekyll serve
```

## 遇到问题及解决
### 1.gem install jekyll时报错，而且还是乱码！
```bash
C:\User>gem install jekyll
Temporarily enhancing PATH for MSYS/MINGW...
Building native extensions. This could take a while...
ERROR:  Error installing jekyll:
    ERROR: Failed to build gem native extension.

current directory: C:/Ruby25-x64/lib/ruby/gems/2.5.0/gems/http_parser.rb-0.6.0/ext/ruby_http_parser
C:/Ruby25-x64/bin/ruby.exe -r ./siteconf20180308-3672-ueo7ea.rb extconf.rb
creating Makefile

current directory: C:/Ruby25-x64/lib/ruby/gems/2.5.0/gems/http_parser.rb-0.6.0/ext/ruby_http_parser
 make "DESTDIR=" clean
 'make' �����ڲ����ⲿ���Ҳ���ǿ����еĳ���
���������ļ���

current directory: C:/Ruby25-x64/lib/ruby/gems/2.5.0/gems/http_parser.rb-0.6.0/ext/ruby_http_parser
make "DESTDIR="
'make' �����ڲ����ⲿ���Ҳ���ǿ����еĳ���
 ���������ļ���

 make failed, exit code 1

Gem files will remain installed in C:/Ruby24-x64/bin/ruby_builtin_dlls/Ruby24-x6
4/lib/ruby/gems/2.4.0/gems/http_parser.rb-0.6.0 for inspection.
Results logged to C:/Ruby24-x64/bin/ruby_builtin_dlls/Ruby24-x64/lib/ruby/gems/2
.4.0/extensions/x64-mingw32/2.4.0/http_parser.rb-0.6.0/gem_make.out
```
参考oneclick/rubyinstaller2的issue #98。

首先cmd中输入：
```bash
$ chcp 850
```

切换编码之后安装：
```bash
$ gem install jekyll
```

下面是报错：
```bash
Temporarily enhancing PATH for MSYS/MINGW...
Building native extensions. This could take a while...
ERROR:  Error installing jekyll:
        ERROR: Failed to build gem native extension.

    current directory: C:/Ruby25-x64/lib/ruby/gems/2.5.0/gems/http_parser.rb-0.6.0/ext/ruby_http_parser
C:/Ruby25-x64/bin/ruby.exe -r ./siteconf20180308-3672-ueo7ea.rb extconf.rb
creating Makefile

current directory: C:/Ruby25-x64/lib/ruby/gems/2.5.0/gems/http_parser.rb-0.6.0/ext/ruby_http_parser
make "DESTDIR=" clean
'make' 不是内部或外部命令，也不是可运行的程序或批处理文件。

current directory: C:/Ruby25-x64/lib/ruby/gems/2.5.0/gems/http_parser.rb-0.6.0/ext/ruby_http_parser
make "DESTDIR="
'make' 不是内部或外部命令，也不是可运行的程序或批处理文件。

make failed, exit code 1

Gem files will remain installed in C:/Ruby25-x64/lib/ruby/gems/2.5.0/gems/http_p
arser.rb-0.6.0 for inspection.
Results logged to C:/Ruby25-x64/lib/ruby/gems/2.5.0/extensions/x64-mingw32/2.5.0
/http_parser.rb-0.6.0/gem_make.out
```
原来是没有make指令，上面的步骤其实已经安装了msys2，所以不会出现问题。对于没有勾选的童鞋，可以在cmd中输入下面命令来安装：
```bash
$ ridk install
```
安装完成之后再次安装jekyll和jekyll-paginate就ok了。
### 2. jekyll serve启动报错
```bash
Incremental build: disabled. Enable with --incremental
      Generating...
jekyll 3.7.3 | Error:  Permission denied @ rb_sysopen - C:/Users/username/NTUSER.DAT
```
这是因为jekyll默认使用4000端口，而4000是FoxitProtect（福昕阅读器的一个服务）的默认端口。网上有教程说kill掉FoxitProtect的进程，但是我觉得首先这个比较麻烦，其次重启计算机时FoxitProtect是默认启动的，除非关闭这个服务，这样又可能带来其他问题。所以最简单的办法还是指定端口：
```bash
$ jekyll serve -P 5555
```
## 参考链接

[RubyInstaller2 issue98](https://github.com/oneclick/rubyinstaller2/issues/98 "RubyInstaller2 issue98")  

[Install Ruby and the Ruby DevKit](http://jekyll-windows.juthilo.com/1-ruby-and-devkit/ "Install Ruby and the Ruby DevKit")  
