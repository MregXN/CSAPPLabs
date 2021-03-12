# Bomb Lab Notes

Bomb，炸弹。在这次实验中需要输入六个正确的字符串，如若错误，程序则会提示__“BOOM!!!”__。因此，我们需要将实验提供的二进制文件反汇编，使用调试器调试该汇编程序来猜测正确的字符串。

### 文件结构

```
bomb
├── bomb
├── bomb.c
├── note.md
└── README
```

bomb：可执行的炸弹

bomb.c :  bomb文件的源文件，通过该文件可以知悉程序的执行流程

README： 关于此实验的说明

note.md： 这份笔记



### 反汇编

使用以下指令产生bomb的反汇编程序，并重定向到bomb.asm方便查看

```shell
objdump -d bomb > bomb.asm
```

可以得到类似下方的代码段

```assembly
0000000000400da0 <main>:
  400da0:	53                   	push   %rbx
  400da1:	83 ff 01             	cmp    $0x1,%edi
  400da4:	75 10                	jne    400db6 <main+0x16>
  400da6:	48 8b 05 9b 29 20 00 	mov    0x20299b(%rip),%rax        # 603748 <stdin@@GLIBC_2.2.5>
  400dad:	48 89 05 b4 29 20 00 	mov    %rax,0x2029b4(%rip)        # 603768 <infile>
  400db4:	eb 63                	jmp    400e19 <main+0x79>
  400db6:	48 89 f3             	mov    %rsi,%rbx
  400db9:	83 ff 02             	cmp    $0x2,%edi
  400dbc:	75 3a                	jne    400df8 <main+0x58>
  400dbe:	48 8b 7e 08          	mov    0x8(%rsi),%rdi
  400dc2:	be b4 22 40 00       	mov    $0x4022b4,%esi
  400dc7:	e8 44 fe ff ff       	callq  400c10 <fopen@plt>
  400dcc:	48 89 05 95 29 20 00 	mov    %rax,0x202995(%rip)        # 603768 <infile>
```

以第7行为例， 

__400dad:__  表示地址偏移量  

__48 89 05 b4 29 20 00 __表示运行的机器码 

__mov    %rax,0x2029b4(%rip)  __表示汇编指令 

__# 603768 <infile>__  是代码注释



### phase_1

```assembly
0000000000400ee0 <phase_1>:
  400ee0:	48 83 ec 08          	sub    $0x8,%rsp
  400ee4:	be 00 24 40 00       	mov    $0x402400,%esi
  400ee9:	e8 4a 04 00 00       	callq  401338 <strings_not_equal>
  400eee:	85 c0                	test   %eax,%eax
  400ef0:	74 05                	je     400ef7 <phase_1+0x17>
  400ef2:	e8 43 05 00 00       	callq  40143a <explode_bomb>
  400ef7:	48 83 c4 08          	add    $0x8,%rsp
  400efb:	c3                   	retq   
```

先的阅读一下代码： 将rsp（栈指针）减8，向esi寄存器填入0x402400，之后调用位于401338的strings_not_equal函数，test用于判断eax与上eax的值，与的结果为0则跳过调用explode_bomb这一步，之后栈指针加回8，返回。 其中esi为rsi的低32位，是由调用者自行保存的寄存器；eax为rax的低32位，用于保存返回值。

因此可以进行猜测，strings_not_equal是用于检测输入字符串和目标字符串是否相等的函数，相等时返回0，即可跳过炸弹的引爆。目前可以看出目标字符串被存放在rsi中，而直接查看strings_not_equal函数的代码，可以发现用于与rsi作比较的寄存器，最源头的名字是rdi。 为了验证猜测，以下是GDB加断点调试的部分。



使用以下指令进入gdb

```shell
gdb bomb    
```

在phase_1中跳转到字符串比较前加入断点

```
b strings_not_equal
```

运行程序并输入任意字符串

```
run
mregxn
```

在断点处停止后，查看rdi寄存器的值

```
x /s $rdi
```

