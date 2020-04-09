
## 安装

1. 下载Intel_CS_WebRTC
2. 安装nodejs
```
cd /opt
wget https://nodejs.org/download/release/v8.11.1/node-v8.11.1-linux-x64.tar.gz
sudo ln -s /opt/webrtc/node-v8.11.1-linux-x64/bin/node /usr/local/bin/node
sudo ln -s /opt/webrtc/node-v8.11.1-linux-x64/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm
```
3. 安装依赖
```
./bin/init-all.sh --deps
```
4. 启动
```
更改 management_api/package.json 内node版本号
./bin/start-all.sh
```
5. 访问
```
https://:3300/console
```


superServiceId: 5ddb878383d5b3220ab8ae91
superServiceKey: Obx/pdXRDOemoumXdb1DzEBQFoS2UWuSXcsUshKA9E+mju5ZWSNofay1K8SwwWZI+/4vSOETDVvpmN2XFQC/FjBsrAkLne6K6UZEmPVXEk7c7MNCumy0I2LcRT4EKaYrVNAdFFdYuhwhS4bXOdv53HoILxXs0D/P4zGj6OKNTE0=
sampleServiceId: 5ddb878383d5b3220ab8ae92
sampleServiceKey: /PYkARDKC0pCtfmAFjm3iShepjmUJjJMzpPUP+VRGnBPGOiPMbE+qnMzuI0kY+c2h9dz6D1ZAgKYCLGTZrQoSoA7W2jIkiwOCnB/r5dAciTGj0cWlLK3mpnTW3IsXFiIY2rA4dz+e4X8BVvp1iBS8/SLfIAhMAThIeCGvW+NFf4=
