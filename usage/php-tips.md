
## 写日志文件
注意日志文件权限问题
```
file_put_contents("/tmp/wp.log", var_export($_SERVER, true), FILE_APPEND);
```
