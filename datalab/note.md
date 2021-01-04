# Data Lab Notes

bits.c中包含13道编程题。 我们需要使用顺序执行的代码（即无循环或条件）以及有限数量的C算术符和逻辑运算符来完成每道编程题。  有关详细规则和所需编码风格的讨论，可以参见bits.c中的注释。



##  1.bitXor

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



## 2.tmin

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

_【知识点补充】_

_在CSAPP中提到，为了表示负数，最常见的有符号数的计算机表示方式就是补码（two's complement）形式。_

_最高有效位为负权（negative weight），1为负， 0为正。用于表示计算过程中是否将**最高有效位**的计算添加负号，而其余位始终为正_

_例：_

_1011 =  -2^3    +   0*2^2  + 2^1 + 2^0_

_0011 = +0\*2^3  +  0\*2^2 + 2^1 + 2^0_

因此最小的二进制补码为最高位置1，其余置0；同理最大值为最高位置0，其余为1



## 3.isTmax

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



## 4.allOddBits

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





## 5.negate

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





## 6.isAsciiDigit

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

存在特征： 与范围边界相减，判断符号位，当x在范围内时，x减去0x30符号位必为0， x减去0x3a（0x39+1）符号位必为1。题目要求不能使用- ，即可使用5.negate中取反+1得到相反数的做法代替减法。 



## 7. conditional

```c
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
int conditional(int x, int y, int z)
{
  return ((!x << 31 >> 31) & z) | ((!!x << 31 >> 31) & y);
}
```

要求判断x是否为0，是则输出z，否则输出y。

_【知识点补充1】_

_C语言中，左移指将所有位向左移动若干位，丢弃移出位，低位补0；_

_右移类似，区别在于补位的机制： 逻辑右移直接补0，算数右移补符号位数值（正数补0，负数补1）。_

_因此在C中，逻辑左移与算数左移完全相同，但逻辑右移与算数右移存在差别。_

思路：根据x状态构造32位全0或全1的数据，分别与y、z做按位与运算以完成题目要求的选择功能。 利用叹号或双叹号使x称为0或1，左移至符号位（ <<31 ）后算数右移至最低位( >>31 )，即可构造出全0或全1的数据.





## 8.isLessOrEqual

```c
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y)
{
  int sy = (y >> 31);
  int signSame = ~(sy ^ (x >> 31)); //same is all 1, diff is all 0

  return  (signSame & !!((x + ~(y+1) + 1) >> 31)) || (~signSame & !sy);
}
```

要求判断x是否小于等于y。

思路可以借鉴6.isAsciiDigit，根据x-y的符号位判断x与y的关系。但需要注意当x与y符号不相同时溢出可能会产生错误结果，因此首先判断x、y符号，符号相同时再进一步通过相减判断大小关系。

因此，首先利用signSame表示x、y的符号关系，全1时表示符号相同，全0时表示符号相反。 当符号相同时，x-（y+1）符号位为负时输出1；当符号位不同时，y为正数时输出1。这两个条件相或即可。



## 9.logicalNeg

```c
/* 
 * logicalNeg - implement the ! operator, using all of 
 *              the legal operators except !
 *   Examples: logicalNeg(3) = 0, logicalNeg(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
int logicalNeg(int x)
{
	x |= x>>16; // 若高16位有1，则传递给低16位的对应位
	x |= x>>8;	// 若低16位的高8位有1，则传递给低8位的对应位
	x |= x>>4;	// 若低8位的高4位有1，则传递给低4位的对应位
	x |= x>>2;	// 若低4位的高2位有1，则传递给低2位的对应位
	x |= x>>1;	// 若低2位的高1位有1，则传递给最低1位
	x ^= 1; 	// 只要x包含1，则必定会导致此时的x为1，x^=1即取反
	return x&1; 
}
```

要求利用其余所有操作符实现逻辑非。

_【知识点补充】_

_逻辑非使非0数得0, 0得1_

思路：判断x中是否存在1。



## 10.howManyBits