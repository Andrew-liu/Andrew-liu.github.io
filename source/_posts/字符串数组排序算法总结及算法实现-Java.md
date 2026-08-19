title: 字符串数组排序算法总结及算法实现(Java)
date: 2015-05-31 15:48:20
tags: Algorithm
---



本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.

> 对于字符串数据排序问题的总结, 如有错误, 欢迎指出.

<!--more-->

#键索引计数法

> 这种排序算法是以下两种排序算法的基础


**假设有以下一组数据**

|组号 |姓名 |
|---|---|
|2|Anderson |
|3|Brown|
|3|Davis|
|1|Harris|

算法步骤:

- 首先进行组号出现的频率统计, 使用count数组记录每个组号出现的次数

```
for(i = 0; i < N; i++) 
    count[a[i].key() + 1)]++; //a[i].key()返回组号, count的下标从2开始, 可以由下面找到答案
```
- 将频率转换为索引, 使用count来计算每个组号在排序结果中的起始索引位置

```
for(int r = 0; r < R; r++)
    count[r + 1] += count[r]  #1组数组下标应该从零开始, 没有第0组, 第2组下标应该从1开始(1组有一个成员), 3组下标从2开始(1组和2组共两个成员)
```
- 数据分类, 将count[]转换为索引表后, 将名字移动到对应的辅助数组中排序

```
for(int i = 0; i < N; i++)
    aux[count[a[i].key()]++] = a[i] //count[a[i].key()]表示某一组的下标, ++表示同组的下一个名字的下标
```

- 回写, 将aux数组的数据回写到原排序数组中

```
for(i = 0; i < N; i++)
    a[i] = aux[i];
```



#低位优先的字符串排序

**低位优先排序算法**是通过建索引计数法完成的, 从每个字符串的最右侧开始, 以每个位置的字符作为key键(相当于组号), 用`键索引计数法`对字符串排序W遍(假设所有字符串长度为W)

> 由于键索引计数法的排序是稳定的, 则可以推出低位优先的字符串排序算法能够稳定的排序定长字符串

算法缺点: 

- 只能用于处于等长字符串排序
- 需要额外的count[]和aux[]数组空间


```java
public class LSD {
    public static void sort(String[] a, int W) {
        //通过后W个字符将a[]排序, 从低位开始
        int N = a.length; //字符串数组长度
        int R = 256;  //radix; 最多表示256的字符extern-ASCII
        String[] aux = new String[N];  //存储排序后的字符串数组
        for (int d = W - 1; d >= 0; d--) {  //低位排序从这里体现
            int[] count = new int[R + 1];
            //计算出现频率
            for (int i = 0; i < N; i++) {
                count[a[i].charAt(d) + 1]++;  //count每一位统计对应的字符的出现次数
            }
            //将频率转换成索引, 从0开始, 256个字符从第二个字符串开始, 所在的索引应该是第一个字符的频率, 第三个字符串索引应该是第一个字符加上第二个字符
            for (int r = 0; r < R; r++) {
                count[r + 1] += count[r];  //第一个字符换索引为0
            }
            for (int i = 0; i < N; i++) {
                aux[count[a[i].charAt(d)]++] = a[i];  //将字符串应对的d位的数, 放在当前索引下, 然后索引加1
            }
            //写会a[], 重复W词, 因为按照W个字符排序
            for (int i = 0; i < N; i++) {
                a[i] = aux[i];
            }
        }
    }
}
```


#高位优先的字符串排序

**高位优先排序算法**用于更通用的情形(字符串不定长), 首先用`键索引计数法`对所有字符串首字母排序, 然后在`递归的`将每个首字母所对应的子数组排序(忽略首字母)

当指定位置超出字符串的末尾应该返回`-1`.**越界则输出-1. 这种转换意味着字符串中的每个字符都可能产生R+1中不同的值，同时键索引计数法本来就需要一个额外的位置，所以使用代码int count[] = new int[R + 2];**


