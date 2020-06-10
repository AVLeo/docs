# Openresty Lua GeoIP2 使用

## 安装openersty  
参考安装文档：https://openresty.org/en/installation.html

## GeoIP2 安装配置
1. 从 https://dev.maxmind.com/geoip/geoip2/geolite2/ 下载MaxMind格式的GeoIP2数据库

2. geoip lua库安装  
github地址：https://github.com/anjia0532/lua-resty-maxminddb 可在openresty安装目录使用如下命令安装
```
bin/opm get anjia0532/lua-resty-maxminddb
```

3. geoip lua依赖库安装  
github地址：https://github.com/maxmind/libmaxminddb/ 获取源码包，通过如下命令安装
```
./configure
make
make check
sudo make install
sudo ldconfig
```

4. lua测试脚本
```
local cjson = require("cjson")
local geo = require("resty.maxminddb")
local arg_ip = ngx.var.arg_ip
local arg_node = ngx.var.arg_node

if not geo.initted() then
        geo.init("/opt/geoip/GeoLite2-City_20200609/GeoLite2-City.mmdb")
end


local res,err=geo.lookup(arg_ip or ngx.var.remote_addr)

ngx.header["Content-Type"]="application/json";

if not res then
        ngx.say("Please check the ip address you provided: <div style='color:red'>",arg_ip,"</div>")
        ngx.log(ngx.ERR,' failed to lookup by ip , reason :',err)
else
        ngx.say(cjson.encode(res))

        if arg_node then
                ngx.say("node name:",ngx.var.arg_node, " , value:",cjson.encode(res[ngx.var.arg_node] or {}))
        end

end
```

5. nginx 配置  
location段添加配置 content_by_lua_file [path to lua script];
