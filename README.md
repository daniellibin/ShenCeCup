# ShenCeCup
A competition on DataCastle which is about text keyword extraction ! Rank 6 / 622 !  
[“神策杯”2018高校算法大师赛](http://www.dcjingsai.com/common/cmpt/%E2%80%9C%E7%A5%9E%E7%AD%96%E6%9D%AF%E2%80%9D2018%E9%AB%98%E6%A0%A1%E7%AE%97%E6%B3%95%E5%A4%A7%E5%B8%88%E8%B5%9B_%E7%AB%9E%E8%B5%9B%E4%BF%A1%E6%81%AF.html)是一个只能高校在校生solo的单人赛。神策数据提供了10万篇左右资讯文章的标题以及正文，其中一千篇文章有对应的标注数据。标注数据中每篇文章的关键词不超过5个，关键词都在文章的标题或正文中出现过。根据已有的数据，需要训练一个“关键词提取”模型，提取没有标注数据的文章的关键词，每篇文章最多提交两个关键词。  
最终排名：6 / 622  
附：[github地址](https://github.com/RHKeng/ShenCeCup)  
##比赛简介（Datacastle）  
（1）比赛介绍：比赛根据神策数据提供的一千篇资讯文章及其关键词，参赛者需要训练出一个”关键词提取”的模型，提取10万篇资讯文章的关键词。  
（2）评价指标：提交的预测结果中，每篇文章最多输出两个关键词。预测结果跟标注结果命中一个得 0.5 分，命中两个得一分。英文关键词不区分大小写。  
##问题分析  
通过初步分析，本次比赛训练集和测试集的样本比例大致是1:100，因此选择采用无监督的模型（tfidf/tfiwf，textRank，主题模型LSI/LDA）进行关键词提取。依据比赛背景，我将其分成两个步骤，首先根据资讯文章和标题选取关键词候选集，然后选择其中两个概率最大的关键词。  
##数据分析  
（1）训练集和测试集的样本比例1:100  
（2）通过分析标注数据以及标题的关系可以看出，1000篇标注文章中，只有20篇左右文章的关键词是不在标题中，而且80%左右文章至少有两个关键词是在标题中，可以看出标题的重要性。大家看到一篇资讯文章，通常会首先关注标题，因为标题会概括出这个文章的主要内容。  
![image.png](https://upload-images.jianshu.io/upload_images/12207295-7843de4c86dab39f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
（3）通过分析标题中的数据可以看到，如果标题中含有书名号或者是人名，95%以上都是对应文章的关键词。这个应该跟每个人的习惯（打标签人的习惯）有关，书名号中的内容通常会对应影片，电视剧，歌曲等的名称，这些名词和人名很大概率会在一开始吸引我么的注意，因此是关键词的概率就很大。  
##样本构造  
由于采用的是无监督模型，因此，可以我把官方提供的一千条标注样本作为线下验证集，来验证模型的精度和效果，基本上可以做到线上线下一致。而对于线上提交结果，我将一千条标注数据的标签作为结巴分词的自定义词，用以提高分词的准确度。  
##数据预处理  
###分词预处理过程  
1. 对于jieba分词，去除了一些常用的停用词（从网上找的），避免后期一些停用词对模型精度产生影响，停用词主要包括英文字符、数字、数学字符、标点符号及使用频率特高的单汉字等。比如的，呢。  
2. 将一千条标注数据的标签作为结巴分词的自定义词，用以提高分词的准确度。主办方有说明过，“训练集文章的关键词构成的集合”与“测试集文章的关键词构成的集合”，这两个集合可能存在交集，但不一定存在包含与被包含的关系。  
3. 在网上找到搜狗的词典，给jieba分词添加用户词典，提高分词准确度。  
4. 利用哈工大ltp分词接口进行分词，同样作为模型样本，用于弥补jieba分词的分词错误。  
5. 利用ltp的接口，同时识别jieba分词和ltp分词的结果中的名词甚至是人名，用于后期的规则。  
###增大title中词语的权重  
在这里，采用的是简单的title复制的方式，对于一条样本，利用句号将n个相同的title和context进行字符串拼接，然后分词并用于后期tfidf的计算，这样就可以达到增大title中词语的权重。这里n的确定，每一条样本的n是不一样的，依据context中句子的个数乘上一定的比例来确定n。（通过训练集，也就是我的线下验证集来确定比例，这个比例，从实际场景来说，就是人们对title关注度的反映）  
##模型选择  
对比无监督的模型（tfidf/tfiwf，textRank，主题模型LSI/LDA）的效果，最终采用tfidf作为基础模型进行关键词候选集的选取。  
###tfidf  
tfidf（词频-逆文档频率）算法是一种统计方法，用以评估一字词对于一个文件集或一个语料库中的其中一份文件的重要程度。字词的重要性随着它在文件中出现的次数成正比增加，但同时会随着它在语料库中出现的频率成反比下降。  
TF（词频）就是某个词在文章中出现的次数，TF（词频） = 某个词在文章中出现的次数 / 该篇文章的总词数；IDF（逆向文档频率）为该词的常见程度，IDF 逆向文档频率 =log (语料库的文档总数 / (包含该词的文档总数+1))，如果一个词越常见，那么其分母就越大，IDF值就越小。  
###Tfiwf  
TF不变，IWF是文档所有词语词频之和/该单词词频  
![image.png](https://upload-images.jianshu.io/upload_images/12207295-7cabb48ad86b2a5b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
###Pagerank（此处列出只为引出下面的textrank）  
需要知道有哪些网页链接到网页A，也就是要首先得到网页A的入链，然后通过入链给网页A的投票来计算网页A的PR值。这样设计可以保证达到这样一个效果：当某些高质量的网页指向网页A的时候，那么网页A的PR值会因为这些高质量的投票而变大，而网页A被较少网页指向或被一些PR值较低的网页指向的时候,A的PR值也不会很大，这样可以合理地反映一个网页的质量水平。  
![image.png](https://upload-images.jianshu.io/upload_images/12207295-dbac2aea7ea1bf4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
Vi表示某个网页，Vj表示链接到Vi的网页（即Vi的入链），S(Vi)表示网页Vi的PR值，In(Vi)表示网页Vi的所有入链的集合,Out(Vj)表示网页Vj链接到其他网页的数量，d表示阻尼系数，是用来克服这个公式中“d *”后面的部分的固有缺陷用的：如果仅仅有求和的部分，那么该公式将无法处理没有入链的网页的PR值，因为这时，根据该公式这些网页的PR值为0，但实际情况却不是这样，所有加入了一个阻尼系数来确保每个网页都有一个大于0的PR值，根据实验的结果，在0.85的阻尼系数下，大约100多次迭代PR值就能收敛到一个稳定的值，而当阻尼系数接近1时，需要的迭代次数会陡然增加很多，且排序不稳定。公式中S(Vj)前面的分数指的是Vj所有出链指向的网页应该平分Vj的PR值，这样才算是把自己的票分给了自己链接到的网页。  
###textrank  
一种用于文本的基于图的排序算法，仅利用单篇文档本身的信息即可实现关键词提取，不依赖于语料库。（调用jieba的接口）  
![image.png](https://upload-images.jianshu.io/upload_images/12207295-b2dcbd229703e1ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
Wji是指Vi和Vj两个句子之间的相似度，可以采用编辑距离和余弦相似度等。当textrank应用到关键词提取时，与自动摘要提取不同：1）词与词之间的关联没有权重，即Wji是1；2）每个词不是与文档中所有词都有链接，而是通过设定固定长度滑动窗口形式，在窗口内的词语间有链接。  
###主题模型  
主题模型认为在词与文档之间没有直接的联系，它们应当还有一个维度串联起来，这个维度就是主题。主题模型就是一种自动分析每个文档，统计文档内词语，根据统计的信息判断当前文档包含哪些主题以及各个主题所占比例各为多少。主题模型是一种生成模型，一篇文章中每个词都是通过“以一定概率选择某个主题，并从这个主题中以一定概率选择某个词语”这样一个过程得到的；  
![image.png](https://upload-images.jianshu.io/upload_images/12207295-65eb75180a68847d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
主题模型常用的方法是LSI（LSA）和LDA，其中LSI是采用SVD（奇异值分解）的方法进行暴力破解，而LDA则是通过贝叶斯学派方法对分布信息进行拟合。通过LSA或LDA算法，可以得到文档对主题的分布和主题对词的分布，可以根据主题对词的分布（贝叶斯方法）得到词对主题的分布，然后通过这个分布和文档对主题的分布计算文档与词的相似性，选择相似性高的词列表作为文档的关键词。  
####LSA  
潜在语义分析(Latent Semantic Analysis, LSA)，也叫做Latent Semantic Indexing, LSI. 是一种常用的简单的主题模型。LSA是基于奇异值分解(SVD)的方法得到文本主题的一种方式。  
![image.png](https://upload-images.jianshu.io/upload_images/12207295-2cfd2a191c9f362c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
Umk代表了文档对主题的分布矩阵，Vnk的转置代表了主题对词语的分布矩阵。  
LSA通过SVD将词、文档进行更本质地表达，映射到低维的空间，并在有限利用文本语义信息的同时，大大降低计算的代价，提高分析质量。但是计算复杂度非常高，特征空间维度较大的，计算效率十分低下。当一个新的文档进入已有特征空间时，需要对整个空间重新训练，才能得到新加入文档后的分布信息。此外，还存在对频率分布不敏感，物理解释性薄弱的问题。  
####pLSA  
在LSA的基础上进行了改进，通过使用EM算法对分布信息进行拟合替代了使用SVD进行暴力破解。  
PLSA中，也是采用词袋模型（词袋模型，是将一篇文档，我们仅考虑一个词汇是否出现，而不考虑其出现的顺序，相反，n-gram考虑了词汇出现的先后顺序），文档和文档之间是独立可交换的，同一个文档内的词也是独立可交换的。在PLSA中，我们会以固定的概率来抽取一个主题词，然后根据抽取出来的主题词，找其对应的词分布，再根据词分布，抽取一个词汇。  
####LDA  
LDA 在 PLSA 的基础上，为主题分布和词分布分别加了两个 Dirichlet 先验分布。PLSA中，主题分布和词分布都是唯一确定的。但是，在LDA中，主题分布和词分布是不确定的，LDA的作者们采用的是贝叶斯派的思想，认为它们应该服从一个分布，主题分布和词分布都是多项式分布，因为多项式分布和狄利克雷分布是共轭结构，在LDA中主题分布和词分布使用了Dirichlet分布作为它们的共轭先验分布。  
在 LDA 中，主题的数目没有一个固定的最优解。模型训练时，需要事先设置主题数，训练人员需要根据训练出来的结果，手动调参，再优化主题数目。  
![image.png](https://upload-images.jianshu.io/upload_images/12207295-28f6df5a8a28353f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
我们可以根据数据的多项式分布和先验分布求出后验分布，然后将这个后验分布作为下一次的先验分布，不断迭代更新。求解方法一般有两种，第一种是基于Gibbs采样算法求解，第二种是基于变分推断EM算法求解。  
###小结  
模型对比：tf-idf的idf值依赖于语料环境,这给他带来了统计上的优势,即它能够预先知道一个词的重要程度，而textrank只依赖文章本身,它认为一开始每个词的重要程度是一样的，但是用到了词之间的关联性(将相邻的词链接起来)。主题模型LSA和LDA都依赖于语料库，在新的一篇文档进来后需要重新训练，但是主题模型可以充分利用到文本中的语义信息。Tfidf和textrank都可以用jieba的接口，而主题模型可以用sklearn中gensim的接口。  
在我们的本次比赛，虽然说可以看出来整个数据集是有一定的主题的，包括娱乐，体育等，但是从关键词标签来看，这个跟主题名称并没有很大的关联，而是跟标题关联性很大，所以tfidf虽然是简单的统计，但是却可以发挥很大的效果（增大title的权重）。  
###规则  
结合前面的分析，加入一系列人工规则，利用tfidf模型得到的10个关键词候选集确定最终概率最大的两个关键词label（人工规则，其实就是给模型加入人的主观性，有助于提高模型精度）  
1.利用re正则表达式获取title中书名号的内容作为重要度最高的候选集；  
2.利用训练集标签构建keyword_set，利用jieba对title分词结果构建jieba_title_set，将10个候选集中同时存在于keyword_set和jieba_title_set中的作为重要度第二高的候选集；  
3.将10个候选集中同时存在于jieba_title_name_list和ltp_title_name_list中的关键词作为重要度第三高的候选集；  
4.将10个候选集中存在于jieba_title_name_list的关键词作为重要度第四高的候选集；  
5.将10个候选集中位于title内且词性为名词的关键词作为重要度第五高的候选集；  
6.将10个候选集中位于keyword_set的关键词作为重要度第六高的候选集；  
7.将10个候选集中位于title中，词性为非名词的关键词作为重要度第七高的候选集；  
8.其余的候选集作为重要度最低的候选集；  
##赛后总结  
这次我是第一次接触跟文本相关的比赛，所以入门了挺多关于文本处理的操作，包括如何分词，如何做数据预处理（去除停用词，提高分词准确性），如何针对特定问题选择相关的模型作为基础模型（tfidf/tfiwf，textRank，主题模型LSI/LDA），以及怎么针对问题对结果进行优化。这次比赛由于跟其他比赛有重叠，所以用在这上面的时间并不是特别多，在前期从几十名不断优化做到第二之后（640分，总分1000），思路有点短路，然后其他比赛时间也相对紧张，所以后面就很少再做改进了，最终A/B榜都是排名第六，模型还是具有鲁棒性的。答辩过后，看到了其他选手的分享，才知道自己在思路上具有一定的局限性，所以没做到前三（前三采用有监督模型，四到六采用无监督模型），下面来总结一下本次比赛的不足以及其他选手的亮点。  
（1）由于是单人赛，而且没有跟其他选手或朋友交流，在一定思路做到极限后没有开拓新的思路，这个确实比较可惜。这次比赛区分答辩选手前后排的根本是，采用的是有监督模型还是无监督模型。官方后面的解释，他们是想引导选手从无监督的角度来做，所以测试集的样本数远远大于训练集的数量，而且训练集的数量只有1000条，因为神策公司是要借鉴选手的模型落地到实际的产品中，也对实时性有一定的要求，此时无监督模型可以在保持一定精度的前提下大大减少训练和预测的时间，有助于算法的落地。  
（2）在答辩的时候，记得评委曾经提问过，为啥我没想到二分类，我的回答是陷入了思维局限了，确实，这也可以看出来，一个人的力量说白了还是很有限的。我在本次比赛做了一堆规则，其实如果将规则对应到一个二分类模型来说，这样二分类模型所学到的东西肯定是比人为定义规则间的优先级要好。一个规则，其实可以对应到二分类模型中的一个甚至是多个特征（比如书名号，可以提取成是否是书名号中的内容这一个特征），这样二分类模型自然会根据样本学习到规则间的相对重要度并体现到结果中。此外，人为做规则，能做的规则是有限的，然而如果是二分类模型，可以提取很多特征（提取候选词的tfidf、LDA等特征，也是一种变相的模型stacking融合了），特征如果是对标签是有区分度的，那么很有可能是可以给模型增加额外信息，提高模型的精度。  
（3）第一名的选手，采用的是二分类模型，直接将标题作为候选集，然后根据是否在标签集中打01标签，提取特征，用lightgbm做二分类，选择概率较大的两个词作为最终提交的label。比较有特色的特征，计算词word2vec与句子doc2vec向量的余弦距离  
部分特征：  
![image.png](https://upload-images.jianshu.io/upload_images/12207295-a6054a18812d1e6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
（4）第二名是通过tfidf选20个候选集，然后再打标签，特色特征：在整个数据集里被当成候选关键词的频率，这个其实就是该候选词在整个数据集中tfidf在前20的频率  
（5）第三名未引入外部词典，使用词的凝聚度和自由度从给定文档中发现新词。  
（6）第五名使用pyhanlp包来进行命名实体识别，识别人名，据说准确度比较高。  
