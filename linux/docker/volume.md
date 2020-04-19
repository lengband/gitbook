### Docker的持久化存储和数据共享

#### 第一种方案：Data Volume
在Dockerfile中添加类似如下信息
`VOLUME /var/lib/mysql`，这样创建出来的mysql容器的数据会在host上面`/var/lib/mysql`存储。

但是这样创建出来的volume是存储在`/var/lib/docker/volumes/dsadf32423f23rfd32xc4c23cf4324d/_data`的，可见随机生成的文件夹不好维护，因此一般都在运行`container`时指定`Volume`名称,如：`docker run -d -v mysql:/var/lib/mysql --name mysql1 -e MYSQL_ALLW_EMPTY_PASSWORD=true msyql`，而且可以在下次创建其它`container`可以复用,如：`docker run -d -v mysql:/var/lib/mysql --name mysql2 -e MYSQL_ALLW_EMPTY_PASSWORD=true msyql`, 这样在mysql1中的数据库添加数据时，mysql2也能同步，因为两个都是一个数据库


#### 第二种方案: Bind Mounting（数据映射）（比较方便和推荐）
`docker run -v /home/aaa:root/aaa` 本地home/aaa映射容器root/aaa
实际场景：
比如现在有一个web项目，在生产环境出现bug，但是本地无法复现，需要在本地调试生产环境，比如项目地址是`https://hub.docker.com/repository/docker/lengband/flask-skeleton`,项目pull下来之后，运行
`docker run -d -p 80:5000 -v ${pwd}:/skeleton --name flask-skeleton lengband/flask-skeleton`,即可本地更改代码同步到docker容器中，打开本地`127.0.0.1`可以看到效果