在高位优先排序中, 每次递归过程都会创建一个count数组并转换成索引, 当数据量过大, 会造成很大的`空间代价`, 所以当进行小数组排序时, 切换为使用`插入排序算法`, 避免重复检查已知相同字符带来的成本


算法缺点:
1. 高位优先的字符串排序使用了两个辅助数组（aux[]和count[]）
2. 高位优先的字符串排序的最坏情况就是所有的键均相同

```java
public class MSD {
    private static int R = 256; //字符数
    private static final int M = 15;  //小数组的切换阈值
    private static String[] aux;  //辅助数组, 存储中间排序结果

    private static int charAt(String s, int d) {
        if (d < s.length())
            return s.charAt(d);
        else
            return -1;
    }
    public static void sort(String[] a) { //排序的外部接口
        int N = a.length;
        aux = new String[N];
        sort(a, 0, N - 1, 0);
    }
    //排序的内部实现, 以第d个字符为键将a[lo]至a[hi]排序, 递归过程
    private static void sort(String[] a, int lo, int hi, int d) {
        if (hi <= lo + M) {
            Insertion.sort(a, lo, hi, d);  //大于阈值则使用插入排序
            return;
        }
        int[] count = new int [R + 2]; //计算频率
        for (int i = lo; i <= hi; i++) {
            count[charAt(a[i], d) + 2]++; //由左到右同此每个字符的频率
        }
        for (int r = 0; r < R + 1; r++) {
            count[r + 1] += count[r]; //频率转换为索引
        }
        //数据分类
        for (int i = lo; i <= hi; i++) {
            aux[count[charAt(a[i], d) + 1]++] = a[i];
        }
        //回写
        for (int r = 0; r < R; r++) {
            sort(a, lo + count[r], lo + count[r + 1] - 1, d + 1); //对每一段相同字符的字符串数据再进行排序, 比如所有以a开头的字符串数据, 对他们第2未再排序
        }
    }
    public static class Insertion {
        public static void sort(String[] a, int lo, int hi, int d) {//从第d个字符开始对a[lo]到a[hi]排序
            for (int i = lo; i <= hi; i++) {
                for (int j = i; j > lo && less(a[j], a[j - 1], d); j--)
                    exch(a, j, j - 1);
            }
        }
        private static boolean less(String v, String w, int d) {
            return v.substring(d).compareTo(w.substring(d)) < 0;
        }
        private static void exch(String[] a, int i, int j) {
            String temp = a[i];
            a[i] = a[j];;
            a[j] = temp;
        }
    }
}
```

#三向字符串快速排序


> 根据高位优先, 改进为快速排序策略, 这种方法是前两种方法的结合

根据字符串数组的首字母进行三向切分, 然后`递归的`将得到的三个子数组排序, 一个含有所有首字母小于切分字符的字符串子数组, 一个含有所有首字母等于切分字符的子数组, 一个含有所有首字母大于气氛字符串的子数组


> 三项字符串快速排序能够很好地处理等值键、有较长公共前缀的键、取值范围较小的键和小数组，且只需要递归所需的隐式栈的额外空间

```java

public class Quick3String {
    private static int charAt(String s, int d) {
        if (d < s.length())
            return s.charAt(d);
        else
            return -1;
    }
    public static void sort(String[] a) {
        sort(a, 0, a.length - 1, 0);
    }
    private static void sort(String[] a, int lo, int hi, int d) {
        if (hi <= lo)
            return;
        int lt = lo, gt = hi;
        int v = charAt(a[lo], d); //povit
        int i = lo + 1;
        while (i <= gt) {
            int t = charAt(a[i], d);
            if (t < v)
                exch(a, lt++, i++);
            else if (t > v)
                exch(a, i, gt--);
            else
                i++;
        }
        sort(a, lo, lt - 1, d);
        if (v >= 0)
            sort(a, lt, gt, d + 1);
        sort(a, gt + 1 , hi, d);
    }
    private static void exch(String[] a, int i, int j) {
        String temp = a[i];
        a[i] = a[j];;
        a[j] = temp;
    }
}
```

#参考链接

- `<算法>`
- `普林斯顿-Algorithm II`

