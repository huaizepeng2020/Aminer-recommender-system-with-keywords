# Aminer关键词场景推荐系统
作者：槐泽鹏

# 产品介绍
首页地址：https://www.aminer.cn/

AMiner是一个学术信息检索网站，对标谷歌学术和arXiv。提供“搜索”和“推荐”两种信息检索服务。

首页最上方是一个大搜索框，下面是推荐模块。

推荐包含两个场景：综合推荐和关键词推荐(区别:是否输入关键词)。如下图所示。

![image](https://github.com/huaizepeng2020/Aminer-recommender-system-with-keywords/blob/main/figure/introduction1.png)

综合推荐是用户无需任何输入，直接根据其历史记录来推荐内容。（例如某用户浏览过“搜推广”等文章，则推荐信息检索方向的文章）

关键词推荐场景是：
用户输入学术关键词(如knowledge graph/contrastive Learning)        
→返回这个方向中用户可能感兴趣的paper、科技资讯、topic(目前支持三类“商品”)

原来代码有两处错误：

1 判别函数少了“协方差矩阵行列式”此项。

2 MQDF算法中计算修正的协方差矩阵时，直接输出了原矩阵，并且代码逻辑较为混乱，因此进行了重新改写。

# 内容
实现四种参数法高斯分类器LDF、QDF、RDA、MQDF和一种非参数分类器parzen windows。

运行环境： Python >3 

requirenments：
pandas，sklearn，matplotlib，numpy

# 使用
1 不需在联网环境下获取数据

2 LDF、QDF、RDA、MQDF方法调用接口为gaussianlinear.py 
  
3 parzen windows 方法接口文件为bayes.py
