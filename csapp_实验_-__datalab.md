# Datalab

## 前言

> 该实验是《深入理解计算机系统》（英文缩写CSAPP）课程附带实验——Lab1：Data Lab，对应书中第二章内容（信息的表示和处理），是所有实验中的第一个实验，

**实验目的  **

datalab实验提供了一个文件夹，我们的目的只是改写bits.c中的15个函数，使其完成相应的功能即可。至于其他文件是用来编译、测试，并且限制你使用一些禁止的[运算符](https://so.csdn.net/so/search?q=运算符&spm=1001.2101.3001.7020)。

**实验说明**

>  Students implement simple logical, two’s complement, and floating point functions, but using a highly restricted subset of C. For example, they might be asked to compute the absolute value of a number using only bit-level operations and straightline code. This lab helps students understand the bit-level representations of C data types and the bit-level behavior of the operations on data.
>  学生实现简单的逻辑、二进制补码和浮点数，但使用c的一个高度受限的子集。例如，他们可能被要求仅使用位级操作和直线代码计算一个数字的绝对值。本实验帮助学生理解C数据类型的位级表示和数据操作的位级行为。
>
>  > 目标是修改bits.c的副本，使其通过btest中的所有测试，而不违反任何编码准则。

```
makefile -生成btest、fshow和show
readme -此文件
bits.c -你将修改并提交的文件
bits.h -头文件
btest.c -主要的btest程序
btest.h -用于构建btest
decl.c -用于构建btest
tests.c - 用于构建btest
test-header.c - 用于构建btest
dlc* -规则检查编译器二进制(数据实验室编译器)
driver .pl* -使用btest和dlc自动升级位的驱动程序
Driverhdrs.pm -可选的“击败教授”比赛的头文件
fshow.c -用于检查浮点表示的工具
ishow.c -用于检查整数表示的实用程序
```

## 一、不允许允许的操作　

1. 使用任何控制结构，如if, do, while, for, switch等。

2. 定义或使用任何宏。

3. 在此文件中定义任何其他函数。

4. 调用任何库函数。

5. 使用任何其他的操作，如&&, ||, -, or ? :

6. 使用任何形式的casting

7. 使用除int以外的任何数据类型。这意味着你不能使用数组、结构等。[^1]

   [^1]: （浮点运算的规则没那么严格，具体看每个函数上方的规则说明） 

## 二、实验环境

将整个文件拖入Linux环境32位环境中，打开终端；
1.make clean 清除所有编译文件
2.make 编译所有文件 （如果bits.c有问题编译不成功）
3…/btest bits.c 测试bits.c
最终结果：（函数正确情况下）
​​
​​​​​​![请添加图片描述](https://img-blog.csdnimg.cn/direct/a8252e62f1634b26981e815e19ed9972.png)

## 三、函数实现

**需要注意的点:**

> 1. 按位右移>> ：
>    对有符号数都是算数右移（最高位是0补0，最高位是1补1）
>    对无符号数都是逻辑右移（最高位补0）
> 2. 提取符号的方法：x&1 （格式化该数，得到其符号）



### 题目列表

![请添加图片描述](https://img-blog.csdnimg.cn/direct/67f5014060eb461dae1f67f61ca42709.jpeg)


---

---

### 1.bitXor(x,y)

> 只使用两种位运算实现异或操作。

+ 这里可以用一个[真值表](https://so.csdn.net/so/search?q=真值表&spm=1001.2101.3001.7020)

![请添加图片描述](https://img-blog.csdnimg.cn/direct/bc160f8dc85b4ab099fa8ce03874b305.png)


+ **思路：**

**摩根定律**

> ```tex
> a^b=
> 1.(a|b)&(~a|~b)
> 2.~(~a&~b)&~(a&b)
> 3.(a&~b)|(~a&b)
> ```

![请添加图片描述](https://img-blog.csdnimg.cn/direct/8924e6dd0ae94bcfb8a643959140ac5a.png)


+ 代码

```
/* 
 * bitXor - x^y using only ~ and & 
 *   Example: bitXor(4, 5) = 1
 *   Legal ops: ~ &
 *   Max ops: 14
 *   Rating: 1
 */
int bitXor(int x, int y) {
  return ~(~x&~y)&~(x&y);
}
```



---



### 2.tmin()

> 使用位运算获取对2补码的最小 `int` 值。这个题目也是比较简单。

```
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

- 思路

C 语言中 `int` 类型是32位，即4字节数。**补码最小值就是符号位为1，其余全为0。**所以只需要得到这个值就行了，我采用的是对数值 `0x1` 进行移位运算，得到结果。

---



### 3.isTmax

> 判断x是不是补码表示的最大值

- 思路

要我们判断输入的x是不是最大的补码整数。首先，最大补码整数为：符号位为0，其余位为1（其实就是最小补码取反）。所以我么可以先构造最大整数：max = ~(0x1<<31)。

假设x为TMax(01111111,8位)，则x+1得到TMin(10000000)，再加x，取反再取非，即可返回1，而其余值返回0
但因为当x=-1时，最后返回值也为1，所以要排除这种情况



```
int isTmax(int x) {
  return !(~(x+1)^x)&!!((x+1)^0x0);
}

```



---

### 4.allOddBits(x)



- 思路

这个题目还是比较简单的，采用掩码方式解决。首先要构造掩码，使用移位运算符构造出奇数位全1的数 `mask` ，然后获取输入 `x` 值的奇数位，其他位清零（`mask&x`），然后与 `mask` 进行异或操作，若相同则最终结果为0，然后返回其值的逻辑非。



> 判断所有奇数位是否都为1，这里的奇数指的是位的阶级是2的几次幂。重在思考转换规律，如何转换为对应的布尔值。



> 所有奇数位为1则返回1。其中位的编号从0（最低有效位）到31（最高有效位）。
>
> 想要判断一个位是否为1很简单：设计一种掩码，需要检测的位设置为1，其余位设置为0，然后和要检测的数按位与，最后判断结果和掩码是否相等即可。
>
> 本题的难点是怎么构造这样一个补码。因为常数被限制在0~255，即我们只能设置最低的8位。先看8位，奇数位为1偶数位为0即0xAA（10101010），采用位移的方式使32位中每个8位都符合这个标准。结果如下：

```
/* 
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   where bits are numbered from 0 (least significant) to 31 (most significant)
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 */

int allOddBits(int x) {
    int A = 0xA;
    int AA= A|(A<<4);
    int AAA=AA|(AA<<8);
    int mask=AAA|(AAA<<16);
  return !((x&mask)^mask);
}
```

---

### **5.negate**

> 返回**-x**

**思路：**
一个数的相反数为其二进制位表示的按位取反，再+1，即-x=(~x)+1

```
/* negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
  return (~x)+1;
}
```

---

### 6.isAsciiDigit(x)

> 计算输入值是否是数字 30-39 的 `ASCII` 值。这个题让我认识到了位级操作的强大。

**思路：**
要使0x30 <= x <= 0x39，则x-0x30>=0 且 0x39-x>=0
即运算结果的符号位为0，可以用移位操作（有符号数右移为算术右移）
另，减法运算可以看作x+(-x), -x=~x+1

```
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
    //0x30 -> 110000
    //0x39 -> 111001
    //leading zero ->26
    int a = x>>6;
    int cond1 = !a;

    //11xxxx
    int b = x>>4;
    int cond2 = !(b^0b11);

    //111001 1001 -> 9
    int c = x&(0xf);
    int res = c-(0xA);
    int cond3 =!!(res>>31);
  return
```

---

### 7.conditional(x, y, z)

> 使用位级运算实现C语言中的 `x?y:z`三目运算符。又是位级运算的一个使用技巧。

+ 思路：
  用倒推的思路，返回值二选一，return结果一定是用 | 连接
  而一个返回y，一个返回z，返回原值可以用补码全1（即-1）和&来实现，返回0可用0和&来实现
  定义中间量condition=-1或0，condition需要与x相关联，则可以用!!x和取相反数的操作来实现
  当 x!=0时，!!x=1, condition=~(!!x)+1=-1
  当 x= 0时，!!x=0, condition=~(!!x)+1= 0

```
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
int conditional(int x, int y, int z) {
	int mask = ((!!x)<<31)>>31;
  	return (mask&y)|((~mask)&z) ;
}
```

---

### **8.isLessOrEqual**

> 如果x<=y，则返回1，否则返回0
>
> 当相同符号时，直接相减判断是否为负数即可。
> 当前者为正，后者为负时，直接返回0，前者为负后者为正时返回1。

- 思路

​       通过位运算实现比较两个数的大小，无非两种情况：一是符号不同正数为大，二是符号相同看差值符号。



```
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 *   1.x=y
 *   !(x^y)
 *
 *   2.x+ y-
 *   signX = x>>31&0x1;
 *   signY = y>>31&0x1;
 *   !signX&&!signY
 *
 *   3.x- y+
 *   signX&&!signY;
 *
 *   4.x+ y+   x- y-
 *   x+(~y)+1 //x-y
 *   ((x+(~y)-1)>>31)&0x1
 */
int isLessOrEqual(int x, int y) {
    int cond1 = !(x^y);
    int signX = (x>>31)&1;
    int signY = (y>>31)&1;
    int cond2 = !((!signX)&(signY));
    int cond3 = signX&(!signY);
    int cond4 = ((x+(~y)+1)>>31)&0b1;

  return cond1 |( cond2 & (cond3 | cond4)) ;
}
```

---

### 9.logicalNeg

> 实现“！”运算符
>
> 使用位级运算求逻辑非 **`!`**

- 思路

​       逻辑非就是非0为1，非非0为0。利用其补码（取反加一）的性质，除了0和最小数（符号位为1，其余为0），      外其他数都是互为相反数关系（符号位取位或为1）。0和最小数的补码是本身，不过0的符号位与其补码符号位位或为0，最小数的为1。利用这一点得到解决方法。 

```
/* 
 * logicalNeg - implement the ! operator, using all of 
 *              the legal operators except !
 *   Examples: logicalNeg(3) = 0, logicalNeg(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
int logicalNeg(int x) {
    int negx = (~x)+1;
    int sign = (negx|x)>>31;
  return sign+1 ;
}
```

---

### **10. howManyBits**

>  求值：“一个数用补码表示最少需要几位？”
>
>  例子：
>
>  howManyBits(12) = 5
>
>  howManyBits(298) = 10
>
>  howManyBits(-5) = 4
>
>  howManyBits(0) = 1
>
>  howManyBits(-1) = 1
>
>  howManyBits(0x80000000) = 32
>
>  这里我们以12为例 
>
>  `    0000 0000 0000 0000 0000 0000 0000 1100`
>
>  > 实际上要得到12只需要4+1（符号位）=5位即可
>  >
>  > 这里我们可以看出，计算一个数最小需要多少比特位的问题转化成了从最高位到最低位寻找第一个数据为1的问题
>  >
>  > 这里我们使用类似于二分的思想
>  >
>  > 先看高低16位（判断是否存在数据为1的比特位），在看8位，4，2，1......
>  >
>  > 这里用   flag = !!(x>>16);
>  >
>  > 如果flag结果为1，表示高16位中存在一位或者多位数据为1的比特位
>  >
>  > 为0表示高16位中不存在数据为1的比特位



```
/* howManyBits - return the minimum number of bits required to represent x in
 *             two's complement
 *  Examples: howManyBits(12) = 5
 *            howManyBits(298) = 10
 *            howManyBits(-5) = 4
 *            hiowManyBits(0)  = 1
 *            howManyBits(-1) = 1
 *            howManyBits(0x80000000) = 32
 *  Legal ops: ! ~ & ^ | + << >>
 *  Max ops: 90
 *  Rating: 4
 */
int howManyBits(int x) {
	int isZero = !x;
	int flag = x>>31;
	int mask = ((!!x)<<31)>>31;
	x = (flag & (~x)) | ((~flag) & x );
	int bit_16,bit_8,bit_4,bit_2,bit_1,bit_0;

	bit_16 =(!((!!(x>>16))^(0x1)))<<4;
	x>>=bit_16;
	bit_8 =(!((!!(x>>8))^(0x1)))<<3;
	x>>=bit_8;
	bit_4 =(!((!!(x>>4))^(0x1)))<<2;
	x>>=bit_4;
	bit_2 =(!((!!(x>>2))^(0x1)))<<1;
	x>>=bit_2;
	bit_1 =(!((!!(x>>1))^(0x1)));
	x>>=bit_1;
	bit_0 =x;

	int ret = bit_16 + bit_8 + bit_4 + bit_2 + bit_1 + bit_0 + 1;

	return isZero | (mask & ret) ;
}
```



---

### 11.unsigned floatScale2(unsigned uf)

> 求浮点数乘以2的结果
>
> 根据IEEE浮点规则我们知道
>
> > $$V  =  (（-1）^ s )  *  M  *  ( 2 ^ E )$$

⚠️这里需要注意的是函数传入的参数以及返回值都是无符号整型数

也就是说变量uf虽然是一个无符号整型数，但在题目中我们需要把它的二进制表示解析成一个单精度浮点数



#### 思路1

![请添加图片描述](https://img-blog.csdnimg.cn/direct/c4dae24f8c104890ae587fe107f81e23.png)


看了图,思考一下如何将一个无符号整型数解析成单精度浮点数呢？

A：根据单精度浮点数的定义来划分变量uf的32个bit位

其中uf二进制表示的最高位（31位）表示的是符号位s

第23～30位表示阶码字段（exp）

第0～22位表示小数字段（frac）

我们需要从uf中提取符号字段、阶码字段和小数字段

那么获取阶码字段exp将uf与0x7F800000（如下）按位“与运算”，再将结果右移23位即可

```
0     1111 1111（exp）   0000.....0000
```

即 exp = （0x7F800000 & uf ) >>23

这样就解析出阶码字段exp，获取其他字段的方式同理

由此，我们将无符号整型数解析成单精度浮点数

根据IEEE浮点规则我们知道

**$$V  =  (（-1）^ s )  *  M  *  ( 2 ^ E )$$**

根据阶码exp的值分类讨论

1.当exp = 0xFF时，表示特殊值，当为特殊值时我们知道有两种情况。一种（当frac不等于0）表示不是一个数（NaN），另一种（frac等于0）表示无穷（根据符号位表示正无穷还是负无穷）。

        当为NaN时根据题目要求，直接返回uf即可
    
        当为无穷时，无穷 * 2 结果还是无穷，返回uf

2.当exp = 0时，表示非规格化的数，由于非规格化的数表示数值0或者非常接近0的数

        当frac = 0 时，此时表示0 ，0乘以任何数都为0，所以直接返回uf（注意当符号位不同时，正零与负零是有区别的，但由于0怎么乘都为0，所以这里我们不做讨论，直接返回uf，不能返回零）
    
        当frac != 0 时，此时表示非常接近0的数，针对这情况只需将小数字段乘以2即可（左移一位）

3.当exp 既不等于0，也不等于0xFF时，此时表示规格化的数

        对于这种情况乘以2只需要对 exp + 1 即可
    
        （具体原因可以看上面的IEEE浮点表示规则）
    
        不过这里还有一种特殊情况要出处理，当exp = 254，此时虽然是一个规格化的数，但阶码字段加1之后会超出规格化所能表示的范围，针对这种情况，我们需要返回无穷（无穷分为正无穷和负无穷，这里根据符号位判断即可）

---

#### **思路**2

1.首先考虑第一种情况

> When argument is NaN, return argument

需要先求出`exp`

```cpp
int exp = (uf&0x7f800000)>>23; //23-30 这8位
int sign=uf>>31&0x1; //符号位
int frac=uf&0x7FFFFF;
```

如果`exp=255` 并且尾数非0 就是`NaN` 直接return 就好 其次如果`frac` 全为0 那么则表示无穷大 这两种情况都可以直接return

1. 如果`exp=0` 则表示非规格化数

![请添加图片描述](https://img-blog.csdnimg.cn/direct/1fa9dac0687f4737ba034c154e99b551.jpeg)


> 那么我们直接返回`uf*2` 就可就是把`frac>>1`





 2.如果`exp!=0 && ！=255` 那么表示规格化数

![请添加图片描述](https://img-blog.csdnimg.cn/direct/dd1e304217474e669eb20628c350a5c6.jpeg)


那么我们的修改就先把`exp+1`

最终用或操作和移位操作将这三个字段拼成一个32bit的数返回即可

#### 代码

```
//float
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
unsigned floatScale2(unsigned uf) {
	//s,expr,frac
	unsigned s = (uf >> 31) & (0x1);
	unsigned expr = (uf>>23) & (0xff);
	unsigned frac = (uf & 0x7fffff);

	//0
	if(expr == 0 && frac == 0)
		return uf;
	//inifity or nor na number
	if(expr == 0xff)
		return uf;
	//denormalize
	if(expr == 0)
	{
		//E = expr - basic
		//    expr - 127 = -127
		// frac
		frac <<= 1;
		return (s << 31 ) | frac;
	 }
	//normalize
	expr++; //即乘以二
	//E = expr - 127
	return (s << 31) | (expr << 23) | (frac);
}
```

---

### 12.int floatFloat2Int(unsigned uf)

> 把单精度浮点数强制转化成整型数

+ **思路**

​     6位IEEE浮点数格式如下

![请添加图片描述](https://img-blog.csdnimg.cn/direct/14c469eeddeb412bbcd9d42010db5915.jpeg)


根据上图我们可以分为三种情况

先计算出`E=exp-bias`

1. 如果是小数 `E< 0`的情况我们直接返回0

2. 如果是`exp=255` 的情况直接返回`0x80000000u` 这里注意如果超范围了也会直接返回0x80000000u
   因此可以直接用`E>=31` 来判断

3. 如果是规格化数则我们进行正常处理 $$𝑉=(−1)𝑠×𝑀×2𝐸$$

4. 1. 先给尾数补充上省略的1
   2. 判断`E<23` 则尾数需要舍去`23-E`位
   3. 根据符号位返回就好



>     1.当E等于0时，此时表示的数不是整数，所以只需返回0即可
>
>     由于浮点数所能表示的范围远大于整型数（32位整型数只能表示 -2^31  ~  2^31 -1 ）
>
>                    当  E = 31， s = 1， V = -2^31
>         
>                        E = 31， s = 0， V = 2^31 
>         
>                        此时浮点数表示的数超过了整型所能表示的最大值
>
>     2.综上，
>     	当E >31 时   浮点数所表示的数值超过了整型数能表示的最大值，此时函数返回0x80000000u即可
>
>     3.E在 0 ～ 31 之间如何转换   
>
>            此时根据exp的值可以知道，数据为规格化的数
>         
>            由于E的取值范围在0 ～ 31，而小数字段的长度是23位，所以E的范围又要分成两种情况讨论
>         
>            0<= E <= 23        
>            对小数字段进行截断处理（如果E = 23，直接返回小数字段，E= 22，舍弃小数字段最后一位-->小数字段右移一位）
>         
>             24 <= E <= 31      E = 24(小数字段左移一位)，E = 25（左移两位）
>             最后看符号位，若为负数则取反加一，正数直接返回即可



```
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
int floatFloat2Int(unsigned uf) {
	unsigned s = (uf >> 31) & (0x1);
        unsigned expr = (uf>>23) & (0xff);
        unsigned frac = (uf & 0x7fffff);
	
	//0
	if(expr == 0 && frac == 0)
	       return 0;
	//Ini / NaN 无穷大或无穷小
	if(expr == 0xff)
		return 1 << 31;
        //denormalize 非规格化
	if(expr ==0)
	{
	   //M 0.1111  < 1
	   //E = 1 - 127 = -126
	   return 0;
   	}
	//normalize 规格化
	//M 1.xxxxx(2)
	int E = expr - 127;
	frac = frac | (1 << 23);
	if(E > 31)
		return 1 << 31;
	else if(E < 0)
	 	return 0;
	//10 -> 1. < 2
	//M * 2^E
	if(E >= 23)
		frac <<= (E - 23);
	else
		frac >>= (23 - E);
	if(s) //负数
		return (~frac) + 1;
	return frac;
}
```

---

### **13 floatPower2**(X)

> 求 $$2.0^𝑥$$

- 思路

2.0的位级表示（ 1.0×21 ）：符号位：0，指数：1+127=128，frac=1.0-1=0。 2.0𝑥=(1.0×21)𝑥=1.0×2𝑥 ，所以x就当做真正的指数的。

这个比较简单，首先得到偏移之后的指数值e，如果e小于等于0（为0时，结果为0，因为2.0的浮点表示frac部分为0），对应的如果e大于等于255则为无穷大或越界了。否则返回正常浮点值，frac为0，直接对应指数即可。

![请添加图片描述](https://img-blog.csdnimg.cn/direct/44ef9058485147f883d88be57a3e89fc.jpeg)


根据上图我们可以得出几个边界

1. `x>127` 返回+NAN
2. `x<-148`太小返回0
3. `x>=-126`规格化数
4. 否则就是非规格化数

```
unsigned floatPower2(int x) {
    if(x>127){
        return 0xFF<<23;
    }
    else if(x<-148)return 0;
    else if(x>=-126){
        int exp = x + 127;
        return (exp << 23);
    } else{
        int t = 148 + x;
        return (1 << t);
    }
}
```



- 代码

```
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
unsigned floatPower2(int x) {
/*	if(x < -149){
		return 0;
	}else if(x < -126){
		//E = x
		//E = 1 -127 = -126
		int shift = 23 + (x +126);
		return (1 << shift;)
	}else if(x <= 127){
		//x = expr - bias
		int expr = x + 127;
		return expr << 23;
	}else{	
		return (0xFF) << 23;
	}
*/
	int exp;
	unsigned ret;

	if(x <-149) return 0;
	if(x >127) return (0xff <<23);

	if(x <-126) return 0x1 << (x +149);
	exp = x + 127;
        ret = exp << 23;
	return ret;       
}
```