可以看到我们之前任意输入的字符串mregxn，此时再打印rsi，即可看到第一个炸弹的答案:Border relations with Canada have never been better.

```
x /s $rsi
```



### phase_2

```assembly
0000000000400efc <phase_2>:
  400efc:	55                   	push   %rbp
  400efd:	53                   	push   %rbx
  400efe:	48 83 ec 28          	sub    $0x28,%rsp
  400f02:	48 89 e6             	mov    %rsp,%rsi
  400f05:	e8 52 05 00 00       	callq  40145c <read_six_numbers>
  400f0a:	83 3c 24 01          	cmpl   $0x1,(%rsp)
  400f0e:	74 20                	je     400f30 <phase_2+0x34>
  400f10:	e8 25 05 00 00       	callq  40143a <explode_bomb>
  400f15:	eb 19                	jmp    400f30 <phase_2+0x34>
  400f17:	8b 43 fc             	mov    -0x4(%rbx),%eax
  400f1a:	01 c0                	add    %eax,%eax
  400f1c:	39 03                	cmp    %eax,(%rbx)
  400f1e:	74 05                	je     400f25 <phase_2+0x29>
  400f20:	e8 15 05 00 00       	callq  40143a <explode_bomb>
  400f25:	48 83 c3 04          	add    $0x4,%rbx
  400f29:	48 39 eb             	cmp    %rbp,%rbx
  400f2c:	75 e9                	jne    400f17 <phase_2+0x1b>
  400f2e:	eb 0c                	jmp    400f3c <phase_2+0x40>
  400f30:	48 8d 5c 24 04       	lea    0x4(%rsp),%rbx
  400f35:	48 8d 6c 24 18       	lea    0x18(%rsp),%rbp
  400f3a:	eb db                	jmp    400f17 <phase_2+0x1b>
  400f3c:	48 83 c4 28          	add    $0x28,%rsp
  400f40:	5b                   	pop    %rbx
  400f41:	5d                   	pop    %rbp
  400f42:	c3                   	retq 
```

先阅读一下代码：先将rbp、rbx入栈保存，然后将栈指针减少（分配内存）。 之后将栈指针作为第2个参数传入rsi，以此调用read_six_numbers函数。之后将栈指针指向头两个字与1比较，如果不相等则引爆炸弹。之后跳转到400f30处，将栈地址后4位以及16+8 = 24位开始的1字节数据分别移动到rbx及rbp，之后跳转到400f17处。此处首先是一个基址+偏移寻址，-0x4(%rbx) 指rbx内所存的地址再往前倒4个字节的内存内所存的值，将它*2后与rbx内所存地址所指向的值比较是否相等，不相等则引爆炸弹。之后跳转至400f25，比较rbx+4后是否与rbp相等，相等则跳至400f3c收尾，否则就到400f17出再次进行如上的比较。

想要弄懂以上的操作的话，需要先弄清楚经过read_six_numbers函数以后，栈指针rsp与我们的输入字符串的关系。

```
000000000040145c <read_six_numbers>:
  40145c:	48 83 ec 18          	sub    $0x18,%rsp
  401460:	48 89 f2             	mov    %rsi,%rdx
  401463:	48 8d 4e 04          	lea    0x4(%rsi),%rcx
  401467:	48 8d 46 14          	lea    0x14(%rsi),%rax
  40146b:	48 89 44 24 08       	mov    %rax,0x8(%rsp)
  401470:	48 8d 46 10          	lea    0x10(%rsi),%rax
  401474:	48 89 04 24          	mov    %rax,(%rsp)
  401478:	4c 8d 4e 0c          	lea    0xc(%rsi),%r9
  40147c:	4c 8d 46 08          	lea    0x8(%rsi),%r8
  401480:	be c3 25 40 00       	mov    $0x4025c3,%esi
  401485:	b8 00 00 00 00       	mov    $0x0,%eax
  40148a:	e8 61 f7 ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  40148f:	83 f8 05             	cmp    $0x5,%eax
  401492:	7f 05                	jg     401499 <read_six_numbers+0x3d>
  401494:	e8 a1 ff ff ff       	callq  40143a <explode_bomb>
  401499:	48 83 c4 18          	add    $0x18,%rsp
  40149d:	c3                   	retq   
```

