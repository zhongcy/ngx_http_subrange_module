ngx_http_subrange_module
========================


当Nginx作为文件下载服务的反向代理,用户请求一个非常大的文件的时候,它会一直占满反向代理服务器与后端主机之间的带宽。因为nginx一次获取整个文件,缓冲获取到的文件,导致客户端不能马上读取到。带宽使用和iowait会很高。

ngx_http_subrange_module就是为了解决这个问题，它能分割HTTP requests。将大数据量的HTTP请求切分为多个子请求，当下载一个1 G的文件,subrange将从后端主机中下载文件块，比如先获取5 M，然后再获取5 M,直到客户端下载完整个文件。


[![Build Status](https://travis-ci.org/shafreeck/ngx_http_subrange_module.svg?branch=master)](https://travis-ci.org/shafreeck/ngx_http_subrange_module)

Split one big download file request to multiple subrange requests to avoid geting too
much data from upstream at one time.
Install:
--------
Configure nginx with `--add-module=/path/to/ngx_http_subrange_module`

Directive:
---------
```
syntax : subrange size
default: subrange 0, 0 means disable
context: http, server, location
```

Example:
---------
```
  location /{  
    root html;  
    subrange 10k;  
  }
```
Introduction:
-------------
When nginx is used as a reverse proxy for file downloading service, it will
always run out of the bandwidth between nginx and upstream when the user requests
a very large file. This is because nginx fetch a whole file at a time and buffer
the left data that the client can not read in time. The bandwidth would be used up
and the disk iowait would be high.

Nginx has an option to turn off the buffer mechanism, for example set `proxy_buffering off`
or `fastcgi_buffering off`, which directive depends on the type of your upstream.
However, this brings it with a problem. If your upstream is PHP or Java, it will
block your PHP fastcgi process or yourJava threads, especially when the client is
downloading a very large file and even worse he has a terrible speed.

The subrange module is created to solve this problem. It splits your HTTP requests.
When you want to download an 1G file, the module will try to download a chunk of the
file from the upstream, for example downloads 5M first, and then the next 5M, until
the client receives the whole file. The whole process is non-sensible to client.
You can set the chunk size in the nginx configuration file.

The module sets the HTTP [Range](http://tools.ietf.org/html/rfc2616#section-14.35) header
 to perform a Range request to get a chunk
from the upstream. So the supporting of Range request is needed by upstream. Supporting
`Range` is easy, all standard HTTP servers like nginx/apache have implemented it.
It is trivial to implement it yourself.

We just have one directive `subrange` which sets the size of chunk being fetched at a
time. The directive takes a `size` as the parameter(1024 or 1k), or a variable as
well. 0 meas disable subrange.

简介：
当 Nginx 用作文件下载服务的反向代理时，当用户请求非常大的文件时，它将始终耗尽 Nginx 和上游之间的带宽。 这是因为 Nginx 一次获取整个文件并缓冲客户端不能时间读取的左数据。 带宽会耗尽，磁盘iowait也会很高。

Nginx 具有关闭缓冲区机制的选项，例如设置 proxy_buffering off 或者 fastcgi_buffering off，这个指令取决于上游的类型。 然而，这带来了一个问题。 如果你的上游是PHP或者 Java，它将阻止你的has进程或者yourJava线程，特别是当客户端下载一个很大的文件。

子区域模块是为了解决这个问题而创建的。 它分割你的HTTP请求。 下载 1G 文件时，模块将从上游下载一个文件块，例如下载 5M，然后再下载下一个文件，直到客户端收到整个文件。 整个过程对客户是不敏感的。 你可以在 Nginx 配置文件中设置区块大小。

模块设置 HTTP 范围 header 执行一个范围请求，从上游获取一个区块。 因此上游需求需要范围请求的支持。 支持 Range 很简单，像 Nginx/apache这样的所有标准HTTP服务器都实现了。 你自己实现它很简单。

我们只有一个指令 subrange，它设置每次提取块的大小。 指令接受 size 作为参数( 1024或者 1 k )，也接受变量。 0 meas禁用子范围。


```
set $size 10m;
location /download{
    subrange $size;
}
```
