## 4.安装etcd集群
### 4.1 准备etcd证书
etcd是k8s非常重要的组件，因此这里采用etcd集群，三个节点都安装etcd，etcd开选举模式。
etcd节点需要提供给其他服务访问，就要验证其他服务的身份，所以需要一个标识自己监听服务的server证书，当有多个etcd节点的时候也需要client证书与etcd集群其他节点交互，当然也可以client和server使用同一个证书因为它们本质上没有区别。
```
#下载etcd证书
$ wget https://github.com/coreos/etcd/releases/download/v3.2.14/etcd-v3.2.14-linux-amd64.tar.gz
$ tar zxvf etcd-v3.2.14-linux-amd64.tar.gz
$ cd etcd-v3.2.14-linux-amd64
$ mv etcd  etcdctl /usr/bin/
#etcd证书放在这
$ mkdir -p /etc/kubernetes/ca/etcd
$ cd /etc/kubernetes/ca/etcd/
#准备etcd证书配置，下载etcd证书到该目录
wget https://github.com/wujihua/kubernetes/blob/master/config/etcd-csr.json
#使用根证书(ca.pem)签发etcd证书
$ cfssl gencert \
        -ca=/etc/kubernetes/ca/ca.pem \
        -ca-key=/etc/kubernetes/ca/ca-key.pem \
        -config=/etc/kubernetes/ca/ca-config.json \
        -profile=kubernetes etcd-csr.json | cfssljson -bare etcd
#跟之前类似生成三个文件etcd.csr是个中间证书请求文件，我们最终要的是etcd-key.pem和etcd.pem
$ ls
etcd.csr  etcd-csr.json  etcd-key.pem  etcd.pem
#etcd证书分发到其他连个master节点上
$ scp -p etcd*.pem root@server02:/etc/kubernetes/ca/etcd    # etcd2
$ scp -p etcd*.pem root@server03:/etc/kubernetes/ca/etcd    # etcd3
```
### 4.2 etcd服务配置
下载etcd.service配置，有几个ip需要替换当前节点的ip地址，name需要替换相应的name：etd1、etcd2、etcd几。
```
wget https://github.com/wujihua/kubernetes/blob/master/config/etcd.service
cp etcd.service /lib/systemd/system/
systemctl daemon-reload
systemctl enable etcd.service
systemctl start etcd.service
```
验证etcd服务
```
#查看集群状态：
$ ETCDCTL_API=3 etcdctl \
  --endpoints=https://192.168.0.101:2379,https://192.168.0.102:2379,https://192.168.0.103:2379  \
  --cacert=/etc/kubernetes/ca/ca.pem \
  --cert=/etc/kubernetes/ca/etcd/etcd.pem \
  --key=/etc/kubernetes/ca/etcd/etcd-key.pem \
  endpoint health
#/查看集群成员：
可以看到集群节点的选举情况
$ ETCDCTL_API=3 etcdctl \
  --endpoints=https://192.168.0.101:2379,https://192.168.0.102:2379,https://192.168.0.103:2379  \
  --cacert=/etc/kubernetes/ca/ca.pem \
  --cert=/etc/kubernetes/ca/etcd/etcd.pem \
  --key=/etc/kubernetes/ca/etcd/etcd-key.pem \
  member list
```

