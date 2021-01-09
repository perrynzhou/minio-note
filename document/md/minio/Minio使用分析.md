## Minio使用分析

| 作者 | 时间 |QQ技术交流群 |
| ------ | ------ |------ |
| perrynzhou@gmail.com |2020/11/01 |中国开源存储技术交流群(672152841) |

#### Minio 部署和使用

- minio采用单个二进制以及多个数据地址能够快速把minio集群搭建起来，对于使用者来说比较便捷
- minio的架构设计比较简洁，每个后端阶段对应接入一定的读写服务，如果对象比较大，能启动数据分片的效果，并行读取这些数据，减少了数据单个磁盘数据争用。
- minio的中的对象存储支持EC，对于三副本来说说成本是节省了一部分
- minio的提供了mc和minio sdk方便用户通过命令行操作对象和接入SDK接入minio服务。


#### Minio对象存储形式

-  4个节点的minio集群

```
root@172.25.78.11 ~ $ ps -ef|grep minio
root      41147  60004  0 08:34 pts/1    00:00:00 grep --color=auto minio
root     157803  60004  1 Nov18 pts/1    00:18:15 ./minio server http://172.25.78.11/data/minio http://172.25.78.12/data/minio http://172.25.78.13/data/minio http://172.25.78.14/data/minio
```
- 测试对象的信息
```

// minio客户端配置
root@172.25.78.11 ~/zhoulin/minio $ ./mc config host add  myminio  http://172.25.78.11:9000  admin123456 admin123456
mc: Configuration written to `/root/.mc/config.json`. Please update your access credentials.
mc: Successfully created `/root/.mc/share`.
mc: Initialized share uploads `/root/.mc/share/uploads.json` file.
mc: Initialized share downloads `/root/.mc/share/downloads.json` file.

// 原始数据的信息
root@172.25.78.11 ~ $ ls -l -a anaconda-ks.cfg 
-rw-------. 1 root root 6813 Nov  4 15:22 anaconda-ks.cfg

//创建test1这个bucket,同时上传测试对象
root@172.25.78.11 ~ $ mc mb myminio/test1
Bucket created successfully `myminio/test1`.
root@172.25.78.11 ~ $ mc cp anaconda-ks.cfg  myminio/test1
anaconda-ks.cfg:               6.65 KiB / 6.65 KiB ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 343.35 KiB/s 0s

//上传到minio集群后测试对象的信息
root@172.25.78.11 ~ $ mc ls myminio/test1
[2020-11-19 08:51:15 CST] 6.7KiB anaconda-ks.cfg
root@172.25.78.11 ~ $ 
```

- 每个minio节点的测试对象的信息

```
// 11节点
root@172.25.78.11 /data/minio $ tree ./
./
└── test1
    └── anaconda-ks.cfg
        ├── 73131daa-b081-40f1-a804-546e361e7fbc
        │   └── part.1
        └── xl.meta

// 12节点
root@172.25.78.12 /data/minio $ tree ./            
./
└── test1
    └── anaconda-ks.cfg
        ├── 73131daa-b081-40f1-a804-546e361e7fbc
        │   └── part.1
        └── xl.meta

// 13节点
root@172.25.78.13 /data/minio $ tree ./ 
./
└── test1
    └── anaconda-ks.cfg
        ├── 73131daa-b081-40f1-a804-546e361e7fbc
        │   └── part.1
        └── xl.meta


// 14节点
root@172.25.78.14 /data/minio $ tree ./
./
└── test1
    └── anaconda-ks.cfg
        ├── 73131daa-b081-40f1-a804-546e361e7fbc
        │   └── part.1
        └── xl.meta


```

- 因为这是4个节点，每个节点都有一块磁盘，所以一个测试对象通过EC会有2个分割的数据块和2个数据校验块，创建bucket实际都是通过minio rpc服务在每个数据目录下创建目录(test1)，一个对象的上传需要在bucket所对应的目录下创建和对象名称相同的目录(anaconda-ks.cfg)，这个目录(test1/anaconda-ks.cfg)下存储有一个UUID目录(数据内容的唯一标识）和一个元数据文件( xl.meta)。UUID目录下存储对象的分片信息(73131daa-b081-40f1-a804-546e361e7fbc/part.1)。
- 假设有4亿小文件，在整个minio集群中每个对象会有4亿个元数据文件+ 4亿个分片(至少1个分片)+4亿个uuid目录，对于操作系统来说，节点的inode会被放大至少3倍。那性能大家就可以想想。其次是每个对象的元数据目前不支持和数据分开的模式，海量的小文件的元数据从慢速设备加载过程也是一个问题，即使能加载到内存，go的内存消耗也比较多。目前我也不清楚为啥minio的元数据不能和数据分开。
- minio完全可以采用数据和元数据分开，元数据采用ssd或者nvme存储，至于数据完全采用hdd；或者元数据直接用一个nosql存储，每个minio节点部署一个nosql,用来查询数据的元数据。但是这样也无法解决小文件问题，本来小文件就很小，在做EC分片后更小，inode也剧增，这是一个问题。
- minio从它的后端存储架构上来看，比较适合比较大的对象存储，比如大于16M的对象。这样即使做EC，每个分片对象至少能在1M以内。