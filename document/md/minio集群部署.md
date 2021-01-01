## minio集群部署

| 作者 | 时间 |QQ技术交流群 |
| ------ | ------ |------ |
| perrynzhou@gmail.com |2020/11/14 |中国开源存储技术交流群(672152841) |

### 硬件环境

| 节点   | 磁盘         | 容量   |
| ------------ | ------------ |------------ |
| 172.25.78.11 | sda(ssd) | 3.5T|
| 172.25.78.12 | sdd(ssd)       | 3.5T    |
| 172.25.78.13 | sdd(ssd)        |  3.5T  |
| 172.25.78.14 | sdg(ssd)       |  3.5T  |


### minio二进制下载

- 服务端下载

```
wget https://dl.min.io/server/minio/release/linux-ppc64le/minio
chmod +x minio
```

- 客户端下载

```
wget https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
```

### 部署服务端

- 准备工作
```
//每个节点
mkfs.xfs -f /dev/sda && mkdir /data/minio && mount  /dev/sda /data/minio 

// 11节点
echo ''/dev/sda  /data/minio                    xfs    defaults        0 0">> /etc/fstab

// 12、13节点
echo ''/dev/sdd  /data/minio                    xfs    defaults        0 0">> /etc/fstab

// 14节点
echo ''/dev/sdg  /data/minio                    xfs    defaults        0 0">> /etc/fstab

```

- 启动minio-sever
```
// 11、12、13、14节点执行如下命令启动minio-server
export MINIO_ACCESS_KEY="admin"
export MINIO_SECRET_KEY="admin"
nohup  ./minio server  http://172.25.78.11/data/minio  http://172.25.78.12/data/minio http://172.25.78.13/data/minio  http://172.25.78.14/data/minio  >  minio_server.log  2>&1  & 


```

- 部署完成后日志

```
Marking http://172.25.78.14:9000/minio/storage/data/minio/v22 temporary offline; caused by Post "http://172.25.78.14:9000/minio/storage/data/minio/v22/readall?disk-id=&file-path=format.json&volume=.minio.sys": dial tcp 172.25.78.14:9000: connect: connection refused
Waiting for all other servers to be online to format the disks.
Formatting 1st zone, 1 set(s), 4 drives per set.
Waiting for all MinIO sub-systems to be initialized.. lock acquired
Attempting encryption of all config, IAM users and policies on MinIO backend
All MinIO sub-systems initialized successfully
Waiting for all MinIO IAM sub-system to be initialized.. lock acquired
Status:         4 Online, 0 Offline. 
Endpoint:  http://172.25.78.11:9000  http://127.0.0.1:9000

Browser Access:
   http://172.25.78.11:9000  http://127.0.0.1:9000

Object API (Amazon S3 compatible):
   Go:         https://docs.min.io/docs/golang-client-quickstart-guide
   Java:       https://docs.min.io/docs/java-client-quickstart-guide
   Python:     https://docs.min.io/docs/python-client-quickstart-guide
   JavaScript: https://docs.min.io/docs/javascript-client-quickstart-guide
   .NET:       https://docs.min.io/docs/dotnet-client-quickstart-guide
```

### 使用客户端mc添加数据

- 初始化客户端配置
```
 $ ./mc config host add  myminio  http://172.25.78.11:9000  admin123456 admin123456
mc: Configuration written to `/root/.mc/config.json`. Please update your access credentials.
mc: Successfully created `/root/.mc/share`.
mc: Initialized share uploads `/root/.mc/share/uploads.json` file.
mc: Initialized share downloads `/root/.mc/share/downloads.json` file.
Added `minio` successfully.
```

- 使用

```
// 这里的minio 是之前 myminio,查看myminio下面的bucket
$ mc ls myminio

//创建test1 bucket
$ mc mb myminio/test1

//上传数据
$ mc cp 1.tar.gz myminio/test1
```

- 浏览器访问minio集群

```
访问 ：http://172.25.78.11:9000
帐号：admin
密码：admin

// 访问的用户和密码如下
export MINIO_ACCESS_KEY="admin"
export MINIO_SECRET_KEY="admin"
```
