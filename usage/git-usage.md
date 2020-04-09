## install gogs
```
# Pull image from Docker Hub.
$ docker pull gogs/gogs

# Create local directory for volume.
$ mkdir -p /var/gogs

# Use `docker run` for the first time.
$ docker run --name=gogs -p 10022:22 -p 10080:3000 -v /var/gogs:/data gogs/gogs

# Use `docker start` if you have stopped it.
$ docker start gogs
```

## ssh key
```
ssh-keygen -t rsa -b 4096 -C "zhengxj@yoostargroup.com"
```

## git 配置
```
git config --global user.email "leo@medialab"
git config --global user.name "leo"
git config --global core.editor "vim"
```

## 代码打包
```
git archive -o v1.1.0.zip HEAD
```

## 删除分支
删除本地分支
```
git branch -d [brnach name]
```
删除远程分支
```
git push origin :[brnach name]
```