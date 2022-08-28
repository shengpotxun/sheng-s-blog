---
title: 简单shell的七个函数实现
tags: 计算机系统
categories: 实验日志
feature: true
cover: 'http://tva1.sinaimg.cn/large/008tudqVgy1h567st53byj32ix1801l4.jpg'
abbrlink: 23f06321
date: 2022-08-14 13:06:00
---
本次的要求是实现一个简单的shell，具体是七个函数的实现。

# eval

eval这个东西是紧跟在main里面的，main获取了命令行的内容，eval就是要对命令行进行求值，而主要任务就是调用parseline系统调用去构造最终传送给execve的argv向量，指向一个参数指针表，其中第一个参数也就是argv[0]，要么是一个命令，要么其他，也就是根据路径来创建新进程运行路径下的其他程序

{% note info %}

#### 解释

shell处理命令行不是根据string来处理的，而是根据数值来处理的，argv向量指向一个数组，这个数组是一个参数指针表，这个指针表其实就是命令行的翻译

而这个时候我们就会发现一个正确的指令有两种情况

- 开头是一个内部指令，包括'ls','cd','rm'
- 开头是'./.../....'指向一个可执行文件

{% endnote %}

那我们前面的内容就可以开始写

```c_cpp
void eval(char *cmdline)
{
    char* argv[MAXARGS];   //新建execve()函数的参数，也就是指向参数指针表的一个指针
    int state = UNDEF;  //工作状态，前台或后台 
    sigset_t set;
    pid_t pid;  //进程id
    // 处理输入的数据
    if(parseline(cmdline, argv) == 1)  //解析命令行，返回给argv数组
        state = BG;
    else
        state = FG;
    if(argv[0] == NULL)  //命令行为空直接返回
        return;
    return;
}
```

同时，根据parseline的返回值还得判断到底是个前台还是个后台

如果接下来我们去看argv[0]的内容是空，那么接下来的所有内容就没什么意义了，直接返回就好，如果有内容又要分为两种情况，那就是，这个东西是不是内置指令，如果是，直接返回就好，因为后面的builtin_cmd函数会专门处理内置指令，如果不是就要根据可执行文件创建子进程了

由于并发，我们无法保证在父进程做好回收子进程的准备工作之前，子进程不会先走一步(也就是子进程已经做完了，父进程还没有把子进程放到作业集里面)，我们直接阻塞父进程信号，直到父进程把子进程加入作业集之后才解开父进程阻塞，但是由于子进程继承父进程信号集的原因(继承页表)，子进程信号集一开始也是阻塞的必须要解开

```c_cpp
void eval(char *cmdline)
{
    char* argv[MAXARGS];   //execve()函数的参数
    int state = UNDEF;  //工作状态，FG或BG 
    sigset_t set;
    pid_t pid;  //进程id
    // 处理输入的数据
    if(parseline(cmdline, argv) == 1)  //解析命令行，返回给argv数组
        state = BG;
    else
        state = FG;
    if(argv[0] == NULL)  //命令行为空直接返回
        return;
    // 如果不是内置命令
    if(!builtin_cmd(argv))
    {
        if(sigemptyset(&set) < 0)
            unix_error("sigemptyset error");
        if(sigaddset(&set, SIGINT) < 0 || sigaddset(&set, SIGTSTP) < 0 || sigaddset(&set, SIGCHLD) < 0)
            unix_error("sigaddset error");
        //在它派生子进程之前阻塞SIGCHLD信号，防止竞争 
        if(sigprocmask(SIG_BLOCK, &set, NULL) < 0)
            unix_error("sigprocmask error");

        if((pid = fork()) < 0)  //fork创建子进程失败 
            unix_error("fork error");
        else if(pid == 0)  //fork创建子进程
        {
            // 子进程的控制流开始
            if(sigprocmask(SIG_UNBLOCK, &set, NULL) < 0)  //解除阻塞
                unix_error("sigprocmask error");
            if(setpgid(0, 0) < 0)  //设置子进程id 
                unix_error("setpgid error");
            if(execve(argv[0], argv, environ) < 0){
                printf("%s: command not found\n", argv[0]);
                exit(0);
            }
        }
        // 将当前进程添加进job中，无论是前台进程还是后台进程
        addjob(jobs, pid, state, cmdline);
        // 恢复受阻塞的信号 SIGINT SIGTSTP SIGCHLD
        if(sigprocmask(SIG_UNBLOCK, &set, NULL) < 0)
            unix_error("sigprocmask error"); 
    return;
}
```

接下来就只剩下前台后台判断了，前台作业等待，后台打印

```
// 判断子进程类型并做处理
        if(state == FG)
            waitfg(pid);  //前台作业等待
        else
            printf("[%d] (%d) %s", pid2jid(pid), pid, cmdline);  //将进程id映射到job id 
```

# builtin_cmd

这就是一个内部指令的处理函数，如果不是内部指令返回0，是内部指令就直接调用对应的函数开始执行，并在执行完毕后返回1，一共就那么几种情况，全部列出来就可以了