阅读该函数前，我们首先需要知道，rsi内存的是进入函数前的栈指针，rsp是当前的栈指针，rdi存着我们录入的字符串地址。

read_six_numbers函数中，栈空间申请和指针备份以后有几步明显的lea指令用于从rsi读取数据，包括：rsi处数据读入rdx，rsi+0x4位置数据读入rcx ；rsi+0x14位置读入rax再读入栈；rsi+0x10位置数据读入rax再读入栈；rsi+0xc位置数据读入r9，rsi+0x8 数据读入r8。这六次读取的位置恰好是从rsi开始每隔4个字节的位置。 完成数据提取之后，之后调用__isoc99_sscanf读取数据入栈。 

因此可以猜测，该函数首先对当前栈相应位置的数据做备份，之后在相应位置上利用scanf填入了六个数字。

使用以下指令进入gdb

```shell
gdb bomb    
```

在phase_2中调用后加入断点

```
b *0x400f0a
```

在输入任意六个数字 1 2 3 4 5 6 之后分别打印rsp内的字符，可以看到正是我们输入的六个数字

```
x /s $rsp
x /s $rsp + 4
x /s $rsp + 8
x /s $rsp + 12
x /s $rsp + 16
x /s $rsp + 20
```

根据在phase_2中的循环比较，$rsp处的数字应为1，后面每一位需要是前一位的2倍，因此答案是:1 2 4 8 16 32



### phase_3

```assembly
0000000000400f43 <phase_3>:
  400f43:	48 83 ec 18          	sub    $0x18,%rsp
  400f47:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx
  400f4c:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx
  400f51:	be cf 25 40 00       	mov    $0x4025cf,%esi
  400f56:	b8 00 00 00 00       	mov    $0x0,%eax
  400f5b:	e8 90 fc ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  400f60:	83 f8 01             	cmp    $0x1,%eax
  400f63:	7f 05                	jg     400f6a <phase_3+0x27>
  400f65:	e8 d0 04 00 00       	callq  40143a <explode_bomb>

  400f6a:	83 7c 24 08 07       	cmpl   $0x7,0x8(%rsp)
  400f6f:	77 3c                	ja     400fad <phase_3+0x6a>
  400f71:	8b 44 24 08          	mov    0x8(%rsp),%eax
  400f75:	ff 24 c5 70 24 40 00 	jmpq   *0x402470(,%rax,8)
  
  400f7c:	b8 cf 00 00 00       	mov    $0xcf,%eax
  400f81:	eb 3b                	jmp    400fbe <phase_3+0x7b>
  400f83:	b8 c3 02 00 00       	mov    $0x2c3,%eax
  400f88:	eb 34                	jmp    400fbe <phase_3+0x7b>
  400f8a:	b8 00 01 00 00       	mov    $0x100,%eax
  400f8f:	eb 2d                	jmp    400fbe <phase_3+0x7b>
  400f91:	b8 85 01 00 00       	mov    $0x185,%eax
  400f96:	eb 26                	jmp    400fbe <phase_3+0x7b>
  400f98:	b8 ce 00 00 00       	mov    $0xce,%eax
  400f9d:	eb 1f                	jmp    400fbe <phase_3+0x7b>
  400f9f:	b8 aa 02 00 00       	mov    $0x2aa,%eax
  400fa4:	eb 18                	jmp    400fbe <phase_3+0x7b>
  400fa6:	b8 47 01 00 00       	mov    $0x147,%eax
  400fab:	eb 11                	jmp    400fbe <phase_3+0x7b>

  400fad:	e8 88 04 00 00       	callq  40143a <explode_bomb>
  400fb2:	b8 00 00 00 00       	mov    $0x0,%eax
  400fb7:	eb 05                	jmp    400fbe <phase_3+0x7b>
  400fb9:	b8 37 01 00 00       	mov    $0x137,%eax

  400fbe:	3b 44 24 0c          	cmp    0xc(%rsp),%eax
  400fc2:	74 05                	je     400fc9 <phase_3+0x86>
  400fc4:	e8 71 04 00 00       	callq  40143a <explode_bomb>

  400fc9:	48 83 c4 18          	add    $0x18,%rsp
  400fcd:	c3                   	retq    
```

