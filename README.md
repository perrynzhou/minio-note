# Minio存储系统 

| 作者 | 时间 |QQ技术交流群 |
| ------ | ------ |------ |
| perrynzhou@gmail.com |2020/12/30 |中国开源存储技术交流群(672152841) |




## minio 运维

- [minio集群部署](./document/md/minio集群部署.md)
- [Minio使用分析](./document/md/Minio使用分析.md)


## minio 源代码分析
- [Goland导入minio源代码](./document/md/Goland导入minio源代码.md)


## 文件解压缩

```
//分割大文件为小文件
split -b 30m  minio-git.tar.gz  minio-git/minio-git.tar.gz.

//合并小文件为大文件
cat  minio-git/minio-git.tar.gz.  > minio-git.tar.gz