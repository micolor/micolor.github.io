##   挂载磁盘并设置共享目录

###  新建并挂载磁盘

### 给服务器新增一块硬盘

```
fdisk -l
lsblk
```

![image-20210127205120078](images/centos/image-20210127205120078.png)

### 给共享硬盘分区，全部默认保存 

```
fdisk  /dev/sdb
```

![image-20210127205500695](images/centos/image-20210127205500695.png)

### 格式化新分区

```mkfs.xfs  -f   /dev/sdb```

![image-20210127205956434](images/centos/image-20210127205956434.png)

### 开启默认挂载磁盘

```
vi /etc/fstab
```

![image-20210127210627086](images/centos/image-20210127210627086.png)

### 挂载

```mount -a
mount -a
```

### 查看挂载状态

![](images/centos/image-20210127210825597.png)



## 设置共享目录

### 安装相工具关包

```
yum -y install nfs-utils rpcbind
systemctl enable rpcbind nfs-server
```

## 修改配置文件

```
vi /etc/exports
/mnt/share 192.168.100.21/255.255.255.0(rw,no_root_squash,no_all_squash,sync)
```

## 启动并查看共享目录

```
#启动服务
exportfs -rv
systemctl enable rpcbind nfs-server
systemctl start nfs-server rpcbind
rpcinfo -p
#查看共享目录
showmount -e
```

##  挂载并使用共享目录

### 手动挂载共享目录到本地

```
mount 192.168.100.21:/mnt/share /mnt/share
```

### 查看挂载状态

![image-20210127221258651](images/centos/image-20210127221258651.png)

### 设置自动挂载共享目录

```
vi /etc/fastab
```

![image-20210127221826844](images/centos/image-20210127221826844.png)

### 挂载

```
mount -a
```



