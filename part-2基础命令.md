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
