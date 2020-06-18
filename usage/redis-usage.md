
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

通配符获取key列表
```
keys *
```

## 数据类型  
### 字符串字符串
```
set key1 "value1"
get key1
del key1
set key1 "1"
incr key1
decr key1
incrby key1 2
decrby key1 2
```

### 链表list  
```
lpush key1 "1"
lpush key1 "2"
rpush key1 "3"
rpush key1 "4"
lrance key1 0 1
lrance key1 0 -1
```

### 集合set  
```
sadd key1 "1"
sadd key1 "2"
smembers key1
sismember key1 "3"
sadd key2 "1"
sadd key2 "2"
sunin key1 key2
```

### 有序集合sorted set  
```
zadd key1 "1"
zadd key1 "3"
zadd key1 "2"

```

## geohash

可用于基于地理位置的就近搜索等功能，相关命令如下 

### GEOADD
添加地理位置信息到key中，可同时添加多个位置。 类似集合的功能。
```
GEOADD key longitude latitude member [longitude latitude member ...]
```

### GEODIST
返回指定两个集合成员的地理位置距离，如果有成员不存在则返回空。
```
GEODIST key member1 member2 [unit]
```
参数'unit'指定返回距离的单位，支持如下单位：
*   **m**   米
*   **km**  千米
*   **mi**  英里
*   **ft**  英尺

### GEOPOS
以数组形式返回指定集合成员的地理位置信息，第一个数组元素为经度，第二个为纬度。成员不存在其数组元素为空。
```
GEOPOS key member [member ...]
```

### GEOHASH
以数组形式返回指定集合成员的 geohash 信息（字符串）。
```
GEOHASH key member [member ...]
```

### GEORADIUS
返回指定地理位置（经纬度）附近指定半径内的所有集合成员
```
GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count]
```
选项说明如下
*   **radius**          半径
*   **m|km|ft\mi**      半径单位
*   **WITHCOORD**       返回成员到中心的距离，距离单位与半径相同
*   **WITHDIST**        返回成员的经纬度信息
*   **WITHHASH**        返回成员的geohansh信息
*   **COUNT**           指定返回的成员数量，默认全部返回
*   **ASC**             成员按距离中心从小到大排序
*   **DESC**            成员按距离中心从大到小排序

返回多个信息时按如下方式显示
1.  距离
2.  geohash
3.  经度和纬度。

### GEORADIUSBYMEMBER
与命令'GEORADIUS'类似，返回指定成员附近指定半径内的所有其它集合成员
```
GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count]
```

### ZREM
删除指定集合成员。geo没有提供专门的删除命令，使用zset命令
```
ZREM key member
```


