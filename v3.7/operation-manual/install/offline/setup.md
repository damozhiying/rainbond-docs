---
title: 离线安装 
summary: 离线安装云帮
toc: true 
---

离线安装对系统环境要求比较严格，准备工作建议参考执行安装前准备和离线准备工作。

## 一、制作离线包

需要在有网的环境下，且已经安装了docker，且有权限读写/opt/rainbond目录。

```bash
# 默认会在/opt/目录下创建rainbond相关目录文件,建议当前系统环境与后续离线系统环境保持一致
curl https://raw.githubusercontent.com/goodrain/rainbond-install/v3.7.2/scripts/offline.sh -o ./offline.sh
chmod +x offline.sh
./offline.sh
```

## 二、准备工作

1. 将离线包同步到离线环境 `tar xf install.offline.3.7.2.2018-11-10.tgz -C /`。**请务必添加 -C 参数指定解压路径**
2. 请提前安装好如下包,避免安装失败。

```bash
# 通用软件工具包
tar ntpdate wget curl tree lsof htop nload net-tools telnet rsync git dstat iotop lvm2 pwgen
# centos
nfs-utils perl bind-utils iproute bash-completion createrepo
# debian/ubuntu
nfs-kernel-server nfs-common dnsutils python-pip python-apt apt-transport-https uuid-runtime iproute2 systemd
```

## 三、安装Rainbond

- 安装首个管理节点

```bash
cd /opt/rainbond/install
./grctl init --install-type offline --domain <自定义域名，可选>
```

- 扩容管理节点

```bash
# 在管理节点manage01执行如下操作
# ssh安装
grctl node add --hostname <主机名> --iip <内网ip> --private-key <信任私钥(/root/.ssh/id_rsa)> --role master
# 密码安装(仅支持root用户)
grctl node add --hostname <主机名> --iip <内网ip> --root-pass <root用户密码> --role master
# 更新部分配置
salt "*" state.sls common.node_conf
# 如管理节点的 entrance 服务
systemctl restart node
systemctl restart rbd-entrance
# 如果扩容前已经有应用运行了，请将管理节点1的rbd-lb的数据同步到其他管理节点
数据路径：/opt/rainbond/etc/rbd-lb/
```

- 扩容计算节点

```bash
# 在管理节点manage01执行如下操作
# ssh安装
grctl node add --hostname <主机名> --iip <内网ip> --private-key <信任私钥(/root/.ssh/id_rsa)> --role worker
# 密码安装(仅支持root用户)
grctl node add --hostname <主机名> --iip <内网ip> --root-pass <root用户密码> --role worker
```

## 四、安装提示报错

- 如果是安装包失败或者提示什么命令未找到，请停止安装,根据提示自行下载相关安装包

```bash
mkdir -p /root/pkgs
# centos
yum install <包名> --downloadonly --downloaddir=/root/pkgs
# debian,deb默认放在/var/cache/apt/archives/目录下
apt install <包名> -d -y
```

手动安装缺失的包或工具,然后重新执行安装。

- 安装docker失败，提示audit包问题

请检查`/opt/rainbond/install/install/pkgs/centos`目录下

```bash
ls | grep au
audit-2.8.1-3.el7.x86_64.rpm
audit-libs-2.8.1-3.el7.x86_64.rpm
audit-libs-python-2.8.1-3.el7.x86_64.rpm
```

正常情况应该是这样的，需要移除其他包或者新增缺失包，相关包(centos el7)可以在[阿里云镜像站](https://opsx.alibaba.com/mirror)下载

## 五、安装问题建议

当离线安装Rainbond 遇到问题时，请参考本篇相关文档。如果问题未解决，请按文档要求收集相关信息通过 Github [提供给 Rainbond开发者](https://github.com/goodrain/rainbond/issues/new)。

1. 系统版本
2. 机器配置
3. 云帮版本
4. 具体什么报错(最好有必要的图文说明)
5. 是否重新执行安装
6. 是否采用了上述的解决方案

## 六、附录

可以使用好雨科技提供的离线安装包,此离线包仅针对阿里云CentOS 7.4做过优化，并不能保证其适用于其他环境下的CentOS服务器。此离线包仅会同步更新大版本。

```bash
wget https://pkg.rainbond.com/releases/offline/v3.7.2/install.offline.3.7.2.2018-11-10.tgz

MD5SUM:
07e9e1594cc206648746e04b3a4f2d6d
SHA512SUM:
65895bddd62c09a7196d6a90074f1fc733ded006e5cfa5b44d753e11867b145f11fdd68d3e9e9a1a2c34d0bf61e563e8e7c9da172eaf32655aec50810278c649

# 特别说明,期间可能修复了某些bug，需要你在有网的环境下进行如下操作：
wget https://pkg.rainbond.com/releases/offline/v3.7.2/install.offline.3.7.2.2018-11-10.tgz
tar xf  https://pkg.rainbond.com/releases/offline/v3.7.2/install.offline.3.7.2.2018-11-10.tgz -C /
cd  /opt/rainbond/install/
git pull
wget https://pkg.rainbond.com/releases/common/v3.7.2/grctl
chmod +x grctl
# 更新离线镜像 /opt/rainbond/install/install/imgs/ 如：rbd-api,rbd-worker,rbd-chaos,rbd-app-ui,rbd-cni等组件
cd /opt/rainbond/install/scripts
./offline.sh image #(默认更新全部镜像)
tar zcvf install.offline.3.7.2.$(date +%F).tgz /opt/rainbond/install
# 离线环境
tar xf install.offline.3.7.2.xxxx.tgz -C / #(解压上述离线包)
cd /opt/rainbond/install/
./grctl init --install-type offline --domain <自定义域名，可选>
```

## 七、已知问题
