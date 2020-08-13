# 库函数
![PCB和文件描述符](https://user-images.githubusercontent.com/37798962/90091567-52d8f700-dd59-11ea-9c77-3ae2fdf4a0e4.jpg)
</br>
![虚拟地址空间](https://user-images.githubusercontent.com/37798962/90100598-9b031400-dd6f-11ea-82ed-7e5da7918df8.jpg)
</br>

# 虚拟地址空间：可用地址空间的大小（如32位机，2^32=4G）
1. 方便编译器和操作系统安排程序的地址分布
2. 方便进程之间隔离
3. 方便os使用内存

# 进程和程序
1. 程序： 编译好的二进制文件，在磁盘上，不占用系统资源；
2. 进程： 是活跃的程序；
3. 专业名词： 时钟中断（硬件手段）、单道程序设计模式、多道程序设计模式（并行运行）
# CPU
# mmu（内存管理单元）
1. 虚拟地址和物理地址之间映射
2. 内存访问级别的设置（inter：4个级别：0-3；0级最高；linux：0，3）
3. 按页分配：不足一页分配一页
# 进程控制块PCB
1. 结构体：task_struct;
2. 内容：
    1. 进程id
    2. 进程状态：就绪、运行、挂起（等待）、停止等
    3. 进程切换时需要保存和恢复一些CPU寄存器
    4. 描述虚拟地址空间的信息
    5. 描述控制终端的信息
    6. 当前工作目录
    7. umask掩码：保护文件创建或者删除
    8. 文件描述符表，指向file的指针
    9. 和信号相关的信息
    10. 用户id和组id
    11. 会话和进程组
    12. 进程可以使用的上限资源（ulimit -a//查看资源的上限）
# 进程控制
1. fork函数
    1. 返回值有两个：1个进程变成两个，各自对fork返回
        1. 返回子进程pid（父进程）
        2. 返回0（子进程）
2. 循环创建子进程：
```
int i;
pid_t pid;
for(int i=0;i<5;i++){
    pid=fork();
    if(pid==-1){
       perror("fork error");
    }else if(pid==0){//新创建的子进程
         break;//防止子进程创建子进程的子进程
    }
}、
//谁先执行谁后执行不确定
if(i<5){//子进程执行
    sleep(i);
    printf("i am %dth child,pid =%u\n",i+1,getpid());
}else{
    sleep(i);
    printf("i am parent\n");
}
```
3. 父子进程共享
    1. 相同点：全局变量、data text 栈 堆 环境变量...
    2. 不同点：进程id fork返回值 父进程id 进程运行时间 闹钟（定时器） 未决信号集
    3. 父子进程遵循：读时共享、写时复制原则
    4. 共享：文件描述符 mmap建立的映射区
4. gdb调试
    set follow-fork-mode child 设置gdb在fork后跟踪子进程。</br>
    set follow-fork-mode parent 设置gdb在fork后跟踪父进程。</br>
    在fork函数调用前设置有效</br>
5. exec函数族：(成功不返回，失败返回-1)
    1. execlp函数：list path</br>
    int execlp(const char *file,const char *arg, ...);</br>
    execlp("ls","ls","-a",NULL);</br>
    实现系统调用程序：ls data cp cat
    2. execl函数：通过路径+程序名加载
    int execl(const char *path,const char * arg,...);</br>
    execlp("/bin/ls","ls","-a",NULL);</br>
    3. execvp函数
    int execl(const char *path,const char * arg[],...);</br>
6. dup2函数
    int dup2(int oldfd, int newfd); newfd覆盖oldfd
7. 孤儿进程：父进程先于子进程结束，子进程成为init的子进程
8. 僵尸进程：进程终止，父进程尚未回收，子进程残留PCB存放内核，变成僵尸进程
    1. 如何清除僵尸进程：
        1. wait函数：
            pid_t wait(int *status);</br>
            wait(NULL);
            //WIFEXITED(status)非0 正常退出</br>
            WEXITSTATUS(status) 获取进程退出状态</br>
            WIFSIGNALED(status) 非0 进程异常终止</br>
            WTERMSIG(status) 获取使得进程终止的信号编号</br>
        2. waitpid函数:
            pid_t waitpid(pid_t pid,int *status,int optition);</br>
    wait和waitpid每次只能kill一个子进程