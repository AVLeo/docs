# build apache traffic server (ats9)


## build from source souce 

### centos 7
```
# packages 
sudo yum install gcc gcc-c++ pkgconfig pcre-devel tcl-devel expat-devel openssl-devel libcap libcap-devel hwloc hwloc-devel ncurses-devel libcurl-devel autoconf automake libtool

# install g++ support c++ 17
sudo yum install centos-release-scl
sudo yum-config-manager --enable rhel-server-rhscl-7-rpms
sudo yum install devtoolset-7
/opt/rh/devtoolset-7/enable

# compile source code
autoreconf -i
./configure && make
```

### ubuntu
```
apt-get install autoconf automake autotools-dev bison debhelper dh-apparmor flex gettext intltool-debian libbison-dev libcap-dev libexpat1-dev libfl-dev libpcre3-dev libpcrecpp0v5 libsigsegv2 libsqlite3-dev libssl-dev libtool m4 po-debconf tcl-dev tcl8.6-dev zlib1g-dev

# on ubuntu 16, not support c++ 17
$ sudo apt-get install software-properties-common python-software-properties
$ sudo add-apt-repository ppa:ubuntu-toolchain-r/test
$ apt-get update
# Optional, but this avoids having apt-get upgrade download other things from the test repository
# Note: xenial for Ubuntu 16.x and 17.x, trusty for Ubuntu 14.x
$ cat > /etc/apt/preferences.d/xenial
Package: *
Pin: release a=xenial
Pin-Priority: 50
$ apt-get -t xenial install gcc-7 g++-7
cd /usr/bin && rm g++-5 && ln -s g++-7 g++

```