有了前两次的经验，这次可以一边阅读代码一边配合gdb调试验证了

代码内首先申请了栈空间， 之后调用了scanf函数读取用户输入的字符串，这里可以利用gdb断点打印0x4025cf处的数据，查看系统期望得到的数据格式。

```shell
b *0x400f5b        //在scanf处加断点
x /s 0x4025cf      // 打印%esi
```

打印结果为： 

0x4025cf:	"%d %d"   ，可见这次需要的到字符串是两个整形数字。

scanf完成后如果返回值正常，则跳转到400f6a处。 此处比较了0x8(%rsp)与7大小，如果该数据大于7则引爆炸弹。这里先验证0x8(%rsp)与输入字符串的关系。

在输入任意两个整形数据后（例如 1 2）后，执行以下指令

```shell
b *0x400f6a      // 在当前位置加入断点
x  $rsp+8        // 打印需要比较的数据
x  $rsp+0xc      // 打印需要比较的数据
```

打印结果分别为：

0x7fffffffe358:	"\001"

0x7fffffffe35c:	"\002"

可以知道0x8(%rsp) 与 0xc(%rsp) 指的是我们录入的两个数字。到这里可以得知我们录入的第一个数据，应该小于等于7。

完成与7的比较之后，代码将第一个数据存入了eax，eax为rax的低32位。接下来利用了这个数据进行了一次比例变址寻址。

```
400f75:	ff 24 c5 70 24 40 00 	jmpq   *0x402470(,%rax,8)
```

这里访问的是0x402470 + %rax\*8 这个地址。而%rax内，存的是我们第一个数据，第一个数据可能为0-7中的数字，因此我们尝试打印看不同的输入数据会跳转到何处

```shell
x  *0x402470           //0
x  *(0x402470+8)       //1
x  *(0x402470+16)      //2
x  *(0x402470+24)      //3
x  *(0x402470+32)      //4
x  *(0x402470+40)      //5
x  *(0x402470+48)      //6
x  *(0x402470+56)      //7
```

打印结果分别如下

0x400f7c <phase_3+57>:	"\270", <incomplete sequence \317>

0x400fb9 <phase_3+118>:	"\270\067\001"

0x400f83 <phase_3+64>:	"\270\303\002"

...

后面我就不一一展示了，打印结果指明了不同的第一位会跳转的位置。查看代码可以发现，这8个跳转的去向代码都写的差不多:

```
mov    不同值,%eax
jmp    400fbe <phase_3+0x7b>
...
400fbe:	3b 44 24 0c          	cmp    0xc(%rsp),%eax
400fc2:	74 05                	je     400fc9 <phase_3+0x86>
400fc4:	e8 71 04 00 00       	callq  40143a <explode_bomb>
```

朝%eax内写入不同的值，然后统一跳转到400fbe处跟第二个录入的值进行比较，不相等则引爆炸弹。

到这里答案就出来了，根据你输入的第一个值，程序会判断第二个值是否跟预期的值相等，相等则拆除成功。 而根据代码中写入eax的值，可以将八组答案列出来，任选一组输入即可：

0 207

1 311

2 707

3 256

4 389

5 206

6 682

7 327



### phase_4

