---
tags:
  - "#data-structure/bloom-filter"
aliases:
  - 布隆过滤器
---

## 开始之前
给你 2000万 个url，又给你一个新的url: $u_1$，让你判断 $u_1$ 是否在原来的 200万个url中，这时你会怎么办呢



### 数据库
你将所有数据一股脑地放入数据库，一切交给数据库处理。

然鹅需求变了，现在又给你了很多 url，让你判断是否存在，这时对于每个查询，数据库都要重新跑一遍，是不是太小题大做了

### HashSet
你将所有 URL 放入一个 HashSet 中，然后判断
时间复杂度：$O(1)$
空间复杂度：所有url的长度和，一般为 $O(n)$

但是当数据量**极大**时，假设每个 url 占用 50 字节，那么需要 **1G** 的内存空间

### Hash预处理
你将所有 url 通过 hash 函数处理后得到的哈希值放入 HashSet（数据库），这时节省了不少空间和时间
时间复杂度：$O(1)$
空间复杂度：$O(c*n)$，c为哈希值长度（md5为128bit，sha-1为160bit）

### BitSet
你更进一步，将所有 url 映射到 位图上，假设你建了个长为 1亿 的位图，只需要 95M 的空间就可以存下所有url。
时间复杂度：$O(1)$，忽略建表时间
空间复杂度：$O(n)$

但是由于哈希冲突的存在，其实在存入 1e4 个不同url之后，哈希冲突的概率就达到了 50%，这令你很沮丧 o(TヘTo)

### 救星来了
实质上上面的算法都忽略了一个重要的隐含条件：**允许小概率的出错**，不一定要100%准确！

## 什么是布隆过滤器

布隆过滤器（Bloom Filter）是 1970 年由布隆提出的。它实际上是一个很长的二进制向量和一系列随机映射函数。布隆过滤器可以用于检索一个元素是否在一个集合中。**它的优点是空间效率和查询时间都比一般的算法要好的多，缺点是有一定的误识别率和删除困难**。

### 算法原理
使用 BitSet，和若干个Hash函数
#### 插入
对于每个url，将所有Hash后的位置设为1

如下图，有4个Hash函数，结果分别为 8，1，6，13
那么将对应位置全部设为1
![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2022/11/06/20221106160914-93444e.png)

#### 那么如何判断元素是否存在呢？
只有重新计算Hash，如果对应位置上全都是1，那么很有可能存在。
反之，如果存在一个0，就说明url不存在。

为什么是可能存在呢？
举个反例：
```
"hello" 的哈希为：
1、2、3、4
"xyz" 的哈希为
2、3、4、5
"sadj" 的哈希为
1、3、4、5
那么在插入 "hello" 和 "xyz" 后能说明 "sadj" 存在吗，不能吧
```
#### 如何计算误判率
误判率（false positive prob）：过滤器认为其存在，但其实不存在的概率

