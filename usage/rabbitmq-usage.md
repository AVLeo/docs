## 安装
```
sudo apt-get install erlang-nox
sudo apt-get install rabbitmq-server
```

## 配置
```
# 启动rabbitmq服务
sudo service rabbitmq-server start 
# 关闭rabbitmq服务
sudo service rabbitmq-server stop
# 重启服务
sudo service rabbitmq-server restart
# 查看服务运行状态
sudo service rabbitmqctl status
sudo rabbitmqctl start_app
# 启用web管理
sudo rabbitmq-plugins enable rabbitmq_management
# 新建用户
sudo rabbitmqctl add_user admin admin
# 赋予管理员权限
sudo rabbitmqctl set_user_tags admin administrator
# 赋予读写/远程访问权限
sudo rabbitmqctl  set_permissions -p / admin '.*' '.*' '.*'
# 查看当前所有用户
sudo rabbitmqctl list_users
# 查看默认guest用户的权限
sudo rabbitmqctl list_user_permissions guest
# 删掉默认用户(由于RabbitMQ默认的账号用户名和密码都是guest。为了安全起见, 可以删掉默认用户）
sudo rabbitmqctl delete_user guest
```
