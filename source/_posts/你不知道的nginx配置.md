---
title: 你不知道的nginx配置(实战)
layout: post
excerpt: nginx 无所不能 。
comments: true
---

### 1.nginx 基本配置
```nginx
server {
    listen 80; # 端口
    listen [::]:80;
    server_name yo.vooverk.top; # 域名
    root /usr/share/nginx/public; # 根路径
}
```

### 2.nginx 静态资源配置

### nginx 开启 pagespeed 效果
##### 找到 pagespeed 镜像源 https://www.modpagespeed.com/doc/build_ngx_pagespeed_from_source
```bash
    # 下载 pagespeed
    wget -O- https://github.com/apache/incubator-pagespeed-ngx/archive/v1.13.35.2-stable.tar.gz | tar -xz
    cd incubator-pagespeed-ngx-1.13.35.2-stable
    # 下载 psol 优化库
    wget -O- https://dl.google.com/dl/page-speed/psol/1.13.35.2-x64.tar.gz | tar -xz
    # 下载对应的 nginx 版本包
    wget -O- http://nginx.org/download/nginx-1.20.1.tar.gz | tar -xz
    # 新增 模块
    ./configure  --add-module=../incubator-pagespeed-ngx-1.13.35.2-stable 
    # 编译 即可
    make && make upgrade 
```
##### nginx.conf 配置
```nginx
    pagespeed on;
    pagespeed FileCachePath /tmp/cache/ngx_pagespeed_cache;
    # 禁用CoreFilters    
    pagespeed RewriteLevel PassThrough;
    # 启用压缩空白过滤器    
    pagespeed EnableFilters collapse_whitespace;
    # 启用JavaScript库卸载    
    pagespeed EnableFilters canonicalize_javascript_libraries; #谷歌被墙，并不确定这
    个设置有没有副作用 
    # 把多个CSS文件合并成一个CSS文件    
    pagespeed EnableFilters combine_css;
    # 把多个JavaScript文件合并成一个JavaScript文件    
    pagespeed EnableFilters combine_javascript;
    # 删除带默认属性的标签    
    pagespeed EnableFilters elide_attributes;
    # 改善资源的可缓存性    
    pagespeed EnableFilters extend_cache;
    # 更换被导入文件的@import，精简CSS文件    
    pagespeed EnableFilters flatten_css_imports;
    pagespeed CssFlattenMaxBytes 5120;
    # 延时加载客户端看不见的图片    
    pagespeed EnableFilters lazyload_images;
    # 启用JavaScript缩小机制    
    pagespeed EnableFilters rewrite_javascript;
    # 启用图片优化机制    
    pagespeed EnableFilters rewrite_images;
    # 预解析DNS查询    
    pagespeed EnableFilters insert_dns_prefetch;
    # 重写CSS，首先加载渲染页面的CSS规则    
    pagespeed EnableFilters prioritize_critical_css;
    server {
        listen       80;
        server_name  yo.vooverk.top;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   /usr/share/nginx/public;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```
##### ⚠️⚠️⚠️⚠️⚠️ _ 通过 yum 安装、centos内置的nginx 没有 ./configure 模块, 需要单独下载一个 nginx 包
- 配置路径: /usr/local/nginx/conf/conf.d
- @TODO 需要调研下 如何更改 centos 内置的nginx
- 参考资料: https://cloud.tencent.com/developer/article/1068607
  

##### 遇到的问题: 
- ### checking for psol ... not found ./configure: error: module ngx_pagespeed requires the pagespeed optimization library. Look in /root/oneinstack/src/nginx-1.20.0/objs/autoconf.err for more details.(升级 gcc 即可)
    ```bash 
    yum -y install libuuid-devel
    yum -y install centos-release-scl
    yum -y install devtoolset-7-gcc devtoolset-7-gcc-c++ devtoolset-7-binutils
    scl enable devtoolset-7 bash
    gcc -v
    ```

### demo3