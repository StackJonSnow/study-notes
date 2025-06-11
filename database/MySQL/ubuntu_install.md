### 创建用户及用户组
1. 新建用户
adduser web_server
2. 新建用户组
groupadd biz
3. 将用户添加到用户组
usermod -aG biz web_server
4. 创建用户组操作目录
mkdir /opt/biz
5. 操作目录授权
更改目录归属
chown web_server:biz /opt/biz
授权
chmod 750 /opt/biz

### 为用户配置ssh登录
1. 创建.ssh目录
mkdir -p ~/.ssh
2. 目录授权
chmod 700 ~/.ssh
3. 添加公钥
echo "公钥内容" >> ~/.ssh/authorized_keys
4. 公钥文件授权
chmod 600 ~/.ssh/authorized_keys
5. 配置ssh服务
vim /etc/ssh/sshd_config
检查如下配置项
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
然后
systemctl restart sshd
6. 登录
ssh web_server@ip

### 安装mysql
1. 更新系统包
apt update
2. 安装
apt install mysql-server
3. 启动
systemctl start mysql
4. 停止
systemctl stop mysql
5. 查看状态
systemctl status mysql
6. 配置 mysql
mysql_secure_installation
7. 登录
mysql -u root -p

### mysql 初始化
1. 创建数据库
CREATE DATABASE example_db CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
2. 创建用户并授权
CREATE USER 'example_user'@'%' IDENTIFIED BY 'password123!';
GRANT ALL PRIVILEGES ON example_db.* TO 'example_user'@'%';
FLUSH PRIVILEGES;


