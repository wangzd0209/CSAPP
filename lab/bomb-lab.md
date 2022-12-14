# bomb-lab

## 预备说明

对于bomb-lab，我认为和重要的一点就是要看gdb操作和汇编操作，同时bomb-lab对于理解汇编以及后续lab有很大的帮助

gdb操作和汇编：https://pan.baidu.com/s/12yAV-Vn9wTDPvmphs5uvQw?pwd=nfux

### 说明

1. 一定要耐心的理解汇编，同时对于一些不理解的可以直接看答案

2. 知乎上的一些bomb-lab的博客写的很好，

   博客：https://zhuanlan.zhihu.com/p/48759303
   
3. 答案 只有1，2，6的答案是唯一的

   >Border relations with Canada have never been better.
   >1 2 4 8 16 32
   >0 207  
   >7 0  
   >ionefg
   >4 3 2 1 6 5

## 正文

将汇编输入到out.txt文件，或者如果你对于gdb有一定了解也可以直接查询phase_1函数，下面两种任选一种

```shell
objdump -d bomb > out.txt  //输出到out.txt
gdb bomb 
```

运行bomb,可以在gdb中直接执行run，或是通过`./bomb`运行发现我们要输入字符串才能运行，同时根据提示发现我们要执行6个字符串

### phase_1 

1. 直接调出phase_1函数的汇编，可以在gdb模式下执行`disas phase_1` 

   ```shell
   Dump of assembler code for function phase_1:
      0x0000000000400ee0 <+0>:		sub    $0x8,%rsp
      0x0000000000400ee4 <+4>:		mov    $0x402400,%esi
      0x0000000000400ee9 <+9>:		callq  0x401338 <strings_not_equal>
      0x0000000000400eee <+14>:	test   %eax,%eax
      0x0000000000400ef0 <+16>:	je     0x400ef7 <phase_1+23>
      0x0000000000400ef2 <+18>:	callq  0x40143a <explode_bomb>
      0x0000000000400ef7 <+23>:	add    $0x8,%rsp
      0x0000000000400efb <+27>:	retq   
   ```

2. 我们发现整体的代码就是调用名为`string_not_equal`的函数，检验返回寄存器`%rax`是否为0，如果是则正常跳出，否则炸弹爆炸，转换为c文件大概就是

   ```c
   int phase_1(char[] string){
       char[] equalString = ...                          //正好是0x402400为首地址的一组字符串
       int flag = string_not_equal(string, equalString); //flag就是返回寄存器的值
       if (flag){
           printf("炸弹爆炸")
       }
   }
   ```

3. 因此我们可以得出第一部分的字符串,在gdb模式下在命令行输入`x/s 0x402400`

   ```
   "Border relations with Canada have never been better."
   ```

但是我们学习并不能仅限于此，因此让我们研究一下`string_not_equal`函数，汇编为

