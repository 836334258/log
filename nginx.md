- 匹配规则

  - 已`=`开头表示精确匹配
  - `^~`开头表示uri已某个常规字符串开头，不是正则匹配
  - `~`开头表示区分大小写的正则匹配
  - `~`*开头表示不区分大小写的正则匹配
  - `/`通用匹配，如果没有其他匹配，任何请求都会匹配到

- gzip配置

  ```nginx
  
  gzip on;        　　　　　　　　　　　　　　　　　　     #表示开启压缩功能
  
  gzip_min_length  1k; 　　　　　　　　　　　　　   #表示允许压缩的页面最小字节数，页面字节数从header头的Content-Length中获取。默认值是0，表示不管页面多大都进行压缩，建议设置成大于1K。如果小于1K可能会越压越大
  
  gzip_buffers     432k;　　 　　　　　　　　　　 　 　  #压缩缓存区大小  有问题
  
  gzip_http_version 1.1; 　　　　　　　　　　　　　　   #压缩版本
  
  gzip_comp_level 9;　　　　　　　　　　　　　　  　   #压缩比率  有问题
  
  gzip_types  text/css text/xml application/javascript;　　#指定压缩的类型
  
  gzip_vary on;　　　　　　　　　　　　　　　　　    　 #vary header支持
  ```

  