```assembly
000000000040100c <phase_4>:  
  40100c:	48 83 ec 18          	sub    $0x18,%rsp
  401010:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx
  401015:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx
  40101a:	be cf 25 40 00       	mov    $0x4025cf,%esi
  40101f:	b8 00 00 00 00       	mov    $0x0,%eax
  401024:	e8 c7 fb ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  401029:	83 f8 02             	cmp    $0x2,%eax
  40102c:	75 07                	jne    401035 <phase_4+0x29>
  40102e:	83 7c 24 08 0e       	cmpl   $0xe,0x8(%rsp)
  401033:	76 05                	jbe    40103a <phase_4+0x2e>

  401035:	e8 00 04 00 00       	callq  40143a <explode_bomb>

  40103a:	ba 0e 00 00 00       	mov    $0xe,%edx
  40103f:	be 00 00 00 00       	mov    $0x0,%esi
  401044:	8b 7c 24 08          	mov    0x8(%rsp),%edi
  401048:	e8 81 ff ff ff       	callq  400fce <func4>
  40104d:	85 c0                	test   %eax,%eax
  40104f:	75 07                	jne    401058 <phase_4+0x4c>
  401051:	83 7c 24 0c 00       	cmpl   $0x0,0xc(%rsp)
  401056:	74 05                	je     40105d <phase_4+0x51>

  401058:	e8 dd 03 00 00       	callq  40143a <explode_bomb>
  40105d:	48 83 c4 18          	add    $0x18,%rsp
  401061:	c3                   	retq   
```

前面的代码与之前的部分类似，我们可以直接查看0x4025cf处的数据，知晓此次bomb需要什么类型的数据。

```
b *0x401024        //在scanf处加断点
x /s 0x4025cf     // 打印%esi
```

打印结果如下：0x4025cf:	"%d %d"，说明这次需要的是两个整形数据。

之后则判断了获取的第一个数字是否小于0xe，大于的话则引爆炸弹。

接下来把0xe赋给edx、0x0赋给esi，rsp+0x8的值赋给edi，然后调用了func4函数。而这个函数的返回值必须为0，否则则引爆炸弹。然后判断了第二位数字，只有为0才能不引爆炸弹。 有了以上的基础之后，我们再来看func4内发生的事情。

```assembly
0000000000400fce <func4>:
  400fce:	48 83 ec 08          	sub    $0x8,%rsp
  400fd2:	89 d0                	mov    %edx,%eax
  400fd4:	29 f0                	sub    %esi,%eax
  400fd6:	89 c1                	mov    %eax,%ecx
  400fd8:	c1 e9 1f             	shr    $0x1f,%ecx
  400fdb:	01 c8                	add    %ecx,%eax
  400fdd:	d1 f8                	sar    %eax
  400fdf:	8d 0c 30             	lea    (%rax,%rsi,1),%ecx
  400fe2:	39 f9                	cmp    %edi,%ecx
  400fe4:	7e 0c                	jle    400ff2 <func4+0x24>
  400fe6:	8d 51 ff             	lea    -0x1(%rcx),%edx
  400fe9:	e8 e0 ff ff ff       	callq  400fce <func4>
  400fee:	01 c0                	add    %eax,%eax
  400ff0:	eb 15                	jmp    401007 <func4+0x39>

  400ff2:	b8 00 00 00 00       	mov    $0x0,%eax
  400ff7:	39 f9                	cmp    %edi,%ecx
  400ff9:	7d 0c                	jge    401007 <func4+0x39>

  400ffb:	8d 71 01             	lea    0x1(%rcx),%esi
  400ffe:	e8 cb ff ff ff       	callq  400fce <func4>
  401003:	8d 44 00 01          	lea    0x1(%rax,%rax,1),%eax
  
  401007:	48 83 c4 08          	add    $0x8,%rsp
  40100b:	c3                   	retq   
```

...

7 0

### phase_5

