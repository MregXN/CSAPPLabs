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

要求返回

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

```c
/* howManyBits - return the minimum number of bits required to represent x in
 *             two's complement
 *  Examples: howManyBits(12) = 5
 *            howManyBits(298) = 10
 *            howManyBits(-5) = 4
 *            howManyBits(0)  = 1
 *            howManyBits(-1) = 1
 *            howManyBits(0x80000000) = 32
 *  Legal ops: ! ~ & ^ | + << >>
 *  Max ops: 90
 *  Rating: 4
 */
int howManyBits(int x) 
{
  int b16,b8,b4,b2,b1,b0;
  int mask = x >> 31;
  x = (mask & ~x) | (~mask & x); //如果为正数，保持不变；如果为负数，按位取反

  //step1:判断高16为是否有1
  b16 = !!(x >> 16) << 4; //如果高16为有1,则b16 = 16，否则为0
  x >>= b16; //如果高16位有1,x右移16位舍弃低16位,在新的低16位继续查找；否则保持不变
  //step2:判断高8位是否有1
  b8 = !!(x >> 8) << 3;
  x >>= b8;
  //step3:高4位
  b4 = !!(x >> 4) << 2;
  x >>= b4;
  //step4:高2位
  b2 = !!(x >> 2) << 1;
  x >>= b2;
  //step5:高1位
  b1 = !!(x >> 1);
  x >>= b1;
  //step6:低1位
  b0 = x;

  return b16 + b8 + b4 + b2 + b1 + b0 + 1;
}
```

要求返回x用补码表示时的最少位数。

因为当不限定位数时，同一个数可以有不止一种表示方式。比如+12，原码表示可以是 0 1010 ，也可以是 00 1010；比如-3， 原码表示可以是 101 （ -4 +1 = 3） ，也可以是1101 （-8 + 4 + 1 = 3） ， 因此需要我们找出最简短的表示方式。

可以发现规律：当为正数时，同一个数不同的原码表示方法只是在高位不断补0 ， 而当为负数时，则为在高位不断补1。因此可以判断x的正负， 负数转换成正数，之后进一步判断最高的1出现的位置，加一即可得到题目所需的结果。



## 11.floatScale2

```C
/* 
 * floatScale2 - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned floatScale2(unsigned uf)
{
  unsigned sign = (uf & 0x80000000);
  unsigned exp = (uf & 0x7f800000) >> 23;
  unsigned frac = ((uf << 9) >> 9);
  if (!exp)
  { //exp == 0
    frac = frac + frac;
    return sign | frac;
  }
  if (!(exp ^ 0xff))
  { // exp is all 1
    return uf;
  }
  if (!(exp ^ 0xfe))
  {
    return sign | 0x7f800000;
  }
  return (((exp + 1) << 23) | sign) | frac;
}
```

输入无符号数uf，输出2*uf的单精度浮点数表示，可以使用所有的运算符以及条件、循环语句。

_【知识点补充】_

_单精度数，0-22位为frac部分(小数字段)，表示 1.frac 或 0.frac 的数字。23-30位为exp部分，为该浮点数的阶码。  31位为该浮点数的符号位s_

_根据 IEEE浮点数标准，需要分成三种情况来表示浮点数：_

- _exp既全不为0，也全不为1（规格化） ，此时结果为 （-1）^s   *    1.f    *  2^(e-127)_

- _exp全0（非规格化），此时结果为 （-1）^s   *    0.f    *  2^(-126)_

- _exp全1（特殊值），当f全0时表示无穷大， f不为全0时表示NaN_

题目要求返回2*uf。针对规格化的情况，exp部分+1即可满足要求；针对非规格化的情况，frac部分\*2即可满足要求；针对特殊值，返回原数即可满足要求。但需要注意的是，规格化情况下若exp加到了全1的情况时应返回无穷大，即frac部分需另外清0。



## 12.floatFloat2Int

```C
/* 
 * floatFloat2Int - Return bit-level equivalent of expression (int) f
 *   for floating point argument f.
 *   Argument is passed as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point value.
 *   Anything out of range (including NaN and infinity) should return
 *   0x80000000u.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
int floatFloat2Int(unsigned uf)
{
  unsigned sign = (uf & 0x80000000);
  unsigned exp = (uf & 0x7f800000) >> 23;
  unsigned frac = (1 << 23) | ((uf << 9) >> 9);
  int E = exp - 127;
  int val;
  if (exp < 127)
    return 0;
  if (exp >= 158)
    return 0x80000000;
  if (E <= 23)
  {
    val = (frac >> (23 - E));
    return !sign ? val : -val;
  }
  val = (frac << (E - 23));
  return !sign ? val : -val;
}
```

输入无符号数uf，输出uf的int型表示，即需要输出uf的整数部分。

_【知识点补充】_

_单精度数，0-22位为frac部分(小数字段)，表示 1.frac 或 0.frac 的数字。23-30位为exp部分，为该浮点数的阶码。  31位为该浮点数的符号位s_

_根据 IEEE浮点数标准，需要分成三种情况来表示浮点数：_

- _exp既全不为0，也全不为1（规格化） ，此时结果为 （-1）^s   *    1.f    *  2^(e-127)_

- _exp全0（非规格化），此时结果为 （-1）^s   *    0.f    *  2^(-126)_

- _exp全1（特殊值），当f全0时表示无穷大， f不为全0时表示NaN_

根据上方标准，可以得出以下规律：

1. exp <127时， uf小于1，可直接输出0
2. exp >=158 时，已经超过了整形能够表示的最大值，可直接输出0x80000000
3. 因为frac共23位，exp对frac的作用相当于左移右移，因此可以根据exp-127与23的关系对frac进行左移或右移 

最后考虑上符号位，分别输出即可。



## 13.floatPower2

```C
/* 
 * floatPower2 - Return bit-level equivalent of the expression 2.0^x
 *   (2.0 raised to the power x) for any 32-bit integer x.
 *
 *   The unsigned value that is returned should have the identical bit
 *   representation as the single-precision floating-point number 2.0^x.
 *   If the result is too small to be represented as a denorm, return
 *   0. If too large, return +INF.
 * 
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. Also if, while 
 *   Max ops: 30 
 *   Rating: 4
 */
unsigned floatPower2(int x)
{
  if (x > 127)
    return (0xFF << 23);
  if (x < -149)
    return 0u;
  if (x >= -126)
  {
    unsigned exp = x + 127;
    return exp << 23;
  }
    
  int frac = x + 148;
  return 1 << frac;
}
```

要求输入整形变量x，输出2的x次方的浮点数表示。

与上一题类似，同样可以根据取值范围得到以下规律：

1. 当x>127时，该操作已经超出了能够表示的最大值，输出int的最大值
2. 当x<-149时，输出0
3. 对于中间的部分：倘若x > -126, 将exp部分指相应位可表示，否则则在frac部分置相应位，但需要注意的是此情况下因为exp部分为全0而被另外乘以的2的-126次方（非规格化的表示方法），因此需要补上148 ( 126+22 )保证结果正确。