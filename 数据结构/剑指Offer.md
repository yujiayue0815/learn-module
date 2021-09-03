# 第一章 整数

## 1.1 基础知识

​	Java 中有4种不同的整数类型（都是有符号整数）：

- 8位的byte（-2^7～2^7-1）
- 16位的short （-215～215-1）
- 32位的int（-231～231-1）
- 64位的long（-263～263-1）

```java
/**
     * 题目：输入2个int型整数，它们进行除法计算并返回商，要求不得使用乘号'*'、除号'/'及求余符号'%'。当发生溢出时，返回最大的整数值。假设除数不为0。例如，输入15和2，输出15/2的结果，即7。
     *
     * @param dividend 被除数
     * @param divisor  除数
     * @return 商
     */
    public int divide(int dividend, int divisor) {
        // 最小值 除 -1 = 最大值
        if (dividend == Integer.MIN_VALUE && divisor == -1) {
            return Integer.MAX_VALUE;
        }
        // 负数个数
        int negative = 2;
        // 被除数是否为负数
        if (dividend > 0) {
            negative--;
            dividend = -dividend;
        }
        // 除数是否为负数
        if (divisor > 0) {
            negative--;
            divisor = -divisor;
        }
        int result = divideCore(dividend, divisor);
        return negative == 1 ? -result : result;
    }

    /**
     * 将被除数和除数转换成负数来进行计算
     *
     * @param dividend 被除数绝对值
     * @param divisor  除数绝对值
     * @return 商
     */
    private int divideCore(int dividend, int divisor) {
        int result = 0;
        while (dividend <= divisor) {
            int value = divisor;
            int quotient = 1;
          	// 0xc0000000 = -2^30 = Integer.MIN_VALUE / 2
            while (value >= 0xc0000000 && dividend <= value + value) {
                quotient += quotient;
                value += value;
            }
            result += quotient;
            dividend -= value;
        }
        return result;
    }
```

## 1.2 二进制

​	二进制运算（6种）：非、与、或、异或、左移、右移

![image-20210824080008689](../imges/image-20210824080008689.png)

​	左移运算符m＜＜n表示把m左移n位。如果左移n位，那么最左边的n位将被丢弃，同时在最右边补上n个0。

```tex
00001010 << 2=00101000
10001010 << 3=01010000
```

​	右移运算符m>>n表示把m右移n位。如果右移n位，则最右边的n位将被丢弃。如果数字原先是一个正数，则右移之后在最左边补n个0；如果数字原先是一个负数，则右移之后在最左边补n个1。

```tex
00001010 >> 2 = 00000010
10001010 >> 3 = 11110001
```

​	Java中增加了一种无符号右移位操作符“>>>”。无论是对正数还是负数进行无符号右移操作，都将在最左边插入0。（其他编程语言没有）

```
00001010>>>2=00000010
10001010>>>3=00010001
```

