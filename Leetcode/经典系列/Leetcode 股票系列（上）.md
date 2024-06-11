股票系列经典题目：
> 给你一个整数数组 prices 和一个整数 k ，其中 prices[i] 是某支给定的股票在第 i 天的价格。
设计一个算法来计算你所能获取的最大利润。你最多可以完成 k 笔交易。也就是说，你最多可以买 k 次，卖 k 次。

本题目是经典的【动态规划】。为什么？因为第K次交易，只和K-1次交易有关，和第K-2次交易是无关的（无后效性）；然后第N天的交易状态，只和N-1天的交易状态有关，和之前的也无关；是典型的多维动态规划问题。
## 1.先对题目进行大体分析
【卖】才能得到收益；但是想【卖】就必须先【买】（不然卖啥）。

- 对于【买】来说，就是减去当前price[n]
- 对于【卖】来说，就是加上当前price[n]

也就是说当我有十元，以5元买入，我还有5元；以3元卖出，我就还有8元。
:::warning
举个🌰🌰第五天我要出售，那么在第四天之前我必须有一笔买进才行（不然卖啥）。但是注意，第五天卖出的股票，不一定是第四天买入的，也许是第三天、第二天或者第一天就买入的。所以，第n天要出售，只需要第n-1天是买入状态（注意是状态）即可。
:::
反过来也一样，由于题意我们不能同时买多个股票，所以第n天想买入，那么第n-1天至少要是卖出（可买入状态）才行，同理，也可以是n-2天卖出，n-3天卖出等等。
## 2.看一下单次交易（最基础的情况）
我们只做一次交易，也就是买一次，卖一次，那我们可以用两个数组：
:::info

- buy[]：第n天执行过买入时的最大金额
- sell[]：第n天执行过卖出后的最大金额
:::
对于buy[]数组来说，我们可以计算buy[n]：

- 要么是第n天买入，那么总金额S=0-price[n]；
- 要么是之前已经买入，第n天什么都不做，那么S不变,S=buy[n-1]；
- 所以buy[n] = max(-price[n], buy[n-1])

同理对于sell[]来说，sell之前必须买入，也就是想计算sell[n]需要借助buy[n-1]

- 要么当前卖出，那么总金额S = buy[n-1] + price[n];
- 要么之前已经卖出，第n天什么都不做，S不变S=sell[n-1]
- 所以sell[n] = max(buy[n-1] + price[n], sell[n-1])