```shell
Dump of assembler code for function strings_not_equal:
   0x0000000000401338 <+0>:		push   %r12   
   0x000000000040133a <+2>:		push   %rbp
   0x000000000040133b <+3>:		push   %rbx           //被调用者保存
   0x000000000040133c <+4>:		mov    %rdi,%rbx      
   0x000000000040133f <+7>:		mov    %rsi,%rbp
   0x0000000000401342 <+10>:	callq  0x40131b <string_length> //获取输入的字符串长度放到%r12d中
   0x0000000000401347 <+15>:	mov    %eax,%r12d  
   0x000000000040134a <+18>:	mov    %rbp,%rdi                
   0x000000000040134d <+21>:	callq  0x40131b <string_length> //获取比较字符串长度
   0x0000000000401352 <+26>:	mov    $0x1,%edx
   0x0000000000401357 <+31>:	cmp    %eax,%r12d               //比较长度，不一样长则返回非0
   0x000000000040135a <+34>:	jne    0x40139b <strings_not_equal+99>
   0x000000000040135c <+36>:	movzbl (%rbx),%eax              //将字符串保存在%eax中
   0x000000000040135f <+39>:	test   %al,%al                  //如果为零则说明比较结束，退出并返回0
   0x0000000000401361 <+41>:	je     0x401388 <strings_not_equal+80>
   0x0000000000401363 <+43>:	cmp    0x0(%rbp),%al            如果不为零则比较第一个字符是否相等，如果不等则退出并返回1
   0x0000000000401366 <+46>:	je     0x401372 <strings_not_equal+58>
   0x0000000000401368 <+48>:	jmp    0x40138f <strings_not_equal+87>
   0x000000000040136a <+50>:	cmp    0x0(%rbp),%al
   0x000000000040136d <+53>:	nopl   (%rax)
   0x0000000000401370 <+56>:	jne    0x401396 <strings_not_equal+94>
   0x0000000000401372 <+58>:	add    $0x1,%rbx                 //两边的字符加一
   0x0000000000401376 <+62>:	add    $0x1,%rbp
   0x000000000040137a <+66>:	movzbl (%rbx),%eax               //在将输入字符串放入%edx中
   0x000000000040137d <+69>:	test   %al,%al                   //记录是否为零，不为零则执行比较循环否则退出，并返回0
   0x000000000040137f <+71>:	jne    0x40136a <strings_not_equal+50>
   0x0000000000401381 <+73>:	mov    $0x0,%edx
   0x0000000000401386 <+78>:	jmp    0x40139b <strings_not_equal+99>
   0x0000000000401388 <+80>:	mov    $0x0,%edx
   0x000000000040138d <+85>:	jmp    0x40139b <strings_not_equal+99>
   0x000000000040138f <+87>:	mov    $0x1,%edx
   0x0000000000401394 <+92>:	jmp    0x40139b <strings_not_equal+99>
   0x0000000000401396 <+94>:	mov    $0x1,%edx
   0x000000000040139b <+99>:	mov    %edx,%eax
   0x000000000040139d <+101>:	pop    %rbx
   0x000000000040139e <+102>:	pop    %rbp
   0x000000000040139f <+103>:	pop    %r12
   0x00000000004013a1 <+105>:	retq   
```

这是我们又发现调用了一个`string_length`的函数，汇编如下：

```shell
Dump of assembler code for function string_length:
0x000000000040131b <+0>:		cmpb   $0x0,(%rdi) //判断字符串是否为00，因为对于字符串00就代表结束了
0x000000000040131e <+3>:		je     0x401332 <string_length+23> //如果为零，说明字符串为空，则字长为零，退出函数
0x0000000000401320 <+5>:		mov    %rdi,%rdx  //如果不为零就检验下一个字符是否为零，如果为零则退出，不为零则递归操作
0x0000000000401323 <+8>:		add    $0x1,%rdx
0x0000000000401327 <+12>:	mov    %edx,%eax
0x0000000000401329 <+14>:	sub    %edi,%eax
0x000000000040132b <+16>:	cmpb   $0x0,(%rdx)
0x000000000040132e <+19>:	jne    0x401323 <string_length+8>
0x0000000000401330 <+21>:	repz retq 
0x0000000000401332 <+23>:	mov    $0x0,%eax
0x0000000000401337 <+28>:	retq   
End of assembler dump.
```

然后让我们在用c语言写下大致的代码

```c
int string_length(char[] string){
 if(string == 0){
     return 0;
 }
 string = a;
 a = a + 1;
 if(a == 0){
     return;
 }
 string_length(a);
}
```

### phase_2

1. 输出phase_2的汇编

   ```shell
   Dump of assembler code for function phase_2:
      0x0000000000400efc <+0>:		push   %rbp
      0x0000000000400efd <+1>:		push   %rbx
      0x0000000000400efe <+2>:		sub    $0x28,%rsp
      0x0000000000400f02 <+6>:		mov    %rsp,%rsi
      0x0000000000400f05 <+9>:		callq  0x40145c <read_six_numbers>  //读六个数
      0x0000000000400f0a <+14>:	cmpl   $0x1,(%rsp)                  //判断第一个是否为为1
      0x0000000000400f0e <+18>:	je     0x400f30 <phase_2+52>        //不为1则爆炸
      0x0000000000400f10 <+20>:	callq  0x40143a <explode_bomb>
      0x0000000000400f15 <+25>:	jmp    0x400f30 <phase_2+52>
      0x0000000000400f17 <+27>:	mov    -0x4(%rbx),%eax              //读取前一个数
      0x0000000000400f1a <+30>:	add    %eax,%eax                    //前一个数x2
      0x0000000000400f1c <+32>:	cmp    %eax,(%rbx)                  //和当前数比较不对则爆炸
      0x0000000000400f1e <+34>:	je     0x400f25 <phase_2+41>
      0x0000000000400f20 <+36>:	callq  0x40143a <explode_bomb>   
      0x0000000000400f25 <+41>:	add    $0x4,%rbx                    //指向下一个数
      0x0000000000400f29 <+45>:	cmp    %rbp,%rbx                    //判断是否是最后一个数不是则循环否则退出
      0x0000000000400f2c <+48>:	jne    0x400f17 <phase_2+27>
      0x0000000000400f2e <+50>:	jmp    0x400f3c <phase_2+64>
      0x0000000000400f30 <+52>:	lea    0x4(%rsp),%rbx                //将第二个数的地址放在%rdx
      0x0000000000400f35 <+57>:	lea    0x18(%rsp),%rbp               //将第最后一个数的地址放在%rdp
      0x0000000000400f3a <+62>:	jmp    0x400f17 <phase_2+27>
      0x0000000000400f3c <+64>:	add    $0x28,%rsp
      0x0000000000400f40 <+68>:	pop    %rbx
      0x0000000000400f41 <+69>:	pop    %rbp
      0x0000000000400f42 <+70>:	retq   
   End of assembler dump.
   ```

