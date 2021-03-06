+++
Description = "面试 问题"
Tags = ["面试", "问题",]
date = "2018-09-23T15:18:40+08:00"
title = "几个有趣的面试概率问题汇总"
+++


最近经历了一些面试，总结了一些有意思的两个问题。他们用到的概率只是都很简单。但是想不到就是想不到。

<!--more-->
## 蓄水池抽样

百度面试的时候问了我这样一个问题，如何对一个数据流进行采样，保证每一个元素而言他们采样的概率都是均等且内存占用比较小。
面试的时候问这个问题我是懵的，只是想到了采用两个数值相消来保证均等概率的思路，但是具体如何操作我是一点头绪都没有。面试官看我不会，也和我说了方案让我写代码，方案其实很简单。。。

后来下来查了一下，这个在大数据中叫做[`蓄水池抽样 Reservoir sampling`](https://en.wikipedia.org/wiki/Reservoir_sampling)问题。

先说一下算法过程，维基百科对这个算法描述比较详细。

假设我们需要抽样10个元素，那么我们的做法如下：

* 对于前10个元素而言，我们直接把它保留在内存中
* 对于10个元素之后的第i个元素
    + 以10/i的概率来保证它留下，如果需要替换，那么就从10个元素中随机的选择一个元素来进行替换。
    + 对于已有的老元素而言，以1-10/i的概率来保留

下面分析一下概率：

* 一开始，前10个元素被选中的概率就是1，因为里面没有啥元素
* 来了第11个元素后，第11个元素被保留下的概率是(10/11) (我们以(10/11)的概率决定元素留下)，而对于蓄水池已经有的元素，他们被留下的概率为 1 * (1/11 + (10/11)*(9/10)) = 1/11 + 9/11 = 10/11，1为前10个元素的概率，(1/11)为第11个元素没有进行替换的概率，(10/11)*(9/10)为第11个元素决定替换，但是该元素被保留下来的概率。所以综上，前11个元素中，每一个元素被留下的概率均为10/11
* 到第12个元素的时候，同样，我们决定它被保留的概率是10/12，而蓄水池中已有的元素，他们被保留的概率是(10/11)*(2/12 + (10/12)*(9/10)) = (10/11)*(11/12) = 10/12， (10/11)为上一把中这个元素被保留的概率，(2/12)为第12个元素没有被保留的概率，(10/12)*(9/10)为第12个元素保留了，但是没有被替换的概率。

具体算法非常简单，伪代码如下：
```C++
//Pseudocode
class ReservoirSampling {
public:
    //Sampling k elements
    ReservoirSampling(int k) {
        m_k = k;
        m_reservoir = new int(k);
        m_count = 0;
    }

    void add(int number) {
        if(m_count < m_k) {
            m_reservoir[m_count] = number;
        }else{
            int index = rand(0, m_count); //generator [0,m_count]
            if index < m_k {
                m_reservoir[index] = num;
            }
        }

        m_count++;
    }

private:
    int m_k;
    int m_count;
    int *m_reservoir;
};
```

看起来非常简单了，这里面的概率分析如下：

* 对于第k+1个元素而言，其留在蓄水池中的概率为(k/k+1)。蓄水池中每一个元素被替换的概率为(k/k+1) * (1/k) = (1/(k+1))，那么其留在蓄水池中的概率为(1 - (1/(k+1))) = (k / (k + 1))
* 对数据流中的第i个元素而言，其留在蓄水池中的概率为(k/i)。且蓄水池中的每一个元素被替换的概率为(k/i) * (1/k) = (1/i)。则留在蓄水池中的概率为(1-(1/i)) = ((i-1)/i)
* 所以对于数据流中的第i+1个元素而言，用P(i+1)表示留下的概率其留下的概率P(i+1) P (k+1) * P(k+2) * ... * P(i+1) = (k/(i+1))

这就得到了一个稳定的概率，保证了相同概率选取。

维基百科里面还有更多东西，包括分布式的蓄水池，有兴趣大家可以深入阅读。

## 网易云音乐随机算法
某条问了我个问题，怎么确保网易云音乐随机算法中所有歌曲被随机到的概率都是一样的。

我第一反应是每次都把歌单shuffle一下，听完所有的歌曲又shuffle下。这样都是随机的。但是面试的时候忘了shuffle算法了，囧。

计算机中的Shuffle算法叫做[Fisher–Yates shuffle](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle)算法。其主要作用是来将一个长度为n的元素打乱，使得每个元素在固定位置出现的概率均等。

其伪代码描述如下：
```
-- To shuffle an array a of n elements (indices 0..n-1):  
for i from n−1 downto 1 do  
     j ← random integer such that 0 ≤ j ≤ i  
     exchange a[j] and a[i]  

```


* 第1个循环中，第n-1个元素数被保留的概率为1/n
* 第2个循环中，第n-2个元素被保留的概率为((n-1)/n) * (1/(n-1)) = (1/n)。
* 第三个循环，第n-3个元素被保留下的概率为((n-1)/n) * ((n-1)/(n-2)) * (1/(n-2)) = (1/n)

依此类推，循环完成后每个元素留下的概率都是一样的。

数学还是绕不开的大坑。。