# gcc编译阶段： 
1. 预处理：头文件展开，宏替换     gcc -E main.c -o main.i
2. 编译：c变成汇编               gcc -S main.i -o main.s
3. 汇编：汇编文件变成二进制文件   gcc -c main.s -o main.o
4. 链接：函数库中相应的代码组合到目标文件中 gcc main.o -o main.exe
5. 执行 ./main.exe
6. 直接完成四步： gcc main.c -o main.exe
7. 指定头文件目录：gcc main.c -I 目录名 -o main.exe
8. 指定宏：gcc main.c -I 目录名 -o main.exe -D 宏 
9. 优化：-O0/1/2/3（优化代码） -Wall 提示 -g 增加调试信息
# 静态库：
1. 命名规则
    1. lib+库名+.a
    2. libmytest.a
2. 制作步骤
    1. 生成对应.o文件
    2. 生成的.o文件打包：ar rcs libmytest.a *.o
3. 发布和使用静态库
    1. 发布静态库：
    2. 头文件：
    3. 使用：gcc main.c 静态库 -o main.exe -I 目录名
    4. 使用：gcc main.c -I 头文件目录名 -L 静态库目录名 -l mytest -o main.exe
4. 优点：
    1. 发布程序时不需要提供库接口
    2. 加载速度快
5. 缺点：
    1. 库被打包到应用程序，库体积很大
    2. 库发生改变，需要重新编译
# 共享库：
1. 命名规则： lib+名字+.so
2. 制作步骤：
    1. 生成与位置无关代码： gcc -fPIC -c *.c
    2. 打包成共享库：gcc -shared -o libmytest.so *.o
3. 发布和使用动态库：
    1. 使用：gcc main.c libmytest.so -o main.exe 
    2. 使用：gcc main.c -I 头文件目录 -L 动态库目录 -l mytest -o main.exe
ldd main.exe 查找所有依赖的库
4. 动态链接库链接失败：
    1. 临时：export LD_LIBRARY_PATH=当前动态库所在目录
    2. 方法：
        1. 需要找动态链接器的配置文件 -- etc/ld.so.conf
        2. 动态库的路径写到配置文件中 -- 当前动态库所在目录的路径
        3. 更新-- sudo ldconfig -v
    3. 放到系统/lib目录下 ---不可以使用
    4. 在家目录的.bashrc中添加一句话：export LD_LIBRARY_PATH=当前动态库所在目录的路径，然后重启终端
5. 优点：
    1. 执行体积小
    2. 动态库更新不需要重新编译程序
6. 缺点：
    1. 发布程序前需要将动态库提供给用户
    2. 没有被打包到应用程序中，加载速度比静态库慢
# gdb调试
 1. 启动：
    1. start 只执行一步 n s c
    2. 查看代码：l   l 文件名：行号
    3.设置断点：
        b 行号
        b 文件名 行号 
    4. 删除断点：
        d 断点编号
    5. 单步调试
        进入函数体：s 
        跳出：finish
        不进入函数体：n
        退出循环：u
    6. 查看变量值： p
    7. 查看变量类型： ptype  变量名
    8. 设置变量值：set var 变量名=值
    9. 设置追踪变量：
        display 变量名
        undisplay 变量名
        info display
    10. 退出gdb调试：
        quit
# makefile
1. makefile的命名：makefile/Makefile
2. 规则：目标、依赖、命令
```
    目标:依赖条件
        命令
```
3. 函数实用：
    1. 方法一
```
    app:main.o sub.o mul.o
        gcc main.0 sub.o mul.o -o app
    %.o:%.c
        gcc -c %< -o %@
```
2. 方法二
```
obj=main.o sub.o mul.o
target = app
//makefile自己维护的变量（大写）
CC = cc //gcc
CPPFLAG = -I//预处理器
CFLAGS = -Wall//编译时使用的参数-g -c
LDFLAGS = -L //链接库使用选项
&(target):$(obj)
    gcc $(obj) -o $(target)
    %.o:%.c
        gcc -c %< -o %@
```
%<: 规则中第一个依赖</br>
%@：规则中的目标</br>
%^：规则中的所有依赖，只能在规则的命令中使用</br>
3. 方法三：
```
src = $(wildcard ./*.c)
obj = $(patsubst ./%.o, ./%.c, $(src))
target = app
//makefile自己维护的变量（大写）
CC = cc //gcc
CPPFLAG = -I//预处理器
CFLAGS = -Wall//编译时使用的参数-g -c
LDFLAGS = -L //链接库使用选项
&(target):$(obj)
    gcc $(obj) -o $(target)
    %.o:%.c
        gcc -c %< -o %@
.PHONY:clean
clean: //make clean
    -mkdir /test//加"-"可以忽略该命令
    rm $(obj) $(target) -f
hello: //make hello
    echo "hello !"
```