# shell-lab

##  预先说明

### 博客推荐

我个人认为考虑的比较周到的博客：https://zhuanlan.zhihu.com/p/119034923

对于信号阻塞比较到位的github代码：https://github.com/BlackDragonF/CSAPPLabs.git

### 相关资料

csapp书的说明：http://csapp.cs.cmu.edu/3e/shlab.pdf

cmu复习：https://link.zhihu.com/?target=http%3A//www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/recitations/rec09.pdf

### 我的说明

对于这个实验来说，我认为更多的是去观察各个博客之间的一些不同，对于一些博客而言并没有充分理解所有函数，同时关于这篇文章也有一个没有实质代码但是有解决方案的问题(主要是对于内部一些实现不是很好去做)，但是我认为我这篇博客可以说是十分详尽了，但是我只是大致有思路，代码多数是抄我推荐的第一个博客的。



## 正文

1. eval函数

   ```c
   void eval(char *cmdline)
   {
       char * argv[MAXARGS];                                //命令行参数
       int bg_falg;                                         //判断是否为后台
       sigset_t mask_one, prev_mask, mask_all;                 
       pid_t pid;
   
       bg_flag = parseline(cmdline, argv);                  //首先通过parseline去判断是否为后台， 同时将cmdline->argv
       
       if(!builtin_cmd(argv)){                              //如果是非内置命令则继续
           sigemptyset(&mask_one);
           sigaddset(&mask_one, SIGCHLD);
   
           sigpromask(SIG_BLOCK, &mask_one, &prev_mask);    //阻塞SIGCHLD信号
           if(pid = fork() == 0){                           //生成子进程
               setpgid(0,0);                                //将生成的子进程放在一个单独的进程组
               sigpromask(SIG_SETMASK, &prev_mask, NULL);  
               if(execve(argv[0], argv, environ)<0){        //执行文件 如果不成功则退出并输出错误
                   printf("%s: Command Not Found\n", argv[0]);
                   exit(0);
               }
           }
   
           sigfillset(&mask_all);                           
           sigpromask(SIG_BLOCK, &mask_all, NULL);
           addjob(jobs, pid, bg_falg?BG:FG, cmdline);       
           sigpromask(SIG_SETMASK, &prev_mask, NULL);        //调用addjob时阻塞所有信号  防止deletejob与addjob竞争 
   
           if(bg_falg){
               printf("[%d](%d)&s",pid2jid(pid), pid, cmdline);
           }else{
               waitfg();                                     //如果是后台的则输出，否则等待子进程发出SIGCHLD信号后再退出
           }
       }
   }
   ```

   说明：对于eval函数主要是读取命令行然后执行，但是如果是内置命令则会通过`builtin_cmd`函数，同时再[shell-lab.pdf](http://csapp.cs.cmu.edu/3e/shlab.pdf)中也有建议

   > In eval, the parent must use sigprocmask to block SIGCHLD signals before it forks the child, and then unblock these signals, again using sigprocmask after it adds the child to the job list by calling addjob. Since children inherit the blocked vectors of their parents, the child must be sure to then unblock SIGCHLD signals before it execs the new program. 
   >
   > //大意来说就是要在fork之前阻塞SIGCHLD信号，然后再执行时放开， 让后再阻塞所有信号进行addjob()

   对于这个整体是没有太大难度的就是要注意对于非内置命令要用一个新的进程组，因为比如当前job重启时，如果不是新的进程组则需要一个一个遍历去给信号。

   >Here is the workaround: After the fork, but before the execve, the child process should call setpgid(0, 0), which puts the child in a new process group whose group ID is identical to the child’s PID. This ensures that there will be only one process, your shell, in the foreground process group. When you type ctrl-c, the shell should catch the resulting SIGINT and then forward it to the appropriate foreground job (or more precisely, the process group that contains the foreground job).

2. builtin_cmd

   ```c
   int builtin_cmd(char **argv)
   {
       char *first_arg = argv[0];
       if(first_arg==NULL){                 //如果只是打了给空格则说明没有事情发生
           return 1;
       }
   
       if(!strcmp(first_arg, "quit")){      //退出
           exit(0);
       }
   
       if(strcmp(first_arg, "jobs")){       //列出所有job
           listjobs(jobs);
           return 1;
       }
   
       if(!strcmp(first_arg, "bg") || !strcmp(first_arg, "fg")){   
           do_bgfg(argv);
           return 1;
       }
       return 0;     /* not a builtin command */
   }
   ```

   >四个内置函数
   >
   >- quit  退出
   >- jobs  列出job
   >- bg    The bg  command restarts  by sending it a SIGCONT signal, and then runs it in the background. The  argument can be either a PID or a JID.
   >- fg    The fg  command restarts  by sending it a SIGCONT signal, and then runs it in the foreground. The  argument can be either a PID or a JID.

3. do_fgbg

   ```c
   void do_bgfg(char **argv)
   {
       char *first_argv = argv[0];
       struct job_t *job;
       int id;
       sigset_t mask, prev_mask;
   
   
       if (argv[1] != NULL){
               printf("%s comment need PID or %%jobid argument \n", argv[0]);
       }
   
       if(sscanf(argv[1], "%%%d", &id)){
           job = getjobjid(jobs, id);
           if(job == NULL || job->state == UNDEF){
               printf("%%%d: No such job\n", id);
               return;
           }
       }else if(sscanf(argv[1], "%d", &id)){
           job = getjobpid(jobs, id);
           if(job == NULL || job->state == UNDEF){
               printf("%d: No such PID\n", id);
           }
       }else{
           printf("%s: argument must be a PID or %%%jobid\n",argv[0]);
           return;
       }                                                                 //通过PID或是jid查询job
   
   
       sigfillset(&mask);
       sigpromask(SIG_BLOCK, &mask, &prev_mask);
       if(!strcmp(argv[0], "fg")){
           job->state = FG;
       }else{
           job->state = BG;
       }
       sigpromask(SIG_SETMASK, &prev_mask, NULL);                      //阻塞信号修改job
   
       kill(-(job->pid), SIGCONT);                                     //发送   
       if(!strcmp(argv[0], "fg")){
           waitfg(job->pid);                                            //前台的等待完成
       }else{
           printf("[%d] (%d) %s",job->jid, job->pid, job->cmdline);
       }
   
   }
   
   ```

   最主要的就是通过`%1(jid)`或`1(PID)`得到当前的`struct job_t`从而修改`job->state`和发送信号。

4. waitfg

   ```c
   void waitfg(pid_t pid)
   {
       while(pid = fgpid(jobs)){
           sleep(1);
       }
   }
   ```

   >– In waitfg, use a busy loop around the sleep function.

5. SIGINT和SIGNSTOP信号处理函数

   ```c
   void sigint_handler(int sig)
   {
       int old_errno = errno;
       pid_t pid = fgpid(jobs);
       if(pid != 0){
           kill(-pid, sig);
       }
       erron = old_errno;
   }
   
   void sigtstp_handler(int sig)
   {
       int old_errno = errno;
       pid_t pid = fgpid(jobs);
       if(pid != 0){
           kill(-pid, sig);
       }
       erron = old_errno;
   }
   ```

6. SIGCHLD处理函数

   ```c
   void sigchld_handler(int sig)
   {
       int old_errno = errno;
       pid_t pid;
       sigset_t mask, prev_mask;
       int statue;
       struct job_t *job;
   
       sigfillset(&mask);
       while((pid = waitpid(-1, &statue, WNOHANG|WUNTRACED)) >0){                 
           sigpromask(SIG_BLOCK, &mask, &prev_mask);
           if(WIFEXITED(statue)){
               deletejob(jobs, pid);
           }else if(WIFSIGNLED(statue)){
               printf("Job [%d] (%d) terminated by signal %d\n", pid2jid(pid), pid, WTERMSIG(statue));
               deletejob(jobs, pid);
           }else if(WIFSTOPPED(statue)){
               job = getjobpid(jobs, pid);
               job->state = ST;
               printf("Job [%d] (%d) stopped by signal %d\n", job->jid, pid, WSTOPSIG(state));
           }
           signpromask(SIG_SETMASK, &prev_mask, NULL);
       }
       errno = old_errno;
   }
   ```

   这个函数其实比较写起来比较简单，但是对于waitpid的几个参数要十分熟悉。



## 几个问题

这里我说一些我遇到的比较难理解的问题

1. 为什么子进程会再收到SIGINT和SIGSTOP后会返回SIGCHLD信号？

   >这是因为我们所设置的对于`SIGINT`和`SIGSTOP`的处理只有在父进程和子进程未`exev`前时是通过我们所写的处理的，当exev时会覆盖原先的虚拟内存，从而使得所以`exev`会将原先设置为要捕捉的信号都更改为默认动作。因此终止和停止进程，并向父进程发送SIGCHLD信号。    关于exev可以在第九章看待

2. 内置命令，运行期间，如果通过ctrl+c触发sigint信号处理，那他kill会给谁呢？

    >现象：这里会产生一个循环会持续的发送SIGINT信号，也就是如果我让内置命令的时间加长点，并在运行期间ctrl+c则会发生递归调用sigint信号处理程序，也就是kill发送sigint信号又被处理程序捕捉，又调用kill。
    >
    >解决思路：通过c语言的非本地跳转即这点可以参考bash的处理方法，比如我们执行 `read line`,然后再ctrl + c这个时候 `read line` 这个内置命令(read)会被终止，所以内置命令的实现就可以采用进入内置命令时设置singal handle 为一个siglongjmp，并在内置命令中用sigsetjmp处理。