2. 我们发现我们读取六个数，一次我们这次的字符串就是6个数，同时我们发现这六个数的规律是第一个数为1，后一个数是前一个数x2，因此字符串为

   ```
   1 2 4 8 16 32
   ```

同时学习么，要跟深入，让我们看一下`read_six_numbers`函数的汇编

```shell
Dump of assembler code for function read_six_numbers:
   0x000000000040145c <+0>:		sub    $0x18,%rsp
   0x0000000000401460 <+4>:		mov    %rsi,%rdx
   0x0000000000401463 <+7>:		lea    0x4(%rsi),%rcx   
   0x0000000000401467 <+11>:	lea    0x14(%rsi),%rax
   0x000000000040146b <+15>:	mov    %rax,0x8(%rsp)
   0x0000000000401470 <+20>:	lea    0x10(%rsi),%rax
   0x0000000000401474 <+24>:	mov    %rax,(%rsp)
   0x0000000000401478 <+28>:	lea    0xc(%rsi),%r9
   0x000000000040147c <+32>:	lea    0x8(%rsi),%r8
   0x0000000000401480 <+36>:	mov    $0x4025c3,%esi   //0x4025c3:	"%d %d %d %d %d %d"
   0x0000000000401485 <+41>:	mov    $0x0,%eax
   0x000000000040148a <+46>:	callq  0x400bf0 <__isoc99_sscanf@plt> //返回输入的个数
   0x000000000040148f <+51>:	cmp    $0x5,%eax          //判断输入的的是否大于5
   0x0000000000401492 <+54>:	jg     0x401499 <read_six_numbers+61>
   0x0000000000401494 <+56>:	callq  0x40143a <explode_bomb>
   0x0000000000401499 <+61>:	add    $0x18,%rsp
   0x000000000040149d <+65>:	retq   
End of assembler dump.
```
我们可以得到寄存器和栈的关系

>%rdx => 0x0(%rsp)
>%rcx => 0x4(%rsp)
>%r8 => 0x8(%rsp)
>%r9 => 0xc(%rsp)
>(%rsp) => 0x10(%rsp)
>0x8(%rsp) => 0x14(%rsp)

### phase_3

