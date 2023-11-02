
# Debian系统下搭建VerneMQ
## 依赖包的安装

```shell
apt install build-essential erlang libsnappy-dev 
```

## 构建
如果可以开代理，尽量打开代理，可能构建过程中需要访问github资源。

```shell
mkdir /tmp/vernemq_build
cd  /tmp/vernemq_build
git clone https://github.com/vernemq/vernemq.git
cd vernemq
make rel
```

## 安装

约定：请在使用的终端中临时设置一个变量：VERNEMQ_INSTALL_PATH  
作用：设置vernemq的安装路径，可以根据实际情况调整，这里我们安装到：/usr/local/vernemq

```shell
VERNEMQ_INSTALL_PATH=/usr/local/vernemq
```


编译结果是在: _build/default/rel/vernemq 中，可以复制到指定的目录中，如：  

```shell
sudo cp -r _build/default/rel/vernemq $VERNEMQ_INSTALL_PATH
```


# Debian12自动构建安装脚本

```shell
# 安装依赖
apt install build-essential erlang libsnappy-dev 
# 构建软件
mkdir /tmp/vernemq_build
cd  /tmp/vernemq_build
git clone https://github.com/vernemq/vernemq.git
cd vernemq
make rel
#设置安装目录
VERNEMQ_INSTALL_PATH=/usr/local/vernemq
# 安装软件
sudo cp -r _build/default/rel/vernemq $VERNEMQ_INSTALL_PATH
```
## 添加环境变量
目的: 可以任意位置执行vernemq相关命令  
实现方案: 在/etc/profile中添加vernemq的bin目录路径
```shell
cp /etc/profile /etc/profile.vmq_back
echo -e "export PATH=$VERNEMQ_INSTALL_PATH/bin:\$PATH\n" >> /etc/profile
source /etc/profile
```
以上代码意思是：先备份/etc/profile文件，然后使用echo追加一个配置，最后让配置立即生效

## 添加账号和秘密
添加账号可以根据需求喜好添加，这里添加一个admin用户，命令执行时会提示设置秘密
```
 vmq-passwd -c $VERNEMQ_INSTALL_PATH/etc/vmq.passwd admin
```

## 使用systemctl管理vernemq
0. 在/usr/lib/systemd/system中创建名为vernemq.service的文件

```shell
nano /usr/lib/systemd/system/vernemq.service
```
1. vernemq.service并添加如下内容
> 注意:不要无脑照搬，/usr/local/vernemq/应改成前面约定的VERNEMQ_INSTALL_PATH变量所存储的路径值

```shell
[Unit]
Description=VerneMQ - A distributed MQTT message broker
Wants=network.target
After=network.target

[Service] 
ExecStart=/usr/local/vernemq/bin/vernemq start 
ExecStop=/usr/local/vernemq/bin/vernemq stop 
ExecReload=/usr/local/vernemq/bin/vernemq restart 
Restart=on-failure
User=root 
Group=root 
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

2. 重新加载配置

```shell
sudo systemctl daemon-reload
```

3. 把vernemq服务设置到开机自启

```shell
systemctl enable vernemq.service
```

## 启动vernemq

```shell
systemctl start vernemq.service
```

## 停止vernemq

```shell
systemctl start vernemq.service
```

## 查看vernemq状态

```shell
systemctl status vernemq.service
```

## 页面查看集群状态
访问如下地址
```shell
http://127.0.0.1:8888/status
```


