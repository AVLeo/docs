
## 安装 配置
ubuntu 为例
```
sudo apt install redis-server
```
若默认开启ipv6, 在 /etc/redis/redis.conf 中
```
#bind 127.0.0.1 ::1
bind 127.0.0.1
```
启动/连接
```
systemctl start redis-server
redis-cli -h 127.0.0.1 -p 6379
```

## 基础操作  
key-value
```
set key1 value1
get key1
del key1
```
获取key列表
```
keys *
keys key*
```

