---
title: 质数算法 判断质数 小于n的质数 前n个质数
date: 2016-10-12 15:56:09
categories:
- 技术
- 算法与数据结构
tags:
- 算法与数据结构
---

场景一：输入一个整数n，判断n是否为质数。
```
/**
* 判断是否是质数
* 思路（依次优化）
* 1. 从2到n-1遍历判断
* 2. 从2到n/2遍历判断
* 3. 从2到√n遍历判断
* 4. 从2到√n中所有奇数进行遍历
* 5. 从2到√n中所有质数进行遍历
* @param n 参数1
* @return 是否是质数
*/
public  static boolean isPrime(int n){

        if(n<=1)
            return false;
        if(n==2||n==3)
            return true;
        if(n%2==0)
            return false;
        
        for(int i=3;i*i<=n;i+=2){
            if(n%i==0)
                return false;
        }

        return true;
}

```
参考链接：[计蒜客【挑战难题】系列讲解（三）判断质数][1]


场景二：输入一个整数n，输出小于n的所有质数，并返回质数个数。

思路：判断是否>=2，如果是则遍历2到n区间的所有数并做“是否为质数”的判断；显然这样做需要计算量太大了。常用的较快算法采用空间来换取时间，借组一个n+1空间的数组，该数组的下标对应[0-n]区间整数，其中下标为0和下标1不需要用到。然后把质数2、3、7...N（小于n）的倍数（>=2）全部标记，最后数组剩下没有被标记的为所求质数集。如下图，绿色代表当前质数，红色代表已经被标记，从下标数组下标2到n+1依次扫描，当扫到的时质数的（刚好值为FALSE），就开始标记。

```
//方法一  时间复杂度接近O(nlogn)
    public static int printNA(int n){

        int result =0;

        for(int i=0;i<n;i++){
            if(isPrimeC(i)){
                System.out.println(i);
                result++;
            }
        }
        return result;
    }

    public  static boolean isPrimeC(int n){

        if(n<=1)
            return false;
        if(n==2||n==3)
            return true;
        if(n%2==0)
            return false;
        
        for(int i=3;i*i<=n;i+=2){
            if(n%i==0)
                return false;
        }

        return true;
    }


    //方法二  以空间换时间
    public static int printNB(int n){

        if(n<=1)
            return 0;

        boolean[] flags = new boolean[n+1];

        for(int i=0;i<flags.length;i++)
            flags[i]=true;

        for(int i=2;i<flags.length;i++){
            if(flags[i]){
                int count = n/i;
                for(int j=2;j<=count;j++){
                    flags[i*j]=false;
                }
            }
        }

        int result =0;

        for(int i=2;i<flags.length;i++){
            if(flags[i]){
                System.out.println(i);
                result++;
            }
        }
        return result;
    }
```

场景三：输入一个整数n，输出前n个质数。

思路：当n比较小的时候可以采用遍历法；当n比较大的时候，由于质数的分布式越来越稀疏，意味着n个质数可能分布在[2,N]区间中，N会很大。由n和N/InN在数学上的关系相对紧密，所以可由n算出N，然后再利用“场景二”求出N区间内的质数后再输出前n个。

```
// n比较小的时候
    public static void printNA(int n){

        if(n<=0)
            return;
        int count=0;
        for(int i=2;;i++){
            if(isPrime(i)){
                System.out.println(i);
                count++;
                if(count==n)
                    break;
            }
        }

    }

    public  static boolean isPrime(int n){

        if(n<=1)
            return false;
        if(n==2||n==3)
            return true;
        if(n%2==0)
            return false;
        for(int i=3;i*i<=n;i+=2){
            if(n%i==0)
                return false;
        }
        return true;
    }


    // n 比较大的时候
    public static void printNB(int n) {

        if (n < 1)
            return;

        int m = (int) Math.pow(Math.pow(n, Math.E),1.0/(Math.E-1));	

        if (n == 1) {
            System.out.print(2);
            return;
        }

        boolean[] flags = new boolean[m + 1];

        for (int i = 0; i < flags.length; i++)
            flags[i] = true;

        for (int i = 2; i < flags.length; i++) {
            if (flags[i]) {
                int count = m / i;
                for (int j = 2; j <= count; j++) {
                    flags[i * j] = false;
                }
            }

        }

        int count = 0;

        for (int i = 2; i < flags.length; i++) {
            if (flags[i]) {
                System.out.println(i);
                count++;
            }
            if (count >= n)
                break;
        }
        
    }
```

参考链接：[【质数算法】——判断质数、求小于N的质数、求前N个质数][3]


[1]: http://jasinyip.com/CC/jskproblem3.html

[3]: http://blog.csdn.net/u010794180/article/details/48847015