1. 输出phase_3的汇编

    ```shell
    Dump of assembler code for function phase_3:
       0x0000000000400f43 <+0>:		sub    $0x18,%rsp
       0x0000000000400f47 <+4>:		lea    0xc(%rsp),%rcx                 //第一个数
       0x0000000000400f4c <+9>:		lea    0x8(%rsp),%rdx                 //第二个数
       0x0000000000400f51 <+14>:	mov    $0x4025cf,%esi                 //0x4025cf:	"%d %d"
       0x0000000000400f56 <+19>:	mov    $0x0,%eax
       0x0000000000400f5b <+24>:	callq  0x400bf0 <__isoc99_sscanf@plt> //输入两个数
       0x0000000000400f60 <+29>:	cmp    $0x1,%eax                      //判断输入数的个数是否大于1，不大于1则爆炸
       0x0000000000400f63 <+32>:	jg     0x400f6a <phase_3+39>
       0x0000000000400f65 <+34>:	callq  0x40143a <explode_bomb>
       0x0000000000400f6a <+39>:	cmpl   $0x7,0x8(%rsp)                 //判断第1个数是否大于7，大于7爆炸
       0x0000000000400f6f <+44>:	ja     0x400fad <phase_3+106>
       0x0000000000400f71 <+46>:	mov    0x8(%rsp),%eax                 //保存第2个数
       0x0000000000400f75 <+50>:	jmpq   *0x402470(,%rax,8)             //简介跳转到0x402470+第二个数x8的位置 
       0x0000000000400f7c <+57>:	mov    $0xcf,%eax                   //例如为第一个数为零，则0x400f7c <phase_3+57>:	"\270"
       0x0000000000400f81 <+62>:	jmp    0x400fbe <phase_3+123>         //得到一个270的数和跳转的地址
       0x0000000000400f83 <+64>:	mov    $0x2c3,%eax
       0x0000000000400f88 <+69>:	jmp    0x400fbe <phase_3+123>
       0x0000000000400f8a <+71>:	mov    $0x100,%eax
       0x0000000000400f8f <+76>:	jmp    0x400fbe <phase_3+123>
       0x0000000000400f91 <+78>:	mov    $0x185,%eax
       0x0000000000400f96 <+83>:	jmp    0x400fbe <phase_3+123>
       0x0000000000400f98 <+85>:	mov    $0xce,%eax
       0x0000000000400f9d <+90>:	jmp    0x400fbe <phase_3+123>
       0x0000000000400f9f <+92>:	mov    $0x2aa,%eax
       0x0000000000400fa4 <+97>:	jmp    0x400fbe <phase_3+123>
       0x0000000000400fa6 <+99>:	mov    $0x147,%eax
       0x0000000000400fab <+104>:	jmp    0x400fbe <phase_3+123>
       0x0000000000400fad <+106>:	callq  0x40143a <explode_bomb>
       0x0000000000400fb2 <+111>:	mov    $0x0,%eax
       0x0000000000400fb7 <+116>:	jmp    0x400fbe <phase_3+123>
       0x0000000000400fb9 <+118>:	mov    $0x137,%eax
       0x0000000000400fbe <+123>:	cmp    0xc(%rsp),%eax                //判断第二个数是否于获取的数相同
       0x0000000000400fc2 <+127>:	je     0x400fc9 <phase_3+134>
       0x0000000000400fc4 <+129>:	callq  0x40143a <explode_bomb>
       0x0000000000400fc9 <+134>:	add    $0x18,%rsp
       0x0000000000400fcd <+138>:	retq   
    End of assembler dump.
    ```

2. 因为第一个数要小于7则有八种可能

   ```
   0 207 ( 0xcf )
   1 311 ( 0x137 )
   2 707 ( 0x2c3 )
   3 256 ( 0x100 )
   4 389 ( 0x185 )
   5 206 ( 0xce )
   6 682 ( 0x2aa )
   7 327 ( 0x147 )
   ```

### phase_4

1. 依旧是先输出汇编

   ```shell
   Dump of assembler code for function phase_4:
      0x000000000040100c <+0>:		sub    $0x18,%rsp
      0x0000000000401010 <+4>:		lea    0xc(%rsp),%rcx
      0x0000000000401015 <+9>:		lea    0x8(%rsp),%rdx
      0x000000000040101a <+14>:	mov    $0x4025cf,%esi                //0x4025cf:	"%d %d"
      0x000000000040101f <+19>:	mov    $0x0,%eax
      0x0000000000401024 <+24>:	callq  0x400bf0 <__isoc99_sscanf@plt>
      0x0000000000401029 <+29>:	cmp    $0x2,%eax                      //同样是输入两个数
      0x000000000040102c <+32>:	jne    0x401035 <phase_4+41>            
      0x000000000040102e <+34>:	cmpl   $0xe,0x8(%rsp)                 //第一个数要小于12
      0x0000000000401033 <+39>:	jbe    0x40103a <phase_4+46>
      0x0000000000401035 <+41>:	callq  0x40143a <explode_bomb>
      0x000000000040103a <+46>:	mov    $0xe,%edx                      //
      0x000000000040103f <+51>:	mov    $0x0,%esi
      0x0000000000401044 <+56>:	mov    0x8(%rsp),%edi
      0x0000000000401048 <+60>:	callq  0x400fce <func4>               //跳转到func4函数  
      0x000000000040104d <+65>:	test   %eax,%eax                      //fun4返回非0则爆炸
      0x000000000040104f <+67>:	jne    0x401058 <phase_4+76>
      0x0000000000401051 <+69>:	cmpl   $0x0,0xc(%rsp)                  //比较第而个数是否为0，不为0则爆炸
      0x0000000000401056 <+74>:	je     0x40105d <phase_4+81>
      0x0000000000401058 <+76>:	callq  0x40143a <explode_bomb>
      0x000000000040105d <+81>:	add    $0x18,%rsp
      0x0000000000401061 <+85>:	retq   
   End of assembler dump.
   ```

