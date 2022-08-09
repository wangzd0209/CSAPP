# data-lab

## 1 bitXor

要求：只使用`~`和`&`完成`^`运算，翻译使用非，和完成异或运算

```c
/* 
 * bitXor - x^y using only ~ and & 
 *   Example: bitXor(4, 5) = 1
 *   Legal ops: ~ &
 *   Max ops: 14
 *   Rating: 1
 */
int bitXor(int x, int y) {
  return ~((~x)&(~y))&~(x&y);
}
```

思路：使用概率论的概率运算完成

```
x^y=(((~x)&y)|(x&(~y)))  // 定义，因此现在就时将|变为~和&的关系式
同时巧妙使用~(x|y)= (~x)&(~y) 解决
```

## 2 tmin

要求找到最小数(补码形式)

```c
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
  return 0x1<<31;
}
```

## 3 isTmax 

判断一个数是否是最大值(补码形式)

```c
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise 
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 */
int isTmax(int x) {
  int i=x+1;
  x=x+i;
  x=~x;
  i=!i;
  x=x+i;
  return !x;
}
```

思路：`将一个数减去最大值判断是否为0`，这是就转化到如何将减去最大值变为legal中可以操作的，最大值的位数为01
加1后变为最小值10，两个相加再取反就计算出0，但是此时如果为参数为-1的话也会变成0，因此还要判断x+1不等于0即可

**最重要的是解答思路将一个数减去最大值判断是否为0，以及如何合理操作将最大值变为0**



## 4 allOddBits

要求：检验所有奇数位1

```c
/* 
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   where bits are numbered from 0 (least significant) to 31 (most significant)
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 */
int allOddBits(int x) {
  int mask = 0xAA+(0xAA<<8);
  mask = (mask<<16) + mask;
  return !((mask&x)^mask);
}
```

思路：通过掩码的形式，构造一个奇数位都位1的32位数，然后将偶数位清零，再比较奇数位。

(mask&x):清零

(x^mask):比较奇数位

## 5 negate

```c
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
  return ~x+1;
}
```

 思路，我也不知道，我查了资料好像就是一种规律吧。

## 6 isAsciiDigit

要求：判断值是否再0x30和0x39之间

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
int isAsciiDigit(int x) {
  int mask = 0x3<<4;
  int c = 0xf;
  c = ~c;
  int b = !((x&c)^mask);
  x = x + 6;
  int a = !((x&c)^mask);
  return a & b;
}
```

思路：掩码先确定高位是否正确，通过掩码和allOddBits的想法相同，，然后再加上值使得最大值溢出。

## 7 conditional

要求：实行三目运算

```c
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
int conditional(int x, int y, int z) {
  x = !!x;
  x = ~x+1;
  return (x&y)|(~x&z);
}
```

思路：为0将x变为00，非0将x变为11，如果为0将后面值于11按&位运算输出，将前一个值于00按&位运算输出。



## 8 isLessOrEqual

要求：判断是否x<=y,返回1，否则返回0

```c
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) {
  int x1 = ~x+1;
  int x2 = y+x1;
  int sign = x2>>31&1;
  int leftBit = 1<<31;
  int xLeft = x&leftBit;
  int yLeft = y&leftBit;
  int bitXor = xLeft ^ yLeft;
  bitXor = (bitXor>>31)&1;
  return ((!bitXor)&(!sign))|(bitXor&xLeft>>31);
}
```

思路:先判断是否最高位相同，不同的话返回x的最高位，因为如果下x>y则x最高位为0.相同的话两数相减，如果y>x则最高位为0然后取！

## 9 logicalNeg 

要求:相反实现!运算

```c
/* 
 * logicalNeg - implement the ! operator, using all of 
 *              the legal operators except !
 *   Examples: logicalNeg(3) = 0, logicalNeg(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
int logicalNeg(int x) {
  int negx = ~x+1;
  
  return ((x|negx)>>31)+1;
}
```

思路:如果为零则相反数的最高位为零，否则最高位一定为1

## 10 howManyBits

要求: 计算位数

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
int howManyBits(int x) {
  int b16,b8,b4,b2,b1,b0;
  int sign = x >> 31;
  x = (sign&~x)|(~sign&x); //正数则位取反，负数则不变
    
  b16 = !!(x>>16)<<4; 
  x = x >> b16;
  b8 = !!(x>>8)<<3;
  x = x>>b8;
  b4 = !!(x>>4)<<2;
  x = x>>b4;
  b2 = !!(x>>2)<<1;
  x = x>>b2;
  b1 = !!(x>>1);
  x = x>>b1;
  b0 = x; 
  return b16+b8+b4+b2+b1+b0+1;  //计算位数再加符号位(+1)
}
```

想法:如果是正数则找最高1位，如果为负数则找最高0位计算加上符号位。