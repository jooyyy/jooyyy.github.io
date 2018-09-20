## Docker 归纳

#### 核心概念

* 镜像 image
* 容器 container
* 仓库 

#### 常用命令

* docker pull ubuntu:14.04 

  以上命令相当于

  ```
  docker pull registry.hub.docker.com/ubuntu:xxxx
  
  这个是美国的源，可切换到国内的源
  
  dl.dockerpool.com
  ```

* docker inspect [image id] 

* docker search NAME

  搜索官方仓库中的镜像

  * --automated=false  显示自动创建的镜像
  * --no-trunc=false 输出信息不截断显示
  * -s, --stars=0 指定显示为指定星级以上的镜像

* docker image ls [--all]

* docker image rm [image id]

* docker container ls [--all]

* docker container kill [CONTAINER ID]

* docker container rm [CONTAINER ID]

* docker container run --rm -p 8000:3000 -it [image name]

* 发布 image 

  * 登录 docker

  * 给 image 标注用户名和版本

    ```
    docker iamge tag ubuntu:0.0.1 jooyyy/ubuntu:0.0.1
    ```



#### 创建镜像（三种方案）

* 基于已有镜像的容器创建

  拉一个镜像到本地，运行容器，进入容器，装上自己的应用-退出-保存-上传

  ```
  // eg: 制作一个带 tgl 工具的 linux 镜像
  
  
  ```

  

  * docker commit 

    * -a， --author="" 作者信息
    * -m，--message="" 提交信息
    * -p， --pause=true 提交时暂停容器运行

    ```
    docker commit -m "add file" -a "joy" dc6704fe4313 test 
    ```

  

* 基于模板文件导入（不怎么常用呢？）

  * 推荐使用 OpenVZ 提供的模板来创建

  * [http://openvz.org/Download/templates/precreated](http://openvz.org/Download/templates/precreated)

    ```
    cat ubuntu-14.04-x86_64-minimal.tar.gz | docker import - ubuntu:14.04
    ```

  

* 基于 Dockerfile 

  写一个 Dockerfile 文件，里面写上自动执行脚本，去下载哪些工具，如何配置网络和数据卷 blabla

  >#### Dockerfile编辑
  >
  >最佳实践总结
  >
  >- 编写.dockerignore文件
  >
  >  - 控制 image 大小，减少冗余代码
  >
  >- 容器只运行单个应用
  >
  >  - 从技术的角度，一个容器中可以运行多个进程，但是这样 build 的时间会很长，修改单一的进程模块成本很大；所以推荐为每个进程构建单独的Docker 镜像，然后用 Docker Compose 运行多个 Docker 容器
  >
  >- 将多个RUN指令合并为一个
  >
  >- 基础镜像的标签不要用latest
  >
  >- 每个RUN指令后删除多余文件
  >
  >- 选择合适的基础镜像(alpine版本最好)
  >
  >  ```
  >  FROM node:7-alpine
  >  ```
  >
  >- 设置WORKDIR和CMD
  >
  >  WORKINGDIR 是运行 RUN / CMD / ENTRYPOINT 指令的地方
  >
  >  CMD 可以直接写命令，但最好是将命令放在一个数组中
  >
  >  ```
  >  CMD ["npm", "start"]
  >  ```
  >
  >- 使用ENTRYPOINT (可选)
  >
  >  预执行的 bash 脚本路径
  >
  >  ```
  >  ENTRYPOINT["./entrypoint.sh"]
  >  ```
  >
  >  
  >
  >- 在entrypoint脚本中使用exec
  >
  >- COPY与ADD优先使用前者
  >
  >- 合理调整COPY与RUN的顺序
  >
  >- 设置默认的环境变量，映射端口和数据卷
  >
  >- 使用LABEL设置镜像元数据
  >
  >- 添加HEALTHCHECK
  >
  >  运行容器的时候可以指定--restart always 让 Docker 守护进程重启容器；如果配置错误则会一直重启，这就需要一个 break，HEALTHCHECK就是这样一个判断容器状态的参数
  >
  >  ```
  >  EXPOSE $APP_PORT  
  >  HEALTHCHECK CMD curl --fail http://localhost:$APP_PORT || exit 1
  >  ```
  >
  >  

#### 本地存出和载入镜像

```
docker save -o ubuntu_14.04.tar ubuntu:14.04
docker load --input ubuntu_14.04.tar
```



#### 上传镜像

* 注册 docker 账号

* 给自己的镜像打标签

  ```
  docker tag test:latest user/test:latest
  ```

* 推到远程仓库

  ```
  docker push user/test:latest
  ```

  