```powershell
0000000000401062 <phase_5>:
  401062:	53                   	push   %rbx
  401063:	48 83 ec 20          	sub    $0x20,%rsp
  401067:	48 89 fb             	mov    %rdi,%rbx
  40106a:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax
  401071:	00 00 
  401073:	48 89 44 24 18       	mov    %rax,0x18(%rsp)
  401078:	31 c0                	xor    %eax,%eax
  40107a:	e8 9c 02 00 00       	callq  40131b <string_length>
  40107f:	83 f8 06             	cmp    $0x6,%eax
  401082:	74 4e                	je     4010d2 <phase_5+0x70>
  401084:	e8 b1 03 00 00       	callq  40143a <explode_bomb>
  401089:	eb 47                	jmp    4010d2 <phase_5+0x70>

  40108b:	0f b6 0c 03          	movzbl (%rbx,%rax,1),%ecx
  40108f:	88 0c 24             	mov    %cl,(%rsp)
  401092:	48 8b 14 24          	mov    (%rsp),%rdx
  401096:	83 e2 0f             	and    $0xf,%edx
  401099:	0f b6 92 b0 24 40 00 	movzbl 0x4024b0(%rdx),%edx
  4010a0:	88 54 04 10          	mov    %dl,0x10(%rsp,%rax,1)
  4010a4:	48 83 c0 01          	add    $0x1,%rax
  4010a8:	48 83 f8 06          	cmp    $0x6,%rax
      
  4010ac:	75 dd                	jne    40108b <phase_5+0x29>
  4010ae:	c6 44 24 16 00       	movb   $0x0,0x16(%rsp)
  4010b3:	be 5e 24 40 00       	mov    $0x40245e,%esi
  4010b8:	48 8d 7c 24 10       	lea    0x10(%rsp),%rdi
  4010bd:	e8 76 02 00 00       	callq  401338 <strings_not_equal>
  4010c2:	85 c0                	test   %eax,%eax
  4010c4:	74 13                	je     4010d9 <phase_5+0x77>
  4010c6:	e8 6f 03 00 00       	callq  40143a <explode_bomb>
  4010cb:	0f 1f 44 00 00       	nopl   0x0(%rax,%rax,1)
  4010d0:	eb 07                	jmp    4010d9 <phase_5+0x77>

  4010d2:	b8 00 00 00 00       	mov    $0x0,%eax
  4010d7:	eb b2                	jmp    40108b <phase_5+0x29>
  4010d9:	48 8b 44 24 18       	mov    0x18(%rsp),%rax
  4010de:	64 48 33 04 25 28 00 	xor    %fs:0x28,%rax
  4010e5:	00 00 
  4010e7:	74 05                	je     4010ee <phase_5+0x8c>
  4010e9:	e8 42 fa ff ff       	callq  400b30 <__stack_chk_fail@plt>
  
  4010ee:	48 83 c4 20          	add    $0x20,%rsp
  4010f2:	5b                   	pop    %rbx
  4010f3:	c3                   	retq  
```

401062 到 401089 完成的是堆栈初始化、插入哨兵以及长度检测的工作，根据 ` cmp    $0x6,%eax` 可知我们输入的字符串长度应为6，否则引爆。

接下来进入了一段循环：

```powershell
  40108b:	0f b6 0c 03          	movzbl (%rbx,%rax,1),%ecx
  40108f:	88 0c 24             	mov    %cl,(%rsp)
  401092:	48 8b 14 24          	mov    (%rsp),%rdx
  401096:	83 e2 0f             	and    $0xf,%edx
  401099:	0f b6 92 b0 24 40 00 	movzbl 0x4024b0(%rdx),%edx
  4010a0:	88 54 04 10          	mov    %dl,0x10(%rsp,%rax,1)
  4010a4:	48 83 c0 01          	add    $0x1,%rax
  4010a8:	48 83 f8 06          	cmp    $0x6,%rax
  4010ac:	75 dd                	jne    40108b <phase_5+0x29>
```

40108b 利用 movzbl 将 从rbx开始的第rax个位置的第一个字节赋值给了ecx，接下来将cl的值赋给了rsp再赋给了rdx，这里cl的值即为上一步得到的结果。`movzbl 0x4024b0(%rdx),%edx`将0x4024b0+rdx位置的第一个字节低4位中，并将这4位复制到`rsp+0x10+rax`中，之后rax自加1，跳回开始处继续循环，直至rax自加到6。

