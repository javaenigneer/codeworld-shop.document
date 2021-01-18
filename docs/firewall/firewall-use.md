## 防火墙的使用
转载：[防火墙的使用](https://how2j.cn/k/vmware/vmware-firewal/2002.html)
### 防火墙的概念
CentOS 上通过 firewall 来进行防火墙的工作。
防火墙的工作简单说，就是运行那些端口开放给访问者用，哪些不开放~
### 安装防火墙
`yum -y install firewalld firewall-config`
### 防火墙生命周期管理
* 为了启动防火墙，要先重启下 dbus
`systemctl restart dbus`
然后通过如下命令进行防火墙生命周期管理
* 启动一个服务
`systemctl start firewalld.service`
* 关闭一个服务
`systemctl stop firewalld.service`
* 重启一个服务
`systemctl restart firewalld.service`
* 显示一个服务的状态
`systemctl status firewalld.service`
* 在开机时启用一个服务
`systemctl enable firewalld.service`
* 在开机时禁用一个服务
`systemctl disable firewalld.service`
* 查看服务是否开机启动
`systemctl is-enabled firewalld.service`
* 查看已启动的服务列表
`systemctl list-unit-files|grep enabled`
* 查看启动失败的服务列表
`systemctl --failed`
### 防火墙配置工作
* 查看版本
`firewall-cmd --version`
* 查看帮助
`firewall-cmd --help`
* 显示状态
`firewall-cmd --state`
* 查看所有打开的端口
`firewall-cmd --zone=public --list-ports`
* 更新防火墙规则
`firewall-cmd --reload`
* 查看区域信息
`firewall-cmd --get-active-zones`
* 查看指定接口所属区域
`firewall-cmd --get-zone-of-interface=eth0`
* 拒绝所有包，测试别用这个。不如然只有到VMWare 的终端上去关闭防火墙 SSH 客户端，稍显麻烦
`firewall-cmd --panic-on`
* 取消拒绝状态
`firewall-cmd --panic-off`
* 查看是否拒绝
`firewall-cmd --query-panic`
### 那怎么开启一个端口呢
添加
注1：--permanent永久生效，没有此参数重启后失效
注2：增加了要用 firewall-cmd --reload，才会生效
`firewall-cmd --zone=public --add-port=80/tcp --permanent`
* 重新载入
`firewall-cmd --reload`
* 查看
`firewall-cmd --zone=public --query-port=80/tcp`
* 删除
`firewall-cmd --zone= public --remove-port=80/tcp --permanent`
