# Aminer关键词场景的推荐系统
作者：槐泽鹏

# 产品介绍
首页地址：https://www.aminer.cn/

AMiner是一个学术信息检索网站，对标谷歌学术和arXiv。提供“搜索”和“推荐”两种信息检索服务。

首页最上方是一个大搜索框，下面是推荐模块。

推荐包含两个场景：综合推荐和关键词推荐(区别:是否输入关键词)。如下图所示。

![image](https://github.com/huaizepeng2020/Aminer-recommender-system-with-keywords/blob/main/figure/introduction1.png)

综合推荐是用户无需任何输入，直接推荐内容。
（例如某用户浏览过“搜推广”的文章，则推荐信息检索方向的信息；
但近期综合推荐的同学调整了冷启动策略，可能部分用户不推paper，只给推荐新闻类等不太动脑的直观的信息）

**关键词推荐场景**<br />
用户输入学术关键词(如knowledge graph/contrastive Learning)<br />
→返回这个方向中用户可能感兴趣的paper、科技资讯、topic(目前支持三类“商品”)

# 我的工作
我负责关键词推荐场景的推荐系统：包含算法及工程开发两个方面。

推荐系统示意图如下图所示。
![image](https://github.com/huaizepeng2020/Aminer-recommender-system-with-keywords/blob/main/figure/introduction1.png)

# 算法
一个完整的推荐系统包含“特征工程→召回→排序”三个基本组成，Aminer关键词场景的推荐系统也不例外。

# 工程开发
特征工程：

运行环境： Python >3 

requirenments：
pandas，sklearn，matplotlib，numpy

# 召回
1 不需在联网环境下获取数据

2 LDF、QDF、RDA、MQDF方法调用接口为gaussianlinear.py 
  
3 parzen windows 方法接口文件为bayes.py
