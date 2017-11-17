# configureNginxComboWithHTTPRequest

## 7. 优化案例：　Nginx 配置 Combo 合并 HTTP 请求
　　Nginx 是一个开源高效的 HTTP 服务器，通过简单的配置能够提供丰富的功，最重要的是，他对服务器的配置要求极低。Nginx 选用了有事驱动的异步非阻塞模型，而不是传统的依赖多线程来响应，因此，在同等资源的情况下能够提供极大的并发请求。对于前端开发者来说，选用 Nginx 是一个主流并且高性价比的选择。
### 7.1 安装 Nginx 和文件合并模块
　　要玩转 Nginx，首先推荐使用 Linux 操作系统的服务器来搭建服务，一台普通的云服务器即可。文件合并模快选择开源插件 nginx-http-concat, 操作系统为 CentOS 7。一般情况下安装 Nginx 推荐添加 EPEL (全称 Extra Package for Enterprise Linux, 即企业级 Linux 附加包)安装源，然后通过 CentOS 包管理器 yum 安装。这种方式的安装简单且方便升级。因为安装模块需要重行编译 Nginx 安装包，所以这里介绍通过二进制包来安装，步骤如下。
1. 下最新的 Nginx 稳定版本二进制包，以版本 1.10.3　为例, 命令如下：
```
wget http://nginx.org/download/nginx-1.10.3.tar.g
```

2. 解压缩安装包，命令如下：
```
tar zxvf nginx-1.10.3.tar.gz
```

3. 克隆创酷镜像，命令如下：
```
git clone git@github.com:alibaba/nginx-http-concat.git
```

4. 切换到 nginx 目录，命令如下：
```
cd nginx-1.10.3
```

5. 在构建参数中添加模块并构建，代码如下：
```
./configure --add-module=/path/to/nginx-http-concat
make && make install
```

6. 查找 Nginx 地址，命令如下：
```
// 找到 Nginx 执行文件的地址，可以配置命令行环境，或者通过路径调用
whereis nginx
// 查找到 Nginx 的默认配置文件地址，如果出现异常，默认在/etc/nginx/conf.d/default.conf
nginx -t
```

通过 cat 命令读取配置文件，可以看到 root 地址，对应网站根目录。

7. 启动 Nginx, 命令如下：
```
// 推荐使用systemctl来启动，也可以通过 nginx 命令来启动
sudo systemctl start nginx
//　开机自启动，可选
sudo systemctl enable nginx
```

## 7.2 配置 Nginx 和 Combo
　　nginx-http-concat Nginx 适用于 Nginx 的文件合并模块，可以将多个对静态资源的 HTTP 请求合并成为一个，进而减少 HTTP 请求数。假设原来需要 a.js, b.js, c.js 三个文件，产生三次请求，通过该模块可以在一个请求中完成，实例地址如下:
```
//使用??作标识表示合并文件
http://example.com/static/??a.js,b.js,c.js
```

需要修改 Nginx 的配置，文件配置如下：
```
location /static/ {
    # nginx-http-concat 主开关
    concat on
    # 最大合并文件数
    # concat_max_files 10；
    # 只允许通内省文件合并
    # concat_unique on
    # 允许合并的文件类型，多个以逗号分隔。如: application/x-javascript, text/css
    # concat_types text/html
}
```
