## 部署项目使用说明（PHP）
### 说明
```
为什么要使用docker来部署
    容器对于服务的性能影响是有的，但是这个影响较小，相比使用容器带来运维成本的降低带来的收益来讲，可谓利大于弊，容器化可以将项目运行环境标准化，并能够快速横向扩展
如何横向扩展
    - 在多台服务器上按照同样的方法部署（改端口号，不再直接暴露80端口）
    - 额外部署一台nginx服务器进行负载均衡即可
    - mysql、redis可以选择一个或多个实例来使用，数据持久到宿主机，docker容器无论是否销毁数据都不会直接丢

本项目中都打包了哪些软件
    - nginx 1.25.3
    - PHP 8.2
        - composer
        - npm
    - mysql 5.7
    - redis 7.2


```
### 0. 准备工作
- 宿主机上安装git
- 宿主机上安装docker
```
注：宿主服务器上只需要安装这两个即可
composer、npm均已经打包进php镜像中了，要使用直接进入容器进行操作
```
### 1. clone lnmp

- git clone https://github.com/weiok550/lnmp.git
- cd lnmp
- mkdir apps

### 2. 克隆自己的php项目并构建

- cd lnmp/apps
- git clone {php项目仓库地址}
- cd {项目目录}
- 以下以laravel为例子，构建项目【如果项目本身无需构建忽略以下步骤】
  - docker exec -it php82 /bin/sh
  - cd /home/www/apps/{项目目录}
  - composer install
  - npm install
  - npm run build
  - composer install --optimize-autoloader --no-dev   //优化自动加载器
  - php artisan config:cache   //优化配置加载
  - php artisan route:cache    //优化路由加载
  - php artisan view:cache     //优化视图加载
  - .env 配置文件优化
    - APP_DEBUG=false   // 关闭调试模式

### 3. 修改 nginx/conf/default.conf
```
1. server_name  改为自己的域名（2处）
2. root  改为php站点根目录（index.php所在目录），/home/www/apps 对应着本地的lnmp/apps目录无需改动，只需要改/home/www/apps/后面的路径 
```

### 4. ssl 证书文件改为正式的

- nginx/conf/cert.perm 文件替换为正式的
- nginx/conf/key.perm  文件替换为正式的

### 5. 启动项目

- 项目构建（以laravel项目为例）
    - docker exec -it php82 /bin/bash   // 进入容器操作
    - cd /home/www/apps/{项目目录}
    - composer install
    - npm install
    - npm run build
    - exit  // 退出容器
- 启动web服务
    - cd lnmp
    - docker-compose up -d 

### 6. 重启项目-无特殊情况不需要重启
- cd lnmp
- docker-compose restart

### 7. 项目更新发布
- 以laravel项目为例
  - cd lnmp/apps/{项目目录}
  - git pull    // 拉取最新的项目代码
  - docker exec -it php82 /bin/bash // 进入容器操作
  - cd /home/www/apps/{项目目录}
  - php artisan cache:clear     // 清除所有缓存
  - npm install
  - npm run build               // 打包静态资源
  - composer install --optimize-autoloader --no-dev   //优化自动加载器
  - php artisan config:cache   //优化配置加载
  - php artisan route:cache    //优化路由加载
  - php artisan view:cache     //优化视图加载 