以上可以知道rax实际上既充当了循环的计数，也充当了读取字符的索引。这个循环一共进行了6次，记录这个6个字符的低4位，并以这6个值为索引从`0x4024b0+rdx`的位置复制对应的6个值入栈。

```powershell
  4010ae:	c6 44 24 16 00       	movb   $0x0,0x16(%rsp)
  4010b3:	be 5e 24 40 00       	mov    $0x40245e,%esi
  4010b8:	48 8d 7c 24 10       	lea    0x10(%rsp),%rdi
  4010bd:	e8 76 02 00 00       	callq  401338 <strings_not_equal>
  4010c2:	85 c0                	test   %eax,%eax
  4010c4:	74 13                	je     4010d9 <phase_5+0x77>
  4010c6:	e8 6f 03 00 00       	callq  40143a <explode_bomb>
  4010cb:	0f 1f 44 00 00       	nopl   0x0(%rax,%rax,1)
  4010d0:	eb 07                	jmp    4010d9 <phase_5+0x77>
```

在末尾插入终止符后，从0x40245e处拿数据与我们刚才的六个字符比较，不相等则引爆炸弹

理清楚逻辑之后我们首先打印0x40245e处的字符串

```
b *0x4010bd        //在scanf处加断点
x /s 0x40245e     // 打印%esi
```

打印结果为 

```
0x40245e:	"flyers"
```

再打印0x4024b0处的字符串

```
x /s 0x4024b0 
0x4024b0 <array.3449>:	"maduiersnfotvbylSo you think you can stop the bomb with ctrl-c, do you?"
```

要从这个字符串中索引到flyers，说明我们输入的六个字符的低16位应该分别为

``` 
十进制表示：        9  15  14  5   6   7 
低四位十六进制表示：  9   f   e   5   6   7
```

查ascii表，可以得到符合条件的答案

如 IONEFG



### phase_6

