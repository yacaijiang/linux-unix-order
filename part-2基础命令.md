# U盘的挂载
1. U盘的卸载：umount u盘的路径//一定不能在U盘目录下卸载，应该退到上一级
2. U盘的挂载：mount  u盘名  挂载目录（一般为/mnt，若挂载到其他目录时，在挂载过程中暂时看不到该目录下原有文件）
3. 获取U盘名：fdisk -l  其中有U盘的名称
# Linux系统 
## 磁盘设备种类：
sd--SCSI Device
hd--硬盘
fd--floppy disk软盘
硬盘1：sda 
    主分区：最多四个：sda1-sda4
    扩展分区：从sda5-...
硬盘2：sdb
硬盘3：sdc

# 压缩包管理
1. 屌丝版：
    不能压缩目录
   1. gzip--.gz格式的压缩包
    压缩：gzip 文件名
    解压缩：gunzip 文件名
   2. bzip2--bz2格式的压缩包
    压缩：bzip2 -k 文件名   //可以保留源文件
    解压缩：bzip2 文件名
2. 
   1. tar  不使用z/j参数，该命令只能对文件或者目录打包
    参数：
        c--创建--压缩
        x--释放--解压缩
        v--显示提示信息--压缩解压缩--可以省略
        f--指定压缩文件名字
        z--使用gzip方式压缩--.gz
        j--使用bzip2方式压缩--.bz2
    压缩：
        tar zcvf 文件名.tar.gz 文件/目录
        tar jcvf 文件名.tar.bz2 文件/目录
    解压缩：
        tar jxvf 压缩包（当前目录）
        tar zxvf 压缩包（当前目录）
        tar jxvf 压缩包 -C 目录 指定目录
        tar zxvf 压缩包 -C 目录 指定目录
   2. rar
    参数：
        压缩：a
        解压缩：x
    压缩：
        rar a 文件名  要压缩的文件/目录
    解压缩：
        rar x 文件名  （指定解压缩目录）
3. 
   1. zip
   参数：
   压缩：
    zip 压缩包名字 要压缩的文件/目录
   解压缩：
    unzip 压缩包名字
    unzip 压缩包名字 -d 解压目录
# 进程管理
    ps a//查看所有用户信息
    ps au//查看更多信息
    ps aux//查看没终端的程序
    PID：进程序号
## 管道
    符号：“|” //ps aux | grep xxxx
1. kill使用：
    kill -l//查看所有命令
    kill -SIGNKILL XXX//杀死某一个进程
2. 环境变量：
    key-value
    key-value：key-value：key-value
3. 任务管理器：
    top
# 网络
1. 查看ip：ifconfig// eth0：当前网卡信息，
2. 用户管理：
    1. 创建用户：
        sudo adduser 用户名
    2. 切换用户：
        su 用户名
    3. sudo useradd -s /bin/bash -g Robin -d /home/Robin -m Robin
    4. 添加用户组：
        sudo groupadd 用户组名
    5. 修改密码：passwd
    6. 退出当前用户：exit
    7. 删除用户:
        sudo deluser 用户名
        sudo userdel -r 用户名
# ftp服务器搭建 vsftpd
## 作用： 文件上传和下载
1. 服务器端：
    1. 修改配置文件
        cd /etc/
        gedit vsftpd.conf
        anonymous_enable=NO//运行匿名用户登录
        local_enable=YES//本地用户登录
        #write_enable=YES//实名用户拥有写权限
        #local_umask=022//设置本地掩码
        #anon_upload_enable=YES//匿名用户可以向ftp服务器上传数据
        #anon_mkdir_write_enable=YES//匿名用户可以在ftp服务器上创建目录     
    2. 重启服务
        sudo service vsftpd restart
2. 客户端：
    1. 实名用户登录
        ftp+IP（server）
        输入用户名
        输入密码
        1. 文件上传：
            put 文件名
        2. 文件下载：
            get 文件名
    2. 匿名用户登录
        ftp+IP（server）
        输入用户名：anonymous
        输入密码：回车
        1. 文件上传：
            put 文件名
        2. 文件下载：
            get 文件名
    3. lftp客户端访问ftp服务器
        1. ftp客户端工具
        2. 登录
            1. 匿名：
                lftp ip 
                login
            2. 实名：
                lftp 用户名@ip
                输入服务器密码
        3. 文件传输：
            put 上传文件
            mput 上传多个文件
            get 下载文件
            mget 下载多个文件
            mirror 下载整个目录极其子目录
            mirror -R 上传整个目录极其子目录
# nfs服务器
## 作用：网络文件系统，共享文件夹
1. 服务器：
    1. 创建共享目录
        mkdir dir
    2. 修改配置文件
        /etc/exports
    3. 重启服务器：
        sudo service nfs-kernel-server restart
2. 客户端：
    1. 挂在服务器共享目录
        mount ip：共享目录名 要挂载的目录
# ssh服务器
1. 服务器：
    1. 安装：
    2. 登录：
    ssh 用户名@ip 
    3. 退出： logout
2. 客户端：
# scp命令
## 作用：超级拷贝，不同主机之间拷贝
scp -r 用户名@ip：待拷贝目录 拷贝到的目录