2. 这是我们关于func4函数得参数为

   ```shell
      0x000000000040103a <+46>:	mov    $0xe,%edx                      //
      0x000000000040103f <+51>:	mov    $0x0,%esi
      0x0000000000401044 <+56>:	mov    0x8(%rsp),%edi
   ```

3. 然后我们进入func4得汇编

   ```she
   Dump of assembler code for function func4:
      0x0000000000400fce <+0>:		sub    $0x8,%rsp
      0x0000000000400fd2 <+4>:		mov    %edx,%eax
      0x0000000000400fd4 <+6>:		sub    %esi,%eax                    
      0x0000000000400fd6 <+8>:		mov    %eax,%ecx                     
      0x0000000000400fd8 <+10>:	shr    $0x1f,%ecx                    //左移31位
      0x0000000000400fdb <+13>:	add    %ecx,%eax                     
      0x0000000000400fdd <+15>:	sar    %eax                          //逻辑右移一位
      0x0000000000400fdf <+17>:	lea    (%rax,%rsi,1),%ecx           
      0x0000000000400fe2 <+20>:	cmp    %edi,%ecx                     //比较%ecx不大于%edi则跳转
      0x0000000000400fe4 <+22>:	jle    0x400ff2 <func4+36>           
      0x0000000000400fe6 <+24>:	lea    -0x1(%rcx),%edx
      0x0000000000400fe9 <+27>:	callq  0x400fce <func4>
      0x0000000000400fee <+32>:	add    %eax,%eax
      0x0000000000400ff0 <+34>:	jmp    0x401007 <func4+57>
      0x0000000000400ff2 <+36>:	mov    $0x0,%eax                     
      0x0000000000400ff7 <+41>:	cmp    %edi,%ecx                     //比较%edi和%ecx小于则跳转退出
      0x0000000000400ff9 <+43>:	jge    0x401007 <func4+57>
      0x0000000000400ffb <+45>:	lea    0x1(%rcx),%esi
      0x0000000000400ffe <+48>:	callq  0x400fce <func4>
      0x0000000000401003 <+53>:	lea    0x1(%rax,%rax,1),%eax
      0x0000000000401007 <+57>:	add    $0x8,%rsp
      0x000000000040100b <+61>:	retq   
   End of assembler dump.
   ```

   对于func4的可以用c语言写：

   ```c
   void func4(int x,int y,int z)  //y的初始值为0，z的初始值为14,t->%rax,k->%ecx
   {
     int t=z-y;
     int k=t>>31;
     t=(t+k)>>1;
     k=t+y;
     if(k>x)
     {
       z=k-1;
       func4(x,y,z);
       t=2t;
       return;
     }
     else
      {
        t=0;
        if(k<x)
        {
           y=k+1;
           func4(x,y,z);
           t=2*t+1;
           return;
        }
        else
        {
            return;
        }
      }
   }
   ```

   又因为我们预想返回值为1而当进行一次第二个循环则返回值必定不为0

4. 最终我们的得出字符串

   ```
   0 0
   1 0
   3 0
   7 0
   ```

### phase_5

