# Data Lab Notes

bits.c中包含13道编程题。 我们需要使用顺序执行的代码（即无循环或条件）以及有限数量的C算术符和逻辑运算符来完成每道编程题。 具体来说，只允许使用以下八个运算符：

 !  ~ & ^  |  +  <<   >>    

某些题目进一步限制了使用条件。同时不允许使用任何大于8位的常量。 有关详细规则和所需编码风格的讨论，可以参见bits.c中的注释。



###  1.bitXor

```c
/* 
 * bitXor - x^y using only ~ and & 
 *   Example: bitXor(4, 5) = 1
 *   Legal ops: ~ &
 *   Max ops: 14
 *   Rating: 1
 */

int bitXor(int x, int y)
{
  return ~(x & y) & ~(~x & ~y);
}
```

要求利用与、非实现异或（相同得0，不同得1）

也就是 01 、10 取1  ； 00、11 取0。因此可以首先选择 x & y =0 的情况（ 01、10、00），再从中剔除00的情况（~x & ~y），两个条件同时满足即可。



### 2.tmin

```c
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void)
{
  return 1 << 31;
}
```

要求返回二进制补码的最小值。



【知识点补充】

在CSAPP中提到，为了表示负数，最常见的有符号数的计算机表示方式就是补码（two's complement）形式。

最高有效位为负权（negative weight），1为负， 0为正。用于表示计算过程中是否将**最高有效位**的计算添加负号，而其余位始终为正

例：

1011 =  -2^3    +   0*2^2  + 2^1 + 2^0

0011 = +0\*2^3  +  0\*2^2 + 2^1 + 2^0



因此最小的二进制补码为最高位置1，其余置0；同理最大值为最高位置0，其余为1



### 3.isTmax

```c
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise 
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 */
int isTmax(int x)
{
  return !(~(x ^ (x + 1))) & !!(x+1);
}
```

要求判断x是否为最大的二进制补码。

最大的二进制补码为0111 1111 1111 1111 1111 1111  1111 1111 ，即除第31位，其余皆置1。

存在特征： +1 之后得到 第31位置1，其余置0 的情况，与原本数据每一位相反，异或之后即可得到全1。 可利用 !(~(x ^ (x + 1))) 进行检测。 但 32位全置1 也满足以上特征，可以用 !(x+1）进行排除，因为全1的数据+1后会溢出得0



### 4.allOddBits

```c
/* 
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   where bits are numbered from 0 (least significant) to 31 (most significant)
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 */
int allOddBits(int x)
{
  int mask = 0xaa + (0xaa << 8) + (0xaa << 16) + (0xaa << 24);
  int y = mask & x;
  return !(y ^ mask);
}
```

要求判断x的奇数位上是否全部置1

构造奇数位为1，偶数位为0的掩码，x与掩码做与运算，将会得到掩码本身

判断两数是否一致，可以通过异或是否得到全0来判断。





### 5.negate

```c
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x)
{
  return ~x + 1;
}
```

要求返回x的相反数。取反+1即可





### 6.isAsciiDigit

```c
/* 
 * isAsciiDigit - return 1 if 0x30 <= x <= 0x39 (ASCII codes for characters '0' to '9')
 *   Example: isAsciiDigit(0x35) = 1.
 *            isAsciiDigit(0x3a) = 0.
 *            isAsciiDigit(0x05) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 3
 */
int isAsciiDigit(int x)
{
	return  !((x + ~0x30 + 1) >> 31) & !!((x + ~0x3a + 1) >> 31);
}
```

要求判断x是否在Ascii码表中在0-9范围内（0x30 <= x <= 0x39）

存在特征： 与范围边界相减，判断符号位。 当x在范围内时，x减去0x30符号位必为0， x减去0x3a（0x39+1）符号位必为1。题目要求不能使用- ，即可使用negate中取反+1得到相反数的做法代替减法。 