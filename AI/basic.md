# AI

## 各种原则
### 金发姑娘原则
* [相关链接](https://en.wikipedia.org/wiki/Goldilocks_principle)
* 解释“金发姑娘原则”指出，凡事都必须有度，而不能超越极限。按照这一原则行事产生的效应就称为“金发姑娘效应”。

### 学习速率
* 通过一个图来解释
    ![优化学习速率](https://i.loli.net/2019/07/18/5d3002110b29b87060.png)

* 当「学习速率」过高，容易导致每一步都在曲线上进行跳跃，沿着曲线向上爬，而不是降到底部。


### 随机梯度下降法/ SGD

### 小批量随机梯度下降法/小批量 SGD

### 协同过滤
* 基于用户的协同过滤算法 UserCF
    * 统计两个用户的相似度。两个用户阅读过的内容 ID 列表如果重合度越高，说明越相似。
    * 适合用在个性化需求不强，热点很明显的领域，比如新闻，电影推荐。
* 基于物品的协同过滤算法 ItemCF
    * 适合用在个性化需求比较强，长尾比较长的领域，比如书、电商的推荐。

### 如何判断用户会不会点一个物品：特征
* 用户特征
    * 历史上点击的文章列表
    * 历史上点击的文章的关键词分布
    * 历史上点击的文章的作者分布
* 文章特征
    * 文章的作者，关键词
* 用户和文章的交叉特征
    * 用户历史上有没有点过这篇文章的作者发表的其他文章
    * 用户历史上有没有点击过和这篇文章关键词类似的其他文章

### 文章冷启动
* 推荐系统是所有的文章在一个候选池互相 PK，找到对当前用户兴趣最好的
* 新文章因为展现少，在 PK 中往往落在下风
* 要做一个专门的冷启动机制，保证新文章在展现小于 X 前在 PK 中取得更大的获胜概率，直到充分展现，可以在 PK 中公平竞争