```powershell
00000000004010f4 <phase_6>:
  4010f4:	41 56                	push   %r14
  4010f6:	41 55                	push   %r13
  4010f8:	41 54                	push   %r12
  4010fa:	55                   	push   %rbp
  4010fb:	53                   	push   %rbx
  4010fc:	48 83 ec 50          	sub    $0x50,%rsp
  401100:	49 89 e5             	mov    %rsp,%r13
  401103:	48 89 e6             	mov    %rsp,%rsi
  401106:	e8 51 03 00 00       	callq  40145c <read_six_numbers>
  40110b:	49 89 e6             	mov    %rsp,%r14
  40110e:	41 bc 00 00 00 00    	mov    $0x0,%r12d
  401114:	4c 89 ed             	mov    %r13,%rbp
  401117:	41 8b 45 00          	mov    0x0(%r13),%eax
  40111b:	83 e8 01             	sub    $0x1,%eax
  40111e:	83 f8 05             	cmp    $0x5,%eax
  401121:	76 05                	jbe    401128 <phase_6+0x34>
  401123:	e8 12 03 00 00       	callq  40143a <explode_bomb>
  401128:	41 83 c4 01          	add    $0x1,%r12d
  40112c:	41 83 fc 06          	cmp    $0x6,%r12d
  401130:	74 21                	je     401153 <phase_6+0x5f>
  401132:	44 89 e3             	mov    %r12d,%ebx
  401135:	48 63 c3             	movslq %ebx,%rax
  401138:	8b 04 84             	mov    (%rsp,%rax,4),%eax
  40113b:	39 45 00             	cmp    %eax,0x0(%rbp)
  40113e:	75 05                	jne    401145 <phase_6+0x51>
  401140:	e8 f5 02 00 00       	callq  40143a <explode_bomb>
  401145:	83 c3 01             	add    $0x1,%ebx
  401148:	83 fb 05             	cmp    $0x5,%ebx
  40114b:	7e e8                	jle    401135 <phase_6+0x41>
  40114d:	49 83 c5 04          	add    $0x4,%r13
  401151:	eb c1                	jmp    401114 <phase_6+0x20>
  401153:	48 8d 74 24 18       	lea    0x18(%rsp),%rsi
  401158:	4c 89 f0             	mov    %r14,%rax
  40115b:	b9 07 00 00 00       	mov    $0x7,%ecx
  401160:	89 ca                	mov    %ecx,%edx
  401162:	2b 10                	sub    (%rax),%edx
  401164:	89 10                	mov    %edx,(%rax)
  401166:	48 83 c0 04          	add    $0x4,%rax
  40116a:	48 39 f0             	cmp    %rsi,%rax
  40116d:	75 f1                	jne    401160 <phase_6+0x6c>
  40116f:	be 00 00 00 00       	mov    $0x0,%esi
  401174:	eb 21                	jmp    401197 <phase_6+0xa3>
  401176:	48 8b 52 08          	mov    0x8(%rdx),%rdx
  40117a:	83 c0 01             	add    $0x1,%eax
  40117d:	39 c8                	cmp    %ecx,%eax
  40117f:	75 f5                	jne    401176 <phase_6+0x82>
  401181:	eb 05                	jmp    401188 <phase_6+0x94>
  401183:	ba d0 32 60 00       	mov    $0x6032d0,%edx
  401188:	48 89 54 74 20       	mov    %rdx,0x20(%rsp,%rsi,2)
  40118d:	48 83 c6 04          	add    $0x4,%rsi
  401191:	48 83 fe 18          	cmp    $0x18,%rsi
  401195:	74 14                	je     4011ab <phase_6+0xb7>
  401197:	8b 0c 34             	mov    (%rsp,%rsi,1),%ecx
  40119a:	83 f9 01             	cmp    $0x1,%ecx
  40119d:	7e e4                	jle    401183 <phase_6+0x8f>
  40119f:	b8 01 00 00 00       	mov    $0x1,%eax
  4011a4:	ba d0 32 60 00       	mov    $0x6032d0,%edx
  4011a9:	eb cb                	jmp    401176 <phase_6+0x82>
  4011ab:	48 8b 5c 24 20       	mov    0x20(%rsp),%rbx
  4011b0:	48 8d 44 24 28       	lea    0x28(%rsp),%rax
  4011b5:	48 8d 74 24 50       	lea    0x50(%rsp),%rsi
  4011ba:	48 89 d9             	mov    %rbx,%rcx
  4011bd:	48 8b 10             	mov    (%rax),%rdx
  4011c0:	48 89 51 08          	mov    %rdx,0x8(%rcx)
  4011c4:	48 83 c0 08          	add    $0x8,%rax
  4011c8:	48 39 f0             	cmp    %rsi,%rax
  4011cb:	74 05                	je     4011d2 <phase_6+0xde>
  4011cd:	48 89 d1             	mov    %rdx,%rcx
  4011d0:	eb eb                	jmp    4011bd <phase_6+0xc9>
  4011d2:	48 c7 42 08 00 00 00 	movq   $0x0,0x8(%rdx)
  4011d9:	00 
  4011da:	bd 05 00 00 00       	mov    $0x5,%ebp
  4011df:	48 8b 43 08          	mov    0x8(%rbx),%rax
  4011e3:	8b 00                	mov    (%rax),%eax
  4011e5:	39 03                	cmp    %eax,(%rbx)
  4011e7:	7d 05                	jge    4011ee <phase_6+0xfa>
  4011e9:	e8 4c 02 00 00       	callq  40143a <explode_bomb>
  4011ee:	48 8b 5b 08          	mov    0x8(%rbx),%rbx
  4011f2:	83 ed 01             	sub    $0x1,%ebp
  4011f5:	75 e8                	jne    4011df <phase_6+0xeb>
  4011f7:	48 83 c4 50          	add    $0x50,%rsp
  4011fb:	5b                   	pop    %rbx
  4011fc:	5d                   	pop    %rbp
  4011fd:	41 5c                	pop    %r12
  4011ff:	41 5d                	pop    %r13
  401201:	41 5e                	pop    %r14
  401203:	c3                   	retq  
```

...