![image.png](https://cdn.nlark.com/yuque/0/2024/png/22367711/1715071855003-6aa1acdb-436d-460e-b019-d18634b2d66e.png#averageHue=%23eae6e2&clientId=uf9a6adae-bd10-4&from=paste&height=294&id=u9eea57d3&originHeight=588&originWidth=918&originalType=binary&ratio=2&rotation=0&showTitle=false&size=309041&status=done&style=none&taskId=ue3367bed-6fb9-4b71-af07-91fee513d48&title=&width=459)
## 3.再看无限次的情况（另一种基础）
如果无限多次交易，那么buy就不能一次性推理出来，因为每一个buy都有可能与上一个sell相关，也就是在上一次交易结束之后的S的基础上继续累加。继续沿用之前的定义：

- sell[]：第n天执行过买入时的最大金额
- buy[]：第n天执行过卖出后的最大金额

我们上一次计算buy的时候，直接令buy[n]=0-price[n]；这是因为单次场景下买入的前提永远是0。但是在无限次场景下，这个‘0’，就已经不再适用。
但是依然简单，对于buy[n]来说

- 第n天买入，也就是n-1天已经卖出了，在sell[n-1]的基础上继续累加，那么总金额S=sell[n-1]-price[n]；
- 之前已经买入，第n天还是什么都不做，那么S不变,S=buy[n-1]；
:::info

- 所以有buy[n]=max(buy[n-1], sell[n-1]-price[n])
:::
相信有了这个推理，sell的数组也容易得出：
:::info

- sell[n]=max(sell[n-1], buy[n-1]+price[n])
:::
我们拿示例数组进行推理，如图所示：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22367711/1715073929767-b944d897-ffe7-4ec6-8f22-427bdc4ca656.png#averageHue=%23e8e4e0&clientId=uf9a6adae-bd10-4&from=paste&height=277&id=u3e6ded8d&originHeight=554&originWidth=784&originalType=binary&ratio=2&rotation=0&showTitle=false&size=269755&status=done&style=none&taskId=u9933fc54-7339-4285-9ec1-88ff67f149e&title=&width=392)
代码整理为：
```java
for(int n=0;n<prices.length;n++){
    buy[n+1]=Math.max(buy[n],sell[n]-prices[n]);
    //只有这里，调整一下，把手续费加进去即可
    sell[n+1]=Math.max(sell[n],buy[n]+prices[n]);
}
```
## 4.有限多次交易的情况（复杂）
首先我们先对比一下单次和无限次的差别：一个是buy和sell数组的轮流计算，一个是穿插计算。穿插计算buy和sell的问题是，无法限制交易次数。
**_有限次交易的核心：第K次交易只能在K-1次交易的基础上发生_**
如果k=1，就是单次交易，始终牢记我们之前推理出的公式：
:::info

- buy只和buy相关，sell只和当前buy相关
- buy[n]=max(buy[n-1], 0-price[n])
- sell[n]=max(sell[n-1], buy[n-1]+price[n])
:::
再来看k=2的情况，前面已经分析过，由于buy需要之前卖掉才能买，所以**第二次的buy和建立在第一次卖出的基础上的**。也就是可以通过第一次交易的数组推理出第二次交易的情况。具体来说就是，计算buy2数组，需要知道sell1数组。
> **_所以最核心的计算逻辑：buy0->sell0->buy1->sell1->buy2->sell2->..._**

我们可以把原本的buy[]和sell[]扩充，改为buy[k][n]和sell[k][n]二维数组。抽象地来说，buy[k][n]和上次sell[k-1][n-1]相关，而sell[k][n]和当次的buy[k][n-1]相关。调整一下我们的递推方程，把**【可进行的交易次数k】**也加进来：k代表最大交易次数，n代表天数

- buy[k][n]=max(buy[k][n-1], sell[k-1][n-1]-price[n])
- sell[k][n]=max(sell[k][n-1], buy[k][n-1]+price[n])

我们先演算Buy2的情况：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22367711/1715059067410-87ba3280-e258-4b5e-9d1f-a23a71239728.png#averageHue=%23ebdcaf&clientId=u8ba655c5-dbc9-4&from=paste&height=342&id=ucf23de8e&originHeight=684&originWidth=780&originalType=binary&ratio=2&rotation=0&showTitle=false&size=361809&status=done&style=none&taskId=u7e865455-6fc4-42c5-810d-ad116e3d0d9&title=&width=390)
再演算Sell2的情况
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22367711/1715059080693-d02f98ee-be11-4d21-b8b1-f14c4c785cc4.png#averageHue=%23ece0bd&clientId=u8ba655c5-dbc9-4&from=paste&height=230&id=u4e79968b&originHeight=460&originWidth=732&originalType=binary&ratio=2&rotation=0&showTitle=false&size=229916&status=done&style=none&taskId=u8b118dcf-1521-4a30-8498-dfc7741d5a9&title=&width=366)
最后我们得到结果，sell2的最后一位（7）就是我们所求的，最多交易k次的最大值。
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22367711/1715059096657-6bd7340c-9923-4159-baca-6a6be7866e80.png#averageHue=%23eae3dc&clientId=u8ba655c5-dbc9-4&from=paste&height=101&id=uf17fe53a&originHeight=202&originWidth=410&originalType=binary&ratio=2&rotation=0&showTitle=false&size=53848&status=done&style=none&taskId=u7783bc36-da5d-4a7c-a53f-8eec9d880ea&title=&width=205)
这样可以提交代码上去了，但是，到此还没完。
## 5.我们要额外注意（状态压缩）
根据我们上面的推理和演算，buy[k]的计算，只用到了sell[k-1]，sell[k]的计算只用到了buy[k]。事实上我们可以做一步【状态压缩】。
也就是我们不再用buy[k][n]这样的二维矩阵，因为这里存储了很多没必要的数据（计算buy3不需要知道buy1、sell1）。因为**_最核心的计算逻辑：buy0->sell0->buy1->sell1->buy2->sell2->..._**
根据上面的计算，**sell2是不需要buy1的**，我们仍然只采用一个buy[n]和sell[n]来进行数据存储。每次计算出新的buy[n]，再根据buy[n]计算新的sell[n]，再根据新的sell[n]来计算新的buy[n]，往后循环即可。
再次拿出我们的推理公式：
:::info

- buy[n]=max(buy[n-1], sell[n-1]-price[n])
- sell[n]=max(sell[n-1], buy[n-1]+price[n])
:::
直接改造为代码即可：
```java
//数据初始化
int[] buy = new int[prices.length + 1];//+1是为了空出第0天，计算方便
int[] sell = new int[prices.length + 1];//+1同理
buy[0] = Integer.MIN_VALUE;//这里把buy0设置为极端最小值

//开始计算，k=最多交易次数，n=第几天
for(int i=0;i<k;i++){//最多进行的第几次交易
    //先计算buy，只用到前一次的sell，就是下面的for所计算出的结果
    for(int n=0;n<prices.length;n++){
        buy[n+1]=Math.max(buy[n],sell[n]-prices[n]);
    }
    //再计算sell，只用到当前次的buy(就是上面的for所计算出的结果)
    for(int n=0;n<prices.length;n++){
        sell[n+1]=Math.max(sell[n],buy[n]+prices[n]);
    }
}
```
## 相关题目
有了188的算法，可以直接带入前面三个题目，k=1，k=len，k=2即可
> [121. 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)（单次交易）
> [122. 买卖股票的最佳时机 II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)（任意次交易）
> [123. 买卖股票的最佳时机 III](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iii/)（最多2次交易）
> [188. 买卖股票的最佳时机 IV](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iv/)（最多K次交易）

再次衍生：（下一期讲）
> [309. 买卖股票的最佳时机含冷冻期](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/)
> [714. 买卖股票的最佳时机含手续费](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)