幸运的是，布隆过滤器可以[预测误判率](http://citeseer.ist.psu.edu/viewdoc/download;jsessionid=6CA79DD1A90B3EFD3D62ACE5523B99E7?doi=10.1.1.127.9672&rep=rep1&type=pdf)：
$$
P_{f} p \approx\left(1-e^{-\frac{k n}{m}}\right)^{k}
$$
其中：
- $n$ 是已经添加元素的数量；
- $k$ 哈希的次数；
- $m$ 布隆过滤器的长度（如位图的大小）；

由于$m$一旦设定不可改变，实际中，我们可以指定可以接受的最大误判率($fpp$)和元素数量$n$来确定$m$，公式如下
$$
m=-\frac{n \ln {(P_{f p}})}{(\ln 2)^{2}}
$$
当然，$k$也可以计算出来：[2](http://en.wikipedia.org/wiki/Bloom_filter#Probability_of_false_positives) [3](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.152.579&rank=1)
$$
\frac{m}{n} \ln (2)
$$
### Bloom filter 的时间复杂度和空间复杂度?

对于一个 _m_ 和 _k_ 值确定的 Bloom filter，插入和测试操作的时间复杂度都是 O(k)。这意味着每次你想要插入一个元素或者查询一个元素是否在集合中，只需要使用 _k_ 个哈希函数对这个元素求值，然后将对应的比特位标记或者检查对应的比特位。

相比之下，Bloom filter 的空间复杂度更难以概述，它取决于你可以忍受的错误率。同时也取决于输入元素的范围，如果这个范围是有限的，那么一个确定的比特向量就可以很好的解决问题。如果你甚至不能很好的估计输入元素的范围，那么你最好选择一个哈希表或者一个可拓展的 Bloom filter。[4](http://gsd.di.uminho.pt/members/cbm/ps/dbloom.pdf) 

## 布隆过滤器实战

### Java

#### 导包

这里使用的是Google实现的guava仓库

```xml
<!-- https://mvnrepository.com/artifact/com.google.guava/guava -->
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>22.0</version>
</dependency>
```

#### 创建测试类

```java
import com.google.common.hash.BloomFilter;
import com.google.common.hash.Funnels;
import org.junit.Test;

import java.nio.charset.Charset;
import java.text.NumberFormat;
import java.util.*;

public class GoogleBloomFilterTest {
    private static int insertions = 10000000;
    @Test
    public static void test() {
 
        BloomFilter<String> bloomFilter = BloomFilter.create(Funnels.stringFunnel(Charset.defaultCharset()), insertions, 0.001);
 
        Set<String> sets = new HashSet<String>(insertions);
 
        List<String> lists = new ArrayList<String>(insertions);
 
        for (int i = 0; i < insertions; i++) {
            String uid = UUID.randomUUID().toString();
            bloomFilter.put(uid);
            sets.add(uid);
            lists.add(uid);
        }
 
        int right = 0;
        int wrong = 0;
 
        for (int i = 0; i < 10000; i++) {
            String data = i % 100 == 0 ? lists.get(i / 100) : UUID.randomUUID().toString();
            if (bloomFilter.mightContain(data)) {
                if (sets.contains(data)) {
                    right++;
                    continue;
                }
                wrong++;
            }
        }
 
        NumberFormat percentFormat = NumberFormat.getPercentInstance();
        percentFormat.setMaximumFractionDigits(2);
        float percent = (float) wrong / 9900;
        float bingo = (float) (9900 - wrong) / 9900;
 
        System.out.println("在 " + insertions + " 条数据中，判断 100 实际存在的元素，布隆过滤器认为存在的数量为：" + right);
        System.out.println("在 " + insertions + " 条数据中，判断 9900 实际不存在的元素，布隆过滤器误认为存在的数量为：" + wrong);
        System.out.println("命中率为：" + percentFormat.format(bingo) + "，误判率为：" + percentFormat.format(percent));
    }
 
}
```

#### 输出

```
539aa3e7_334324324234234sfsfsdfwwrtregreg
1000万次md5Test算法程序运行时间： 4857ms
1000万次murmur3Test算法程序运行时间： 2057ms
```

## 写一个布隆过滤器

### Java

#### 导入计算哈希的包

使用MD5计算哈希

```xml
<!-- https://mvnrepository.com/artifact/commons-codec/commons-codec -->
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.13</version>
</dependency>
```

或者使用 MurMurHash 计算，MurMurHash 的性能比 Md5好。Google已经帮我们实现了MurMurHash

```java
import java.nio.charset.StandardCharsets;
import org.apache.commons.codec.digest.DigestUtils;
import com.google.common.hash.Hashing;
import org.junit.Test;

/**
 * 测试 murmurhash 算法性能
 */
public class TestMurMurHashPerformance {
   @Test
   public void testHashSpeed() {
      
      System.out.println(murmur3Test("334324324234234sfsfsdfwwrtregreg"));
      
       long startTime=System.currentTimeMillis();
       for (int i = 0; i < 10000000; i++) {
          TestMurMurHashPerformance.md5Test("KFETHGRETWERFSDFWEFWEFWF");
        }
        long endTime=System.currentTimeMillis();
        System.out.println("1000万次md5Test算法程序运行时间： " + (endTime - startTime ) + "ms");
        
        long startTime2=System.currentTimeMillis();
       for (int i = 0; i < 10000000; i++) {
          TestMurMurHashPerformance.murmur3Test("KFETHGRETWERFSDFWEFWEFWF");
        }
        long endTime2=System.currentTimeMillis();
        System.out.println("1000万次murmur3Test算法程序运行时间： " + (endTime2 - startTime2 ) + "ms");
      
   }
   
   public static String murmur3Test(String primaryKey) {
        return Hashing.murmur3_32().hashString(primaryKey, StandardCharsets.UTF_8).toString() + 
            "_" + primaryKey;
    }
   
   public static String md5Test(String primaryKey) {
           return DigestUtils.md5Hex(primaryKey)+ "_" + primaryKey;
   }

}
```

#### 首先是接口

这个接口定义了添加元素和检验元素的方法

```java
public interface IBloomFilter<T> {
    public void add(T obj);
    public boolean contains(T obj);
}
```

#### 类实现

```java
package com.github.hyperv0id.bf.bf;

import com.google.common.hash.Hashing;

import java.util.BitSet;

import static java.lang.Math.abs;

public class StandardBloomFilter<T> implements IBloomFilter<T>{
    private final int n;
    private final int m;
    private final BitSet _bitset;

    private final int k;

    private double fpp = 0.03;

    /**
     * 创建过滤器，指定大小，根据fpp为默认值0.03，根据 fpp计算 哈希函数个数 k
     * @param n 插入元素个数
     */
    public StandardBloomFilter(int n) {
        this.n = n;
        // m = n*log(fpp) / (ln2^2)
        this.m = (int) (-(double)n*Math.log(fpp) / Math.pow(Math.log(2), 2));
        this._bitset = new BitSet(m);

        this.k = (int) (((double)m / n) * Math.log(2));

    }

    public StandardBloomFilter(int n, double fpp) {
        this.n = n;
        this.fpp = fpp;
        this.m = (int) (-(double)n*Math.log(fpp) / Math.pow(Math.log(2), 2));
        this._bitset = new BitSet(m);

        this.k = (int) (((double)m / n) * Math.log(2));
    }


    @Override
    public void add(T obj) {
        byte[] bytes = obj.toString().getBytes();

        long hashLong = Hashing.murmur3_128().hashBytes(bytes).asLong();
        int hash1 = (int) hashLong;
        int hash2 = (int) (hashLong>>>32);
        int hashVal;
        for (int i = 1; i <= k; i++) {
            hashVal = ((hash1 + i*hash2)%m + m)%m;
            this.setBit(hashVal);
        }
    }

    private void setBit(int idx){
        _bitset.set(idx);
    }
    private boolean getBit(int idx){
        return _bitset.get(idx);
    }

    @Override
    public boolean contains(T obj) {
        byte[] bytes = obj.toString().getBytes();

        long hashLong = Hashing.murmur3_128().hashBytes(bytes).asLong();
        int hash1 = (int) hashLong;
        int hash2 = (int) (hashLong>>>32);
        int hashVal;
        for (int i = 1; i <= k; i++) {
            hashVal = ((hash1 + i*hash2)%m + m)%m;
            if(!this.getBit(hashVal)){
                return false;
            }
        }
        return true;
    }

    public int getN() {
        return n;
    }

    public int getM() {
        return m;
    }

    public int getK() {
        return k;
    }

    public double getFpp() {
        // P_{f} p \approx\left(1-e^{-\frac{k n}{m}}\right)^{k}
        double t_fpp = (1 - Math.exp(((double) -k * n)/ m));
        t_fpp = Math.pow(t_fpp, k);
        fpp = t_fpp;
        return fpp;
    }
}
```

#### 用于测试的数据集生成器

##### BaseDataset

所有数据集生成器的子类，有基本的创建测试集训练集方法，以及随机数生成方法，子类只需要重写随机数生成方法就可以创建对应分布的数据集

```java
package com.github.hyperv0id.bf.dataset;

import java.util.*;

public class BaseDataset {
    public List<Long> add_list;
    public List<Map.Entry<Long, Boolean>> check_list;
    public int size;
    public int itemsIn;
    Random random;

    public BaseDataset(int size){
        this.size = size;
        itemsIn = 0;
        random = new Random();
        add_list = new ArrayList<>();
        check_list = new ArrayList<>();
    }

    public String name(){
        return "基础数据集";
    }

    public BaseDataset initData(){
        System.out.println("正在创建" + this.name());

        Set<Long> inserted = new HashSet<>();

        long randVal;
        for (int i = 0; i < size; i++) {
            randVal = this.getRandomValue();
            inserted.add(randVal);
            add_list.add(randVal);
        }
        boolean isIn;
        for (int i = 0; i < size; i++) {
            randVal = (i%100 == 0) ? add_list.get(i/100) : this.getRandomValue();
            isIn = inserted.contains(randVal);
            if (isIn){itemsIn++;}
            check_list.add(Map.entry(randVal, isIn));
        }

        System.out.println(name() + "创建成功");
        return this;
    }
    protected long getRandomValue(){
        throw new RuntimeException("不允许使用此方法，请在子类中重写");
    }
}
```



##### 均匀分布的数据集

```java
package com.github.hyperv0id.bf.dataset;


public class UniformDataset extends BaseDataset{
    public UniformDataset(int size) {
        super(size);
    }

    @Override
    public String name() {
        return "均匀分布数据集";
    }
    @Override
    protected long getRandomValue() {
        return random.nextLong();
    }
}
```



##### 高斯分布的数据集

```java
package com.github.hyperv0id.bf.dataset;

public class GaussianDataset extends BaseDataset{
    int range;

    public GaussianDataset(int size, int range) {
        super(size);
        this.range = range;
    }

    @Override
    public String name() {
        return "正态分布数据集";
    }


    @Override
    protected long getRandomValue() {
        return (long)(random.nextGaussian() * range);
    }
}
```



##### 指数分布的数据集

```java
package com.github.hyperv0id.bf.dataset;

public class ExpDataset extends BaseDataset{
    double lambda = 1.0;
    double modifier;
    public ExpDataset(int size, double modifier) {
        super(size);
        this.modifier = modifier;
    }

    @Override
    public String name() {
        return "指数分布数据集";
    }

    @Override
    protected long getRandomValue() {
        return (long) (Math.log(1-random.nextDouble())/(-lambda) * modifier);
    }
}
```



#### 测试一下

```java
package com.github.hyperv0id.bf.bf;

import com.github.hyperv0id.bf.dataset.BaseDataset;
import com.github.hyperv0id.bf.dataset.ExpDataset;
import com.github.hyperv0id.bf.dataset.GaussianDataset;
import com.github.hyperv0id.bf.dataset.UniformDataset;
import org.junit.Test;

import java.util.*;

public class StandardBloomFilterTest {
    public static final int insertions = 10000000;

    static final BaseDataset uniformDataset = new UniformDataset(insertions).initData();
    static final BaseDataset gaussianDataset = new GaussianDataset(insertions, (int)1e8).initData();
    static final BaseDataset expDataset = new ExpDataset(insertions, 1e-5).initData();

    @Test
    public void testSBF_fpp1(){
        StandardBloomFilter<Long> bf = new StandardBloomFilter<>(insertions, 0.01);
        for (long i = 0; i < insertions; i++) {
            bf.add((i*2));
        }
        int wrong=0, times = 1000,exists=times/2;
        for (long i = 0; i < times; i++) {
            if(bf.contains(i) && i%2==1){
                System.out.printf("%d 不存在, 但被误判存在\n", i);
                wrong++;
            }
        }

        double fpp = (double)wrong / (times-exists);
        System.out.printf("在 %d 次查询中，有 %d 次误判，误判率为 %f\n", times, wrong, fpp);
        System.out.println("理论误判率为：" + bf.getFpp());
    }

    @Test
    public void testSBF_fpp2(){
        StandardBloomFilter<String> bf = new StandardBloomFilter<>(insertions);
        Set<String> set = new HashSet<>();
        List<String> list = new ArrayList<>();
        String uid;
        for (int i = 0; i < insertions; i++) {
            uid = UUID.randomUUID().toString();
            bf.add(uid);

            set.add(uid);
            list.add(uid);
        }

        int wrong = 0, times = insertions, exist = 0;
        for (int i = 0; i < times; i++) {
            uid = i % 100 == 0 ? list.get(i / 100) : UUID.randomUUID().toString();
            if(bf.contains(uid)){
                if(!set.contains(uid)){
                    wrong++;
                }
            }
        }
        int notExist = times - exist;
        double fpp = (double)wrong/notExist;
        System.out.printf("在 %d 次查询中，存在 %d 次误判，误判率为 %f\n", times, wrong, fpp);
        System.out.println("理论误判率为：" + bf.getFpp());
    }

    @Test
    public void testSBF_AddPerf(){
        StandardBloomFilter<Long> bf = new StandardBloomFilter<>(insertions);
        long start_time = System.currentTimeMillis();
        for (Long number : uniformDataset.add_list) {
            bf.add(number);
        }
        long end_time = System.currentTimeMillis();
        double time_sp = ((double) end_time-start_time)/1000;
        System.out.printf("添加 %d 个元素 一共用时 %f 秒", insertions, time_sp);
    }

    @Test
    public void testSBF_ContainPerf(){
        StandardBloomFilter<Long> bf = new StandardBloomFilter<>(insertions);
        for (Long number : uniformDataset.add_list) {
            bf.add(number);
        }

        long start_time = System.currentTimeMillis();
        for (Map.Entry<Long, Boolean> pair :
                uniformDataset.check_list) {
            bf.contains(pair.getKey());
        }
        long end_time = System.currentTimeMillis();
        double time_sp = ((double) end_time-start_time)/1000;
        System.out.printf("检验 %d 个元素 一共用时 %f 秒", insertions, time_sp);
    }

    @Test
    public void testSBF_AddAndContainPerf(){
        long start_time = System.currentTimeMillis();
        StandardBloomFilter<Long> bf = new StandardBloomFilter<>(insertions);
        for (Long number : uniformDataset.add_list) {
            bf.add(number);
        }

        for (Map.Entry<Long, Boolean> pair :
                uniformDataset.check_list) {
            bf.contains(pair.getKey());
        }
        long end_time = System.currentTimeMillis();
        double time_sp = ((double) end_time-start_time)/1000;
        System.out.printf("添加并检验 %d 个元素 一共用时 %f 秒", insertions, time_sp);
    }

    @Test
    public void testSBF_Uniform(){
        StandardBloomFilter<Long> bf = new StandardBloomFilter<>(insertions);
        for (Long number : uniformDataset.add_list) {
            bf.add(number);
        }

        int times = uniformDataset.size, wrong = 0;
        for (Map.Entry<Long, Boolean> pair :
                uniformDataset.check_list) {
            if(bf.contains(pair.getKey())){
                if (!pair.getValue()){
                    wrong++;
                }
            }
        }
        double fpp_real = (double) wrong / (times - uniformDataset.itemsIn);
        System.out.printf("均匀分布下：误判率：%f\t理论误判率%f\n", fpp_real, bf.getFpp());
    }

    @Test
    public void testSBF_Gaussian(){
        StandardBloomFilter<Long> bf = new StandardBloomFilter<>(insertions);
        for (Long number : gaussianDataset.add_list) {
            bf.add(number);
        }

        int times = gaussianDataset.size, wrong = 0;
        for (Map.Entry<Long, Boolean> pair :
                gaussianDataset.check_list) {
            if(bf.contains(pair.getKey())){
                if (!pair.getValue()){
                    wrong++;
                }
            }
        }
        double fpp_real = (double) wrong / (times - gaussianDataset.itemsIn);
        System.out.printf("高斯分布下：误判率：%f\t理论误判率%f\n", fpp_real, bf.getFpp());
    }

    @Test
    public void testSBF_Exp(){
        StandardBloomFilter<Long> bf = new StandardBloomFilter<>(insertions);
        for (Long number : expDataset.add_list) {
            bf.add(number);
        }

        int times = expDataset.size, wrong = 0;
        for (Map.Entry<Long, Boolean> pair :
                expDataset.check_list) {
            if(bf.contains(pair.getKey())){
                if (!pair.getValue()){
                    wrong++;
                }
            }
        }
        if(times - expDataset.itemsIn == 0){
            System.out.println("oops, 所有元素都在里面，没有原先不再的");
        }
        double fpp_real = (double) wrong / (times - expDataset.itemsIn);
        System.out.printf("指数分布下：误判率：%f\t理论误判率%f\n", fpp_real, bf.getFpp());
    }
}
```

输出：

```
高斯分布下：误判率：0.028583	理论误判率0.030004
-------------------------------
添加并检验 10000000 个元素 一共用时 8.798000 秒
-------------------------------
添加 10000000 个元素 一共用时 3.853000 秒
-------------------------------
45 不存在, 但被误判存在
49 不存在, 但被误判存在
55 不存在, 但被误判存在
243 不存在, 但被误判存在
309 不存在, 但被误判存在
321 不存在, 但被误判存在
361 不存在, 但被误判存在
643 不存在, 但被误判存在
723 不存在, 但被误判存在
901 不存在, 但被误判存在
在 1000 次查询中，有 10 次误判，误判率为 0.020000
理论误判率为：0.010143159512273257
-------------------------------
在 10000000 次查询中，存在 296216 次误判，误判率为 0.029622
理论误判率为：0.03000441120280409
-------------------------------
检验 10000000 个元素 一共用时 3.907000 秒
-------------------------------
均匀分布下：误判率：0.030034	理论误判率0.030004
-------------------------------
指数分布下：误判率：0.027511	理论误判率0.030004
-------------------------------

进程已结束,退出代码0

```



## 布隆过滤器变体

- 可以删除的布隆过滤器：[Count Bloom Filter（CBF）](Count%20Bloom%20Filter（CBF）.md)
- 可以变长的布隆过滤器：
- Shifting Bloom Filter：
	- [A Shifting Bloom Filter Framework for Set Queries](ShiftingBF/A%20Shifting%20Bloom%20Filter%20Framework%20for%20Set%20Queries.md)
	- [Shifting Bloom Filter For Membership Query](ShiftingBF/Shifting%20Bloom%20Filter%20For%20Membership%20Query.md)
- Spatial Bloom Filters：[Spatial Bloom Filters](Spatial%20Bloom%20Filters.md)
