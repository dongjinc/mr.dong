---
title: 你不知道的nginx配置(实战)
layout: post
excerpt: nginx 无所不能 。
comments: true
---

## 1. nginx 基本配置
```nginx
server {
    listen 80; # 端口
    listen [::]:80;
    server_name yo.vooverk.top; # 域名
    root /usr/share/nginx/public; # 根路径
}
```

## 2. nginx 开启 gzip 压缩
```nginx
# xxx.conf
    gzip            on; # on 开启gzip压缩
    gzip_min_length 1000;
    gzip_proxied    expired no-cache no-store private auth;
    gzip_types      brotli_types text/plain text/css application/javascript application/json image/svg+xml application/xml+rss;
```

### 2.1 gzip 与 brotli 同为压缩两者有什么区别 ?? 
- brotli 仅支持 https、gzip 支持 http 和 https;
- brotli 需要更多的计算能力, 代表着设备和软件设施的成本上涨;
- brotli 压缩比定义在 4 - 5级时, 压缩效果会比gzip更好;

### 2.2 为什么 brotli 不支持 https ?
- 因为部分中间代理(CDN)等, 对非 gzip, deflate 内容编码时往往表现得很差, 用 HTTPS，他们可以在大多数情况下避免此问题
- 对于服务器响应编码内容, 是由请求头 Accept-Encoding: 具体编码类型决定的(<strong>ex: gzip, deflate, br</strong>), 其中 http 并未携带 Accept-Endcoding: br 编码类型, 这也导致了 服务器不会对具体内容编码成br形式; (以谷歌浏览器作为参考)

### 2.3 那 deflate 又是什么 ?
- https://luyuhuang.tech/2020/04/28/gzip-and-deflate.html#%E5%BC%95%E8%A8%80

## 3. nginx 开启 pagespeed 效果

### 3.1 简介
- pagespeed 是由 Google 开源用来自动优化网站的神器, 作为 nginx 第三方模块 ngx_pagespeed 将会重写你的网站. 对图片文件进行转换(png to webp等) 、组合 js 以及 css 等文件, 以达到最大限度减少http请求次数、压缩静态资源等优化方式;
### 3.2 pagespeed 镜像源
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
### 3.3 nginx.conf 配置
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
### 3.4 ⚠️⚠️⚠️⚠️⚠️ _ 通过 yum 安装、centos内置的nginx 没有 ./configure 模块, 需要单独下载一个 nginx 包 ⚠️⚠️⚠️⚠️⚠️
- 配置路径: /usr/local/nginx/conf/conf.d
- @TODO 需要调研下 如何更改 centos 内置的nginx
- 参考资料: https://cloud.tencent.com/developer/article/1068607
  

### 3.5 遇到的问题: 
- checking for psol ... not found ./configure: error: module ngx_pagespeed requires the pagespeed optimization library. Look in /root/oneinstack/src/nginx-1.20.0/objs/autoconf.err for more details.(升级 gcc 即可)
    ```bash 
    yum -y install libuuid-devel
    yum -y install centos-release-scl
    yum -y install devtoolset-7-gcc devtoolset-7-gcc-c++ devtoolset-7-binutils
    scl enable devtoolset-7 bash
    gcc -v
    ```

## 4. 网站性能测试工具
- [PageSpeed Insights](https://pagespeed.web.dev/analysis/http-yo-vooverk-top/x8ca37e2op?form_factor=desktop) 由 Google 开发的一款免费评估网站的性能并提供改进建议. 工具通过分析网站的页面加载速度, 给予相应评分、并提供详细的性能优化建议. 

## 5. xxx 持续更新中 ...