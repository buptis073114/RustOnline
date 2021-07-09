# RustOnline

# 前言
最近要在Ubuntu操作系统使用命令行编译一个工程Rust，此工程依赖了很多国外的源，这就需要使用命令行Online。


# 命令行配置Online


Terminal只支持http、https协议，而爱思爱思使用的是socks协议，爱思爱思R同理。我们可以使用Privoxy来将http和socks相互转换。
首先，使用下面的命令安装Privoxy：

```bash
sudo apt-get install privoxy
```

安装完毕后，打开Privoxy的配置文件/etc/privoxy/config:

```bash
sudo gedit /etc/privoxy/config
```
第一步定位到4.1. listen-address 这一段，找到监听的端口，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210708152032538.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2J1cHQwNzMxMTQ=,size_16,color_FFFFFF,t_70#pic_center)
可以看到端口一般都是8118。
接着找到5.2. forward-socks4, forward-socks4a, forward-socks5 and forward-socks5t这一节，加上如下配置(图中是爱思爱思R的配置):
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210708152119845.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2J1cHQwNzMxMTQ=,size_16,color_FFFFFF,t_70#pic_center)
在这个文本的最后添加如下两行：

```bash
listen-address 127.0.0.1:8118
forward-socks5 / 127.0.0.1:1080
```

如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210708151830532.png#pic_center)
保存后，重启一下Privoxy:
```bash
sudo /etc/init.d/privoxy restart
```
接着配置终端的环境，打开终端配置文件:

```bash
sudo gedit ~/.bashrc
```

在末尾追加下面代码：

```bash
export http_proxy="127.0.0.1:8118"
export https_proxy="127.0.0.1:8118"
```

保存文件后，重启终端或者执行下面的命令重新读取配置文件：

```bash
source ~/.bashrc
```

使用下面代码来测试Online是否成功：

```bash
wget http://www.google.com
```

代理上网成功，接着将Privoxy添加到开机启动，在/etc/rc.local中添加如下命令，注意在exit 0之前:

```bash
sudo /etc/init.d/privoxy start
```

在/etc/profile的末尾添加如下两句:

```bash
export http_proxy="127.0.0.1:8118"
export https_proxy="127.0.0.1:8118"
```

到这里就可以使用Terminal来Online了，注意这部分是根据ss中默认本地端口为1080的情况，如果是ssr，端口是12333，要配置成相应的12333端口。其本质就是ss或ssr进行listen，系统设置Online后，出网的数据要流到127.0.0.1:1080(或12333),ss和ssr listen 后接收数据并通过自己的信道传输。加上privoxy后，就是系统Online发到privoxy，然后privoxy再发到ss ssr上，这中间privoxy可以转发sock5的，也就实现了terminal的Science上网。

# Rust配置Online
编辑Rust的配置文件~/.cargo/config

```bash
sudo vim ~/.cargo/config
```
使用 http Online
```bash
[http]
proxy = "127.0.0.1:8118"
[https]
proxy = "127.0.0.1:8118"
```

如果使用 socks5 Online，需配置：

```bash
[http]
proxy = "socks5://127.0.0.1:1080"
[https]
proxy = "socks5://127.0.0.1:1080"
```

终于可以了
