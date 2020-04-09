

## 内置变量
```
$args  
    请求的所有参数
$content_length  
    等于请求行的“Content_Length”的值。
$content_type  
    等同与请求头部的”Content_Type”的值
$document_root  
    等同于当前请求的root指令指定的值
$document_uri
$host
    与请求头部中“Host”行指定的值或是request到达的server的名字（没有Host行）一样 上述例子中没有体现
$request_method
    等同于request的method，通常是“GET”或“POST”
$remote_addr
    客户端ip 这个再做负载均衡的时候 如果要获取到客户端的请求ip需要这个
$remote_port
    客户端port
$remote_user
    等同于用户名，由ngx_http_auth_basic_module认证
$request_filename
    当前请求的文件的路径名，由root或alias和URI request组合而成
$request_body_file
    请求body内的文件
$request_uri
    含有参数的完整的初始URI
$query_string
    与$args一样 请求中的参数
$sheeme
    http模式（http,https）
$server_protocol
$server_name
    请求到达的服务器名
$server_port
    请求到达的服务器的端口号
$uri
    等同于当前request中的URI，可不同于初始值，例如内部重定向时或使用index
```