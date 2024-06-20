# chatserver
基于muduo的集群聊天服务器项目（技术栈：C++ Nginx MySQL Redis ）
本项目是使用Windows下vscode远程操作Linux完成的
其中采用集群的方式提高并发量，采用了Nginx实现负载均衡
基于发布-订阅的服务器中间件redis消息队列实现跨服务器的客户端通信

编译方式
cd build
1.清理文件 rm -rf *
2.生成makefile  cmake ..
3. make

配置：
nginx配置基于tcp的负载均衡
1.安装
2.nginx编译加入--with-stream参数激活tcp负载均衡模块
3.配置nginx.conf文件添加tcp负载均衡(这里采用的是轮询算法）
stream{
  upstream MyServer{
    server 127.0.0.1:6001 weight = 1 max_fails=3 fail_timeout=30s;
    server 127.0.0.1:6002 weight = 1 max_fails=3 fail_timeout=30s;
    }
    server{
      proxy_connect_timeout 1s;
      listen 800;
      proxy_pass  MyServer;
      tcp_nodelay on;
    }
  }
Redis
1.redis环境安装和配置 ubuntu：sudo apt-get install redis-server
2.redis支持多种不同的客户端编程语言C++对应的则是hiredis，安装hiredis
    git clone https://github.com/redis/hiredis
    cd hiredis
    make
    sudo make install

MySQL ubuntu环境安装mysql-server和mysql开发包
1.安装最新版MySQL服务器  sudo apt-get install mysql-server 
  安装开发包 sudo apt-get install libmysqlclient-dev 
2.修改默认用户名和密码
3. 登录  mysql -u 用户名 -p
4.创建chat数据库
5.创建表：user（存储用户id 姓名 密码 登录状态）
        friend（存储好友id 用户id）
        allgroup（群id 群名 群描述）
        groupuser（群id 在该群中所有用户的id 用户对应的角色（创建者 普通））
        offlinemessag（用户id 离线后收到的messa）

项目内容：
实现了在不同服务器上的客户端进行通信，包括一对一通信，群组通信，返回离线消息。
客户端业务：注册 登录 退出；登录成功后：添加好友，一对一聊天，创建群，加入群，发起群聊，注销登录等
服务器分为三部分，网络层处理网络消息，业务层处理功能业务，数据层存储数据操作数据库
为了解决业务层和数据层的耦合是增加一个usermodel专门处理数据库的增删改查

文件夹介绍：
bin 生成可执行文件：./ChatServer ./ChatClient
include 头文件
src  源文件
build  编译过程中临时中间文件
thirdparty 第三方库文件
CMakeLists.txt  利用cmake管理文件 构建编译环境
