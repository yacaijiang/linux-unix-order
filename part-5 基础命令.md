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
#  IPC（InterProcess Communication）：进程间通信 </br>
   1. 常用进程间通信方式：
    1. 管道：最简单
    2. 信号：开销最小
    3. 共享映射区：无血缘关系
    4. 本地套接字：最稳定
# 管道：
1. 特点：</br>
    本质是一个伪文件</br>
    由两个文件描述符引用，一个表示读，一个表示写</br>
    规定数据从管道的写流入管道，从读流出
2. 原理：</br>
    内核使用环形队列，借助内核缓冲区（4k）实现。
3. 局限性：</br>
    数据自己读不能自己写</br>
    数据一旦被读走，便不在管道中存在，不可反复读取</br>
    由于采用半双工通信，数据单方向流动</br>
    只能在由公共祖先的进程中使用管道</br>
4. 函数：</br>
    #include<unistd.h>
    int pipe(int pipefd[2]);
    int pipe2(int pipefd[2],int flag);
# 共享内存：
1. mmap函数：</br>
    ```
       #include <sys/mman.h>
       void *mmap(void *addr, size_t length, int prot, int flags,
                  int fd, off_t offset);
       int munmap(void *addr, size_t length);//释放mmap映射区
    ```
    1. 参数：</br>
        addr//建立映射区首地址，Linux内核指定，使用时传NULL</br>
        length:想要创建映射区大小</br>
        prot：映射区权限PROT_READ PROT_WRITE PROT_READ|PROT_WRITE</br>
        flags:标志位参数（常用于设定更新物理区域，设置共享、创建匿名映射区）</br>
            MAP_SHARED:将映射区所做操作反映到物理设备（磁盘）</br>
            MAP_PRIVATE:映射区所做的修改不会反映到物理设备</br>
        fd： 用来建立映射区的文件描述符</br>
        offset：映射文件偏移（4k整数倍）</br>
    2. 返回值：</br>
        成功：返回映射区首地址</br>
        失败：MAP_FAILED宏</br>
    3. ![mmap注意事项](https://user-images.githubusercontent.com/37798962/90210830-b169a800-de21-11ea-8921-53bf71147b7e.jpg)
2. 借助共享内存放磁盘文件：</br>
3. 父子进程、血缘关系进程 通信</br>
    MAP_SHARED:共享映射区</br>
    MAP_PRIVATE:独占映射区</br>
4. 匿名映射区</br>
    使用MAP_ANONYMOUS(MAP_ANON):</br>
    int *p=mmap(NULL,4,PROT_READ|PROT_WRITE,MAP_SHARED|MAP_ANONYMOUS,-1,0);//4即位置表大小，按实际需要填写
# 信号
1. 概念：简单、不携带大量信息、满足某个特设条件
# 守护进程
1. 概念： 后台服务进程，通常独立于控制终端并周期性执行某种任务或者等待处理某些发生的事件，一般采用d结尾的名字，如：预读入缓输出机制的：ftp服务器 nfs服务器
2. 创建守护进程模型：
    1. 创建子进程，父进程退出
    2. 子进程中创建新的会话
        sersid（）函数
    3. 改变当前目录为根目录
        chdir（）函数
    4. 重设文件权限掩码
        umask（）函数
    5. 关闭文件描述符
    ```
    close(STDIN_FILENO);
    open("/dev/null",O_RDWR);
    dup2(0,STDIN_FILENO);
    dup2(0,STDIN_FILENO);
    ```
    6. 开始执行守护进程核心工作
    7. 守护进程退出处理程序模型
# 线程：
1. 概念：</br>
    线程：有PCB，没有独立的地址空间（最小的执行单位）</br>
    进程：独立地址空间，有PCB（最小分配资源的单位）</br>
    线程可以看作寄存器和栈的集合</br>
    线程共享资源：</br>
    1. 文件描述符表
    2. 每种信号的处理方式
    3. 当前工作目录
    4. 用户ID和组ID
    5. 内存地址（.text/.data/.bss/heap/共享库）
    非共享资源：</br>
    1. 线程id
    2. 处理器现场和栈指针
    3. 独立的栈空间（用户空间栈）
    4. errno变量
    5. 信号屏蔽字
    6. 调度优先级
    优点：</br>
    1. 提高程序并发性
    2. 开销小
    3. 数据通信、共享数据方便
    缺点：</br>
    1. 库函数不稳定
    2. 调试编写困难，gdb不支持
    3. 对信号支持不好
2. 控制原语：
    1. pthread_self函数
    ```
       #include <pthread.h>

       pthread_t pthread_self(void);

       Compile and link with -pthread.
    ```
    2. pthread_create函数
    ```
        #include <pthread.h>

       int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                          void *(*start_routine) (void *), void *arg);

       Compile and link with -pthread.
    ```
    3. pthread_exit函数
    ```
       #include <pthread.h>

       void pthread_exit(void *retval);

       Compile and link with -pthread.
    ```
3. 属性：
    
4. 注意事项：