```
int builtin_cmd(char **argv)
{
    if(!strcmp(argv[0], "quit"))  //如果命令是quit，退出
        exit(0);
    else if(!strcmp(argv[0], "bg") || !strcmp(argv[0], "fg"))  //如果是bg或者fg命令，执行do_fgbg函数 
        do_bgfg(argv);
    else if(!strcmp(argv[0], "jobs"))  //如果命令是jobs，列出正在运行和停止的后台作业
        listjobs(jobs);
    else
        return 0;     /* not a builtin command */
    return 1;
}
```

# do_bgfg

指导里面告诉我们，这个要求是把目标进程变成后台(bg)/前台(fg)，识别方式，参数可以PID也可以是JID
首先查参数向量表，第一个参数是给builtin_cmd的，第二个参数就是给命令执行函数的，如果没有直接返回报错就行

```
void do_bgfg(char **argv)
{
    int num;
    struct job_t *job;
    // 没有参数的fg/bg应该被丢弃
    if(!argv[1]){  //命令行为空
        printf("%s command requires PID or %%jobid argument\n", argv[0]);
        return ;
    }
    return;
}
```

由于既可以JID当参数又可以PID当参数，就要区分开，直接把参数的第一个字符拿出来检查一下就行了，然后根据PID/JID找到对应的进程组，没找到就报错

```
    if(argv[1][0] == '%'){  //解析jid
        if((num = strtol(&argv[1][1], NULL, 10)) <= 0){
            printf("%s: argument must be a PID or %%jobid\n",argv[0]);//失败,打印错误消息
            return;
        }
        if((job = getjobjid(jobs, num)) == NULL){
            printf("%%%d: No such job\n", num); //没找到对应的job 
            return;
        }
    } else {
        if((num = strtol(argv[1], NULL, 10)) <= 0){
            printf("%s: argument must be a PID or %%jobid\n",argv[0]);//失败,打印错误消息
            return;
        }
        if((job = getjobpid(jobs, num)) == NULL){
            printf("(%d): No such process\n", num);  //没找到对应的进程 
            return;
        }
    }
```

如果能找到，就恢复fg/bg，通过修改状态和向进程组发送信号SIGCONT，注意是负数，因为负数表示PPID

```
if(!strcmp(argv[0], "bg")){
        // bg会启动子进程，并将其放置于后台执行
        job->state = BG;  //设置状态 
        if(kill(-job->pid, SIGCONT) < 0)  //采用负数发送信号到进程组 
            unix_error("kill error");
        printf("[%d] (%d) %s", job->jid, job->pid, job->cmdline);
    } else if(!strcmp(argv[0], "fg")) {
        job->state = FG;  //设置状态 
        if(kill(-job->pid, SIGCONT) < 0)  //采用负数发送信号到进程组 
            unix_error("kill error");
        // 当一个进程被设置为前台执行时，当前tsh应该等待该子进程结束
        waitfg(job->pid);
    } else {
        puts("do_bgfg: Internal error");
        exit(0);
    }
```

# waitfg

就是前台一直睡到成为后台为止，换言之tsh在当前前台变后台之前应该是sleep的状态，无限循环就行，每次循环的时候检查一下状态

```
void waitfg(pid_t pid)
{
    struct job_t *job = getjobpid(jobs, pid);
    if(!job) return;

    // 如果当前子进程的状态没有发生改变，则tsh继续休眠
    while(job->state == FG)
        // 使用sleep的这段代码会比较慢，最好使用sigsuspend
        sleep(1);

    return;
}
```

# sigchld_handler

这个函数的目标是非常明确的，就是要我们处理一个信号，也就是SIGCHILD信号，当子进程**终止/停止**的时候，就会向父进程发送这个信号要求父进程来处理，这个回收一般就是waitpid这个系统调用实现的

书上有使用循环配合waitpid无序回收所有子进程的案例，这里同样可以用，于是就使用的是while，waitpid的返回值是子进程的pid，如果没有东西可以返回了就退出循环，而循环体应该是对当次回收子进程的分析