1. 查询汇编

   ```shell
   Dump of assembler code for function phase_5:
      0x0000000000401062 <+0>:		push   %rbx
      0x0000000000401063 <+1>:		sub    $0x20,%rsp
      0x0000000000401067 <+5>:		mov    %rdi,%rbx
      0x000000000040106a <+8>:		mov    %fs:0x28,%rax                  //生成一个金丝雀
      0x0000000000401073 <+17>:	mov    %rax,0x18(%rsp)                
      0x0000000000401078 <+22>:	xor    %eax,%eax                       //按位取^运算,得到00
      0x000000000040107a <+24>:	callq  0x40131b <string_length>
      0x000000000040107f <+29>:	cmp    $0x6,%eax                      //字符串字长为6，否则爆炸
      0x0000000000401082 <+32>:	je     0x4010d2 <phase_5+112> 
      0x0000000000401084 <+34>:	callq  0x40143a <explode_bomb>
      0x0000000000401089 <+39>:	jmp    0x4010d2 <phase_5+112>
      0x000000000040108b <+41>:	movzbl (%rbx,%rax,1),%ecx              //选取第一个字符
      0x000000000040108f <+45>:	mov    %cl,(%rsp)                      //放入内存
      0x0000000000401092 <+48>:	mov    (%rsp),%rdx                    
      0x0000000000401096 <+52>:	and    $0xf,%edx                       //按位取&运算
      0x0000000000401099 <+55>:	movzbl 0x4024b0(%rdx),%edx             //的到一个字符串
      0x00000000004010a0 <+62>:	mov    %dl,0x10(%rsp,%rax,1)           //将第一个字符放到内存中
      0x00000000004010a4 <+66>:	add    $0x1,%rax                       //加1
      0x00000000004010a8 <+70>:	cmp    $0x6,%rax                       //判断是否完成6次，不满足则循环
      0x00000000004010ac <+74>:	jne    0x40108b <phase_5+41>           
      0x00000000004010ae <+76>:	movb   $0x0,0x16(%rsp)                 //字符串00结尾
      0x00000000004010b3 <+81>:	mov    $0x40245e,%esi                  //0x40245e:	"flyers"
      0x00000000004010b8 <+86>:	lea    0x10(%rsp),%rdi
      0x00000000004010bd <+91>:	callq  0x401338 <strings_not_equal>    //比较两个字符串
      0x00000000004010c2 <+96>:	test   %eax,%eax                       //判断返回值是否为0
      0x00000000004010c4 <+98>:	je     0x4010d9 <phase_5+119>          
      0x00000000004010c6 <+100>:	callq  0x40143a <explode_bomb>
      0x00000000004010cb <+105>:	nopl   0x0(%rax,%rax,1)
      0x00000000004010d0 <+110>:	jmp    0x4010d9 <phase_5+119>
      0x00000000004010d2 <+112>:	mov    $0x0,%eax                            
      0x00000000004010d7 <+117>:	jmp    0x40108b <phase_5+41>
      0x00000000004010d9 <+119>:	mov    0x18(%rsp),%rax                 //比较金丝雀的值
      0x00000000004010de <+124>:	xor    %fs:0x28,%rax
      0x00000000004010e7 <+133>:	je     0x4010ee <phase_5+140>          //相同则取0然后退出函数
      0x00000000004010e9 <+135>:	callq  0x400b30 <__stack_chk_fail@plt>
      0x00000000004010ee <+140>:	add    $0x20,%rsp
      0x00000000004010f2 <+144>:	pop    %rbx
      0x00000000004010f3 <+145>:	retq   
   End of assembler dump.
   ```

2. 因此我们可以发现我们输入的字符串，首先会与0xf进行和运算，然后将作为索引查询在0x4024b0的字符串，循环6次构成一个新的字符串然后和flyers比较相等则退出

3. 0x4024b0字符串为

   ```
   maduiersnfotvbylSo you think you can stop the bomb with ctrl-c, do you?
   ```

   所以的到索引为

   >9 -- 1001
   >
   >f -- 1111
   >
   >e -- 1110
   >
   >5 -- 0101
   >
   >6 -- 0110
   >
   >7 -- 0111

4. 所以得到一组字符串为

   ```
   ionefg
   ```

### phase_6

