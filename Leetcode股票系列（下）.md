# 今天看一下股票衍生版本
# 一、手续费；二、冷却期。

[309. 买卖股票的最佳时机含冷冻期](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

[714. 买卖股票的最佳时机含手续费](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

记的我们上一期的核心递推方程，这两个题目都是无限次交易，可以把外层k循环去掉。应该直接沿用上期推出的无限次交易方程：

- buy[n]=max(buy[n-1], sell[n-1]-price[n])
- sell[n]=max(sell[n-1], buy[n-1]+price[n])

## 手续费
先考虑手续费的情况，根据题意，手续费会影响我们出售时候得到的收益。我们可以把手续费项目添加在sell时获得的收益里面。sell的时候，要么沿用之前的sell，要么本次sell获得price[n]-fee的收益。也就是方程改为：

` sell[n]=max(sell[n-1], buy[n-1]+price[n]-fee)`

继续把之前无限次交易的代码拿来，略微修改：**（就是加一个fee）**
```java 
for(int n=0;n<prices.length;n++){
    buy[n+1]=Math.max(buy[n],sell[n]-prices[n]);
    //只有这里，调整一下，把手续费加进去即可
    sell[n+1]=Math.max(sell[n],buy[n]+prices[n]-fee);
}
```
## 冷却期
我们之前的状态只有两种，也就是【买入态-可以卖】和【卖出态-可以买】，但这个场景下就不够了，因为还有一种**【当前是卖出态，但是不可以买-冷却期】**。
我们需要调整一下对应关系：

- sell = 卖出态（但需要继续拆分为，当前可买，和当前不可买）
   - sell = 卖出态，当前不可买，由buy演变而来
   - empty = 卖出态，当前可买，由sell演变而来
- buy = 买入态 = 随时可卖，与之前的定义是一样的。
- 所以我们的演变关系：sell->empty->buy->sell

在原来方程上：

- buy[n]=max(buy[n-1], sell[n-1]-price[n])
- sell[n]=max(sell[n-1], buy[n-1]+price[n])

我们要做一下调整：

- sell还是来自buy，只要买入了，就随时可卖
- **sell[n]=max(sell[n-1], buy[n-1]+price[n])**
- 但是buy，就不是来自sell了，而是只能来自empty
- **buy[n] = max(buy[n-1], emtpy[n-1]-price[n])**
- 那empty又是怎么来的——卖出以后冷却m天就是empty
- **empty[n] = max(empty[n-1], sell[n-m])**

有了递推方程，写动态规划的代码就易如反掌：
```java
for(int n=0;n<prices.length;n++){
    buy[n+1]=Math.max(buy[n],empty[n]-prices[n]);
    sell[n+1]=Math.max(sell[n],buy[n]+prices[n]);
    empty[n+1]=Math.max(empty[n], sell[n]);
}
```
## 每次这里都会想一下
综合所有股票交易题目：
> 最多交易K次，每次卖出冷冻期P天，每次买入持有期Q天，交易手续费F，这样得用什么算法来推理啊～

- 简单想一下我们的演变关系：buy->hold->sell->empty
- 由于有次数限制，得轮流计算这几个数组
- 大约伪码如下：(也不知道对不对，大概这么个意思)
 

```java
buy[],hold[],sell[],empty[]
//开始计算，k=最多交易次数，n=第几天
for(int i=0;i<k;i++){//最多进行的第几次交易
    for(int n=0;n<prices.length;n++)//empty->buy
        buy[n+1]=max(buy[n],empty[n]-prices[n]);
    for(int n=0;n<prices.length;n++)//buy->hold
        hold[n+1]=max(hold[n],buy[n-Q]);
    for(int n=0;n<prices.length;n++)//hold->sell
        sell[n+1]=max(sell[n],hold[n]+prices[n]-F);
    for(int n=0;n<prices.length;n++)//sell->empty
        empty[n+1]=max(empty[n],sell[n-P]);
}
```