```
void sigchld_handler(int sig)
{
    int status, jid;
    pid_t pid;
    struct job_t *job;

    if(verbose)
        puts("sigchld_handler: entering");

    /*
    以非阻塞方式等待所有子进程
    waitpid 参数3：
        1.     0     ： 执行waitpid时， 只有在子进程 终止 时才会返回。
        2. WNOHANG   : 若子进程仍然在运行，则返回0 。
                注意只有设置了这个标志，waitpid才有可能返回0
        3. WUNTRACED : 如果子进程由于传递信号而停止，则马上返回。
                只有设置了这个标志，waitpid返回时，其WIFSTOPPED(status)才有可能返回true
    */
    while((pid = waitpid(-1, &status, WNOHANG | WUNTRACED)) > 0){

        // 如果当前这个子进程的job已经删除了，则表示有错误发生
        if((job = getjobpid(jobs, pid)) == NULL){
            printf("Lost track of (%d)\n", pid);
            return;
        }

        jid = job->jid;
        //接下来判断三种状态 
        // 如果这个子进程收到了一个暂停信号（还没退出） 
        if(WIFSTOPPED(status)){
            printf("Job [%d] (%d) stopped by signal %d\n", jid, job->pid, WSTOPSIG(status));
            job->state = ST;  //状态设为挂起 
        }
        // 如果子进程通过调用 exit 或者一个返回 (return) 正常终止
        else if(WIFEXITED(status)){
            if(deletejob(jobs, pid))
                if(verbose){
                    printf("sigchld_handler: Job [%d] (%d) deleted\n", jid, pid);
                    printf("sigchld_handler: Job [%d] (%d) terminates OK (status %d)\n", jid, pid, WEXITSTATUS(status));
                }
        }
        // 如果子进程是因为一个未被捕获的信号终止的，例如SIGKILL
        else {
            if(deletejob(jobs, pid)){  //清除进程
                if(verbose)
                    printf("sigchld_handler: Job [%d] (%d) deleted\n", jid, pid);
            }
            printf("Job [%d] (%d) terminated by signal %d\n", jid, pid, WTERMSIG(status));  //返回导致子进程终止的信号的数量
        }
    }

    if(verbose)
        puts("sigchld_handler: exiting");

    return;
}
```

- WIFSTOPPED：：引起返回的子进程当前是被停止的
- WIFSIGNALED：子进程是因为一个未被捕获的信号终止
- WIFEXITED：子进程通过调用exit 或者return正常终止

# sigint_handler

函数fgpid返回前台进程pid

kill函数发送SIGINT信号给前台进程组

kill函数如果返回值为-1，如果返回值为-1表示进程不存在。输出error

```
void sigint_handler(int sig)
{
    if(verbose)
        puts("sigint_handler: entering");
    pid_t pid = fgpid(jobs);

    if(pid){
        // 发送SIGINT给前台进程组里的所有进程
        // 需要注意的是，前台进程组内的进程除了当前前台进程以外，还包括前台进程的子进程。
        // 最多只能存在一个前台进程，但前台进程组内可以存在多个进程
        if(kill(-pid, SIGINT) < 0)
            unix_error("kill (sigint) error");
        if(verbose){
            printf("sigint_handler: Job (%d) killed\n", pid);
        }
    }
    if(verbose)
        puts("sigint_handler: exiting");
    return;
}
```

# sigtstp_handler

SIGTSPT信号默认行为是停止直到下一个 SIGCONT

然后发送SIGTSPT信号到这个前台进程组中的**每个**进程。在默认情况下，结果是停止或挂起前台作业。

fgpid(jobs)获取前台进程pid

kill(-pid,sig)函数发送SIGTSPT信号给前台进程组

```
void sigtstp_handler(int sig)
{
    if(verbose)
        puts("sigstp_handler: entering");

    pid_t pid = fgpid(jobs);
    struct job_t *job = getjobpid(jobs, pid);

    if(pid){
        if(kill(-pid, SIGTSTP) < 0)
            unix_error("kill (tstp) error");
        if(verbose){
            printf("sigstp_handler: Job [%d] (%d) stopped\n", job->jid, pid);
        }
    }
    if(verbose)
        puts("sigstp_handler: exiting");
    return;
}
```

# 检测文件

```
#
# trace15.txt - Putting it all together
#

/bin/echo tsh> ./bogus
./bogus//开子进程，并在子进程里前台执行bogus，找不到bogus，将返回no found

/bin/echo tsh> ./myspin 10
./myspin 10//开子进程，并在子进程里面前台执行myspin 10，前台执行什么都看不见

SLEEP 2
INT//发送INT信号，此时被终止的myspin 10会返回一行提示自己被终止

/bin/echo -e tsh> ./myspin 3 \046
./myspin 3 &//开子进程，并在子进程里面后台执行myspin 3，会有个后台提示

/bin/echo -e tsh> ./myspin 4 \046
./myspin 4 &//开子进程，并在子进程里面后台执行myspin 3，会有个后台提示

/bin/echo tsh> jobs
jobs//查看所有工作，应该会有两个后台的工作运行中，分别是myspin3，myspin4

/bin/echo tsh> fg %1
fg %1//将整个一号集激活并前台运行

SLEEP 2
TSTP//挂起所有前台工作，也就是一号工作集被挂起

/bin/echo tsh> jobs
jobs//此时查看所有工作，应该一停止一运行

/bin/echo tsh> bg %3
bg %3//将三号工作激活并放倒后台运行，由于没有三号工作，返回no found

/bin/echo tsh> bg %1
bg %1//将一号工作激活并放到后台运行，也就是原来挂起状态的一号工作放到后台

/bin/echo tsh> jobs
jobs//此时查看应该是两个工作都在运行

/bin/echo tsh> fg %1
fg %1//将一号工作激活并转到前台

/bin/echo tsh> quit
quit//退出


```