1. 汇编代码，由于phase_6十分长所以我们分段进行分析

   ```shell
   Dump of assembler code for function phase_6:
      0x00000000004010f4 <+0>:		push   %r14
      0x00000000004010f6 <+2>:		push   %r13
      0x00000000004010f8 <+4>:		push   %r12
      0x00000000004010fa <+6>:		push   %rbp
      0x00000000004010fb <+7>:		push   %rbx
      0x00000000004010fc <+8>:		sub    $0x50,%rsp
      0x0000000000401100 <+12>:	mov    %rsp,%r13
      0x0000000000401103 <+15>:	mov    %rsp,%rsi
      0x0000000000401106 <+18>:	callq  0x40145c <read_six_numbers>     //读取6个数和phase_6相同
      0x000000000040110b <+23>:	mov    %rsp,%r14                       
      0x000000000040110e <+26>:	mov    $0x0,%r12d                       //计数器
      0x0000000000401114 <+32>:	mov    %r13,%rbp                        //将%rbp存入当前数的地址           
      0x0000000000401117 <+35>:	mov    0x0(%r13),%eax                   //将第一个数放在%eax中
      0x000000000040111b <+39>:	sub    $0x1,%eax                        
      0x000000000040111e <+42>:	cmp    $0x5,%eax                        //得到当前数在1——6之间
      0x0000000000401121 <+45>:	jbe    0x401128 <phase_6+52>            
      0x0000000000401123 <+47>:	callq  0x40143a <explode_bomb>
      0x0000000000401128 <+52>:	add    $0x1,%r12d                        //计数器加一
      0x000000000040112c <+56>:	cmp    $0x6,%r12d                        //判断计数器是否为6，循环完则进入下一阶段
      0x0000000000401130 <+60>:	je     0x401153 <phase_6+95>
      0x0000000000401132 <+62>:	mov    %r12d,%ebx                        
      0x0000000000401135 <+65>:	movslq %ebx,%rax                         
      0x0000000000401138 <+68>:	mov    (%rsp,%rax,4),%eax                //取出下一个数放在%eax中
      0x000000000040113b <+71>:	cmp    %eax,0x0(%rbp)                    //使得下一个数和当前数不相等
      0x000000000040113e <+74>:	jne    0x401145 <phase_6+81>
      0x0000000000401140 <+76>:	callq  0x40143a <explode_bomb>
      0x0000000000401145 <+81>:	add    $0x1,%ebx                          //循环判断剩下的几个数和当前数是否相等
      0x0000000000401148 <+84>:	cmp    $0x5,%ebx
      0x000000000040114b <+87>:	jle    0x401135 <phase_6+65>
      0x000000000040114d <+89>:	add    $0x4,%r13                           //执行下一个数的地址
      0x0000000000401151 <+93>:	jmp    0x401114 <phase_6+32>
   ```

   因此第一部分我们可以的出一共有六个数且每个数不相同且都小于7

2. 第二部分的汇编

   ```shell
      0x0000000000401153 <+95>:	lea    0x18(%rsp),%rsi                //将第六个数的地址放在%rsi中              
      0x0000000000401158 <+100>:	mov    %r14,%rax                      //将第一个数的地址放在%r14中
      0x000000000040115b <+103>:	mov    $0x7,%ecx                      
      0x0000000000401160 <+108>:	mov    %ecx,%edx
      0x0000000000401162 <+110>:	sub    (%rax),%edx                    
      0x0000000000401164 <+112>:	mov    %edx,(%rax)                    //将当前数变为7-当前数
      0x0000000000401166 <+114>:	add    $0x4,%rax                      //指向下一个数的地址  
      0x000000000040116a <+118>:	cmp    %rsi,%rax                      //判断六个数是否都转化完
      0x000000000040116d <+121>:	jne    0x401160 <phase_6+108>  
      0x000000000040116f <+123>:	mov    $0x0,%esi
      0x0000000000401174 <+128>:	jmp    0x401197 <phase_6+163>
   ```

   因此我们可以看到第二部分为如果我们设当前数为a，则进行变换`a = 7-a`, 跳转到下一部分

3. 第三部分的汇编

    ```shell
       0x0000000000401176 <+130>:	mov    0x8(%rdx),%rdx              //对齐，当前数对应到当前节点的+8位
       0x000000000040117a <+134>:	add    $0x1,%eax
       0x000000000040117d <+137>:	cmp    %ecx,%eax
       0x000000000040117f <+139>:	jne    0x401176 <phase_6+130>
       0x0000000000401181 <+141>:	jmp    0x401188 <phase_6+148>
       0x0000000000401183 <+143>:	mov    $0x6032d0,%edx
       0x0000000000401188 <+148>:	mov    %rdx,0x20(%rsp,%rsi,2)     //将节点放入内存，并在+8位放入当前地址
       0x000000000040118d <+153>:	add    $0x4,%rsi                   //进行下一个数
       0x0000000000401191 <+157>:	cmp    $0x18,%rsi
       0x0000000000401195 <+161>:	je     0x4011ab <phase_6+183>
       0x0000000000401197 <+163>:	mov    (%rsp,%rsi,1),%ecx          //将第一个数放在%ecx中
       0x000000000040119a <+166>:	cmp    $0x1,%ecx                   //比较是否大于1
       0x000000000040119d <+169>:	jle    0x401183 <phase_6+143>      
       0x000000000040119f <+171>:	mov    $0x1,%eax                   //大于1
       0x00000000004011a4 <+176>:	mov    $0x6032d0,%edx              
       0x00000000004011a9 <+181>:	jmp    0x401176 <phase_6+130>
    ```

   因此，在这部分就是构建了6个节点
   
   >$0x6032d0的结构形如
   >0x6032d0 <node1>:	"L\001"     
   >0x6032d3 <node1+3>:	""
   >0x6032d4 <node1+4>:	"\001"
   >0x6032d6 <node1+6>:	""
   >0x6032d7 <node1+7>:	""
   >0x6032d8 <node1+8>:	"\340\062`"
   >0x6032dc <node1+12>:	""
   >0x6032dd <node1+13>:	""
   >0x6032de <node1+14>:	""
   >0x6032df <node1+15>:	""

