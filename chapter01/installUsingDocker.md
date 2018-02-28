# windows 使用docker 安装mysql 方法
### 此markdown 记录windows10 上使用 docker 安装mysql 并在本地使用workbench打开的过程

### step 1 :  修改源，加速镜像下载
方法：
- 进入default 机器
  - docker-machine ssh default
- 修改配置文件
  - vi /var/lib/boot2docker/profile
  -  在--label provider=virtualbox的下一行添加--registry-mirror https://xxxxxxxx.mirror.aliyuncs.com
- exit 退出

### step 2 ： pull mysql docker 镜像

在docker quickStart terminal 中键入如下操作：

```
  docker pull mysql/mysql-server
```

此操作将拉取最新的官方mysql 镜像


### step 3 ： 利用镜像生成一个容器

- 注意这里的要点：
  - 1、需要把镜像的数据库端口映射到给default虚机的端口
  - 2、建立容器时使用 -e 传入需要的环境变量
  - 3、环境变量中需要设置root 密码

- 操作如下：
```
docker run --name xuan-mysql -p 53306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql/mysql-server
```
- 注：
  - name 是容器名称
  - 53306 是需要绑定的default开放的端口
  - MYSQL_ROOT_PASSWORD 用于设定默认执行
  - -d 表示守护进程

### step 5 ： 检查default 是否成功开放53306

- 1 检查容器状态

  ```
    docker ps
  ```
- 2 检查default 开放端口情况

  - ssh 到机器中
    ```
      docker-machine ssh default
    ```
  - 安装 net-tools 工具
    ```
      yum install net-tools
    ```
  - 检查端口开放情况
    ```
      netstat -an | grep 53306   
    ```
  - 看到listen则成功

### step 6 : 更改容器中数据库的远程访问权限
- 进入docker
  ```
    docker ps
    docker exec -it xuan-mysql /bin/bash
  ```
- 登入数据库
  ```
     mysql -uroot -p
  ```

- 执行操作
  ```
    show databases;
    use mysql;
    update user set host = '%' where user = 'root';
    FLUSH PRIVILEGES;
    select host ,user from user;

  ```
- 确认root对应的的host是'%'即可

### step 7 : 从本地连接数据库：
- 注意逻辑：
  - docker tools 在本地建立了一台机器default
  - 机器的局域网ip为192.168.99.100
  - 机器的端口53306 和容器的端口3306绑定
  - 所以，访问容器中的数据库，只需要访问default:53306即可

- 在mysql workbench中新建连接：
  - hostname : 192.168.99.100:53306
  - password : 123456

- 测试连接即可

### 至此，可以成功连接数据库
