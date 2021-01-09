## Goland导入minio源代码

| 作者 | 时间 |QQ技术交流群 |
| ------ | ------ |------ |
| perrynzhou@gmail.com |2020/12/01 |中国开源存储技术交流群(672152841) |

### 下载mino源代码

  ```
  mkdir github.com/minio -p
  // minio项目采用go module方式，minio项目顶层目录为github.com/minio
  cd github.com/minio/ && git https://github.com/minio/minio.git
  ```

  

### Goland 设置GoModule,完成项目导入

- Goland中右下角的 Go Modules  are detected 中，点击 Enable Intergration

    ![goland-1](../images/goland-1.png)
- 设置 Minio项目中的Go Module

    ![goland-2](../images//goland-2.png)