4. 第四部分的汇编

   ```shell
      0x00000000004011ab <+183>:	mov    0x20(%rsp),%rbx            //第一个节点首地址
      0x00000000004011b0 <+188>:	lea    0x28(%rsp),%rax            //指向第二个节点首地址的地址
      0x00000000004011b5 <+193>:	lea    0x50(%rsp),%rsi            //指向最后一个节首地址点的地址
      0x00000000004011ba <+198>:	mov    %rbx,%rcx                  
      0x00000000004011bd <+201>:	mov    (%rax),%rdx                //获取第二个节点的地址
      0x00000000004011c0 <+204>:	mov    %rdx,0x8(%rcx)             //将第二个节点的首地址放入第一个节点的next位置
      0x00000000004011c4 <+208>:	add    $0x8,%rax                  //指向第三个节点的首地址的地址
      0x00000000004011c8 <+212>:	cmp    %rsi,%rax                  //比较是否是指向最后一个地址首地址的地址
      0x00000000004011cb <+215>:	je     0x4011d2 <phase_6+222>  
      0x00000000004011cd <+217>:	mov    %rdx,%rcx                  //循环
      0x00000000004011d0 <+220>:	jmp    0x4011bd <phase_6+201>
   ```

   这部分就是将每个节点连接起来，我建议自己梳理一下三个地址之间的关系。

5. 第5部分

   ```shell
      0x00000000004011d2 <+222>:	movq   $0x0,0x8(%rdx)        //将第六个节点next指向0x00
      0x00000000004011da <+230>:	mov    $0x5,%ebp            
      0x00000000004011df <+235>:	mov    0x8(%rbx),%rax        //获取下一个节点的首地址
      0x00000000004011e3 <+239>:	mov    (%rax),%eax           //获取第二个节点的值
      0x00000000004011e5 <+241>:	cmp    %eax,(%rbx)           //比较当前节点的值，如果当前节点的值小于则爆炸
      0x00000000004011e7 <+243>:	jge    0x4011ee <phase_6+250>  
      0x00000000004011e9 <+245>:	callq  0x40143a <explode_bomb>
      0x00000000004011ee <+250>:	mov    0x8(%rbx),%rbx        //指向下一个节点
      0x00000000004011f2 <+254>:	sub    $0x1,%ebp             //计数器-1
      0x00000000004011f5 <+257>:	jne    0x4011df <phase_6+235>  //判断6个数是否并比较完，比较完则退出
      0x00000000004011f7 <+259>:	add    $0x50,%rsp
      0x00000000004011fb <+263>:	pop    %rbx
      0x00000000004011fc <+264>:	pop    %rbp
      0x00000000004011fd <+265>:	pop    %r12
      0x00000000004011ff <+267>:	pop    %r13
      0x0000000000401201 <+269>:	pop    %r14
      0x0000000000401203 <+271>:	retq   
   End of assembler dump.
   ```

   在这部分我们得出六个节点的顺序是值从大往小

6. 我们输出0x6032d0的六个节点并排序

   ```
   0x6032f0 <node3>:	0x9c	0x03	0x00	0x00	0x03	0x00	0x00	0x00   ==>  039c   03 
   0x603300 <node4>:	0xb3	0x02	0x00	0x00	0x04	0x00	0x00	0x00   ==>  02b3   04
   0x603310 <node5>:	0xdd	0x01	0x00	0x00	0x05	0x00	0x00	0x00   ==>  01dd   05
   0x603320 <node6>:	0xbb	0x01	0x00	0x00	0x06	0x00	0x00	0x00   ==>  01bb   06
   0x6032d0 <node1>:	0x4c	0x01	0x00	0x00	0x01	0x00	0x00	0x00   ==>  014c   01
   0x6032e0 <node2>:	0xa8	0x00	0x00	0x00	0x02	0x00	0x00	0x00   ==>  00a8   02
   ```

7. 因此我们得到数为`3 4 5 6 1 2` 通过第二部分的到我们输入的数

   ```
   4 3 2 1 6 5 
   ```

   
