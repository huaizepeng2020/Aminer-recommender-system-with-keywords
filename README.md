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

推荐系统示意图如下图所示。（该图片较大共12.5M，感兴趣可下载图片到本地放大查看，分辨率保存时设置的较高，即放大后画质无损）
![image](https://github.com/huaizepeng2020/Aminer-recommender-system-with-keywords/blob/main/figure/Aminer_keywords_RS_2021.12.11_wrapper.jpg)

## 算法
一个完整的推荐系统包含“特征工程→召回→排序”三个基本组成，Aminer关键词场景的推荐系统也不例外。

### 特征工程
包含3方面：

1交互记录<br />
输入是网页的埋点日志，采用“时光机”架构，即每天的交互记录保存为一份数据，以日期为key存储在ssdb里面。<br />
2用户编码<br />
将注册用户的原始ID(Mongodb里的"_id"字段)one-hot编码，每天更新一次，并且旧用户的编码不变，新用户的顺延递增。清洗好后存入SSDB。<br />
3商品特征<br />
包含paper的召回池(按引用量筛选，≥10的共84万篇)，独热编码，引用量特征，发表年份特征。清洗好后存入SSDB。<br />

### 召回
1将召回池中所有paper的title，分词，去常用/停用词，将每个title视为一个序列，采用**word2vec**训练，得到一个word2vec词表。(key: keyword;value:embedding)<br />
2用此词表对召回池每篇文章title分词后编码，最终一篇文章包含多个embedding_nlp(以下简称e_nlp)，每个向量代表title中的一个词<br />
3在线服务时，基于此词表对输入关键词编码，得到embedding_c(简称e_c),再在所有e_nlp空间里，基于**局部敏感哈希(lsh)**作近邻搜索，召回~10^4篇paper。<br />
**一个trick**：<br />
分完词之后将每个词转化为了词根(nltk.stem.wordnet.WordNetLemmatizer.lemmatize(w, 'v'))，这样扩大了词表覆盖范围。<br />
例如recommendation和recommender在word2vec词表里就是一个。<br />

### 排序
最终排序得分包含四个部分：<br />
score_final=LR(score1,score2,score3)-score4 <br />
具体如下：<br />

1 基于兴趣的打分(CF-based score1)<br />
采用**lightGCN**训练user&item embedding_cf(简称e_cf)。在线服务时直接对user和target item作点积，并归一化到0-1范围内。<br />

2 基于内容的打分(content-based score2)<br />
采用**word2vec+attention**的框架。<br />
上述提到，word2vec把每篇文章编码了多个e_nlp，<br />
以e_c为query，对target item的多个e_nlp根据向量内积+softmax得到权重，再加权求和得到e_nlp_t。<br />
然后将e_c和e_nlp_t作点积，并归一化到0-1范围内。<br />

3 基于用户序列的打分(behavior-based score3)<br />
采用**DIN**训练神经网络参数及user&item embedding_din(简称e_din)。在线服务时输入用户和商品特征在线排序。<br />

4 **impression discount**打分(score4)<br />
对于曝光过的paper，我们期望不要再曝光，要降低此类商品打分。<br />
根据埋点日志实时将(user,paper)的曝光次数存入redis，<br />
在线服务的时候先从redis取出user和target item的曝光次数，再乘以一个系数，最后归一化到0-1范围内。<br />

注：lightGCN和DIN在阿里云A100服务器上每天训练一次，训练后的数据(包含word2vec词表，e_cf，DIN网络参数及e_din)传输到生产环境，以供在线服务API启动时预加载。

## 工程开发
分为四个方面：数据、API、时间&存储开销、高并发场景。<br />
### 数据
1搭建了主从SSDB。特征工程中数据全部存入SSDB和Redis，并且能存成KV的都存成了KV。<br />
2搭建阿里云A100服务器和生产环境的vpn和数据传输通道。<br />
### API
1暴露给架构的最外层API1是基于surpervisord启的，底层是Django。<br />
2排序中的DIN打分是基于torchserve起的API2。
### 时间&存储开销
1 从请求到来到返回推荐结果(API1)开销在150ms以下。其中主要花费时间项：局部敏感哈希的近邻搜索约50ms/DIN排序50约120ms/有道翻译40ms<br />
2 surpervisord起了3个线程，每个占9.5g，总共占生产环境22%的内存。
### 高并发场景
1为提高高并发能力，采用了异步召回和多线程多协程的逻辑。<br />
2使用locust压测，RPS在64。

# 说明：<br />
1召回和传统推荐系统不同，没有考虑交互数据。由业务场景决定的。

# 结果
在线点击率从初期3-6%逐渐上升至6-9%再到10%+。<br />
几个重要的时间节点：<br />
1召回改为现在的算法<br />
2使用了word2vec+attention的架构<br />
3排序中引入impression discount

# 个人心得
1 从0到1完整的搭建了一个推荐系统。<br />
在AMiner实习之前，甚至还分不清召回和排序有什么区别。经过此次实习，踩了很多坑，跌跌撞撞的搭建了完整系统，积累了很多技术上的个人体会。<br />

2 算法的顶层设计真的很重要，要“简单、实用、可扩展”。现在用的是V2版本，V1版本召回是基于微软MAG学术图谱作粗分类，再在每个类别下召回，繁琐效果差。

3 高大上的模型不一定work，要根据业务场景具体的分析数据和设计模型。

4 对于数据挖掘/信息检索来说，洗数据重要但也真的很耗时间.

5 Let data talk是一种思维方式。

