---
layout: post
title: "朴素贝叶斯文本分类"
date: 2014-06-27 13:11
comments: true
tags: 
- 机器学习
- 朴素贝叶斯
- Machine Learning
- 文本分类
---

回顾朴素贝叶斯(NB)分类器:  

{% math %}  
p(y_k|x)=\frac{p(y_k)p(x|y_k)}{p(x)}\propto p(y_k)p(x|y_k)=p(y_k)\prod_{i=1}^{d}{p(x_i|y_k)}  
{% endmath %}

  对于文本分类任务,即对一篇文章进行分类,是 NLP 中最常见的机器学习任务。一般情况下,类别从几个到几十不等,或者更多。使用朴素贝叶斯文分类器进行文本分类,我们需要首先考虑特征是什么,即x如何表示;$p(x_i|y_k )$的物理意义是什么,如何计算。对于特征方面,文本分类常规都是使用 bag-of-words 的特征,即以文章中出现的词作为特征,而不考虑词语出现的顺序。朴素贝叶斯文分类器也一般使用这种形式。那么特征空间的大小,就取决于词表(vocabulary)的大小,即语料集合中不重复词的个数。对于汉语来说,一般几万到百万不等。

  在 bag-of-words 的特征体系下,特征空间是确定了的,但是具体$x_i$的取值以及对应的$p(x_i |y_k )$的物理意义却可以有不同的考虑,对应着不同的参数计算公式及分类器训练和预测的实现。这取决于我们是否考虑词语在文章中出现的频次。

<!-- more -->

#### 伯努力(Bernoulli)NB

先看不考虑词频的情况。即只看某词语在某文章中是否出现,而不管出现了具体是多少次。这种假设下,每维特征的取值为 0-1,此时对应的 NB 分类器又被称为伯努力(Bernoulli)NB 分类器。
比如,如果词表是{2014、年、巴西、世界杯、足球赛、举行、是、第、20、届、球队},某文档是“2014 年巴西世界杯足球赛是第 20 届世界杯足球赛 ”,那么特征空间是 11,该文档特征向量是:

$$
x = (1,1,1,1,1,0,1,1,1,1,0)
$$

此时,$p(x_i|y_k)$的物理意义可认为是:若文章为 k 类别,则第 i 特征(词表第 i 个词)出现或者不出现的概率。那么:
$$
p(x_i = 0|y_k ) = 1 − p(x_i = 1|y_k)
$$

习惯的,我们经常用$p(x_i |y_k )$来作为$p(x_i = 1|y_k)$同等含义的一种表示。那此时,原NB模型的表达式可以写为:

{% math %}
p\left( { { y }_{ k } }|{ x } \right) \propto 
p\left( { y }_{ k } \right) \prod _{ i=1 }^{ d }{ \left\{ { \alpha  }_{ i }p\left( { { x }_{ i } }|{ { y }_{ k } } \right) 
+\left( 1-{ \alpha  }_{ i } \right) \left( 1-p\left( { { x }_{ i } }|{ { y }_{ k } } \right)  \right)  \right\}  } 
{% endmath %}

$\alpha_i$ 表示第 i 个词在该文档中出现了,没出现则为0.此时要非常注意,计算文章属于某个类别的得分的时候,不只要考虑该文章的 word,还要考虑在词表中的但是在该文章中没出现的 word!这类词对得分的贡献是$1 − p(x_i |y_k )$。因此伯努力 NB 下,分类的时间复杂度是 $O(Cd)$,C 是类别数,d 是词表大小。


那么伯努力 NB 下,p的参数估计表达式是多少呢?假设根据如上定义,及最大似然估计,可以得到:

{%  math %}
p\left( { { x }_{ i } }|{ { y }_{ k } } \right) 
=\frac { \sum _{ t=1 }^{ n }{ I\left\{ y_{ k }={ y }_{t } \right\} I\{ x_{ i }\quad in\quad y_{ t }\}  }  }{ \sum _{ t=1 }^{ t }{ I\left\{ y_{ k }={ y }_{ t } \right\}  }  }
=\frac { 特征词i在第k类文章中出现的文章数 }{ 第k类文章数 } 
{% endmath %}

其中函数I是指示函数，若x出现则值为1。可见,对于高频词,对应的这种条件概率是非常高的。 比如“的”(假设没去除停用词),其对应的条件概率值很可能会接近于 1.

再重复强调一下,此时的概率意义约束是: $p(x_i = 1|y_k) + p(x_i = 0|y_k ) = 1$

看一下伯努力 NB 下参数平滑的问题。使用加 1 平滑,即拉普拉斯平滑,此时在保证概率意义下,其平滑公式应该为:

$$
p(x_i |y_k ) = \frac{第 k 类文章中出现过第 i 词的文章数 + 1}{第k类文章数 + 2}
$$

提醒一下,此处分母加的值是 2,而不是词表大小。注意,平滑一定要使得平滑之后仍满足概率意义。


#### 多项式(Multinomial)NB

当我们考虑文章内词语的频次,而不只是考虑出现或未出现,此时特征的取值不再是 0-1,不过总的特征空间大小仍未变化。拿前面的例子来做对照,词表是{2014、年、巴西、世界杯、足球赛、举行、是、第、20、届、球队},某文档是“2014 年巴西世界杯足球赛是第 20 届世界杯足球赛”,此时该文章的特征向量为:

$$
x = (1,1,1,2,2,0,1,1,1,1,0)
$$

此时对应的 NB 一般称为多项式 NB。设 m 为文章内的总词频数,对应的模型表达式应该如下:

{% math %}
p(y_k|x)∝p(y_k)p(x|y_k)=p(y_k)
\frac{m!}{\prod_{i=1}^{d} x_i!}
\prod_{i=1}^{d}p(w_i|y_k)^{x_i}
∝p(y_k)\prod_{i=1}^{d}p(w_i|y_k)^{x_i}
{% endmath %}

之所以可以省掉这个多项式系数,是因为它是和类别$y_k$无关的。而此时,第 k 类别的所有文章中第 i 词的分布概率:


{% math %}
p\left( { { w }_{ i } }|{ { y }_{ k } } \right)
 =\frac { \sum _{ t=1 }^{ n }{ I\left\{ y_{ k }={ y }_{ t } \right\} x_{ i }^{ t } }  }
{ \sum _{ j=1 }^{ d }{ \sum _{ t=1 }^{ n }{ I\left\{ y_{ k }={ y }_{ t } \right\} x_{ i }^{ t } }  }  } 
=\frac { 特征词i在第k类文章中出现的总词频数 }{ 第k文章总词频数 } 
{% endmath %}

在多项式NB下,即使极高频词,其$p(w_i|y_k)$也很难接近于 1,另外其在模型中作用的时候是:$p(w_i|y_k)^{x_i}$.这时候的概率约束是:


{% math %}
\sum_{i=1}^{d}p(w_i|y_k)=1
{% endmath %}

因此对应的加 1 平滑为:

{%  math %}
p\left( { { w }_{ i } }|{ { y }_{ k } } \right) =\frac { \sum _{ t=1 }^{ n }{ I\left\{ y_{ k }={ y }_{ t } \right\} x_{ i }^{ t } } +1 }
{ \sum _{ j=1 }^{ d }{ \sum _{ t=1 }^{ n }{ I\left\{ y_{ k }={ y }_{ t } \right\} x_{ i }^{ t } }  }  +d} 
{% endmath %}

注意事项：

*  训练时候对于词语平铺的文本,应该要做词的聚合,即行程 bag-of-words 的形式比较有利于后续统计计算,特别是对于伯努利 NB 必须做去重。当然,对于多项式 NB,也可以顺次扫描累加。

*  预测时候的概率连乘,为了防止精度损失,可以改用取 log 相加。

*  对于短一些的文本,伯努利 NB 即可;对于长文本,考虑词频的多项式 NB 即可。当然也可以使用 tf-idf 等特征值,仿照多项式 NB 的形式。

*  预测时候,对于词表中出现但是本文章未出现的词语,伯努利 NB 下对得分有贡献,多项式 NB 下不用考虑;对于在词表中未出现的词,都可以不予以考虑,因为未登录词对各个类别的贡献是一样的。

#### 实验

关于实验数据，可以到[这里](https://github.com/aluenkinglee/mlclass/tree/master/NativeBayes)下载，训练集和测试集都已经很明白
总量在4000-的水平，result.dat是训练结果，可以看到测试集在训练数据的结果上准确率达到了100%。。这个是因为类别太少的缘故。只有3个类别，不过这个已经
可以看到朴素贝叶斯在工业界的应用可以达到较好的性能。

朴素贝叶斯分类器代码：

```java
/**
     * 训练目录下的样本集合
     * 
     * @throws IOException
     */
    public static void trainSamples() throws IOException {
        NaiveBayes classifier = new NaiveBayes();
        File flist = new File("./data-trainning-set");
        for (File f : flist.listFiles()) {
            classifier.training(new Instance(f));
        }
        classifier.save(new File("result.dat"));
        System.out.println("Trainning finished");
    }

    /**
     * specify the training dataset directory, and store result in the outfile
     * 
     * @param directory
     * @param result
     * @throws IOException
     */
    public static void trainSamples(String directory, String outfile)
            throws IOException {
        NaiveBayes classifier = new NaiveBayes();
        File flist = new File(directory);
        for (File f : flist.listFiles()) {
            classifier.training(new Instance(f));
        }
        classifier.save(new File(outfile));
        System.out.println("Trainning finished");
    }

    /**
     * 判断该实例所属的类别category
     * 
     * @param doc
     * @return
     */
    public String getCategory(Instance doc) {
        Collection<String> categories = VARIABLE.getCategories();
        System.out.println(categories);
        double best = Double.NEGATIVE_INFINITY;
        String bestName = null;
        for (String c : categories) {
            double current = getProbability(c, doc);
            System.out.println(c + ":" + current);
            if (best < current) {
                best = current;
                bestName = c;
            }
        }
        return bestName;
    }

    /**
     * 计算P（C)=该类型文档总数/文档总数，返回的数对数值
     * 
     * @param category
     * @return
     */
    public double getCategoryProbability(String category) {
        return Math.log(VARIABLE.getDocCount(category) * 1.0f
                / VARIABLE.getDocCount());
    }

    /**
     * 计算P(feature|cateogry),返回的是取对数后的数值
     * 
     * @param feature
     * @param category
     * @return
     */
    public double getFeatureProbability(String feature, String category) {
        int m = VARIABLE.getFeatureCount();
        return Math.log((VARIABLE.getDocCount(feature, category) + 1.0)
                / (VARIABLE.getDocCount(category) + m));
    }

    /**
     * 计算给定实例文档属于指定类别的概率，返回的是取对数后的数值
     * 
     * @param category
     * @param doc
     * @return
     */
    public double getProbability(String category, Instance doc) {
        double result = getCategoryProbability(category);
        for (String feature : doc.getWords()) {
            if (VARIABLE.containFeature(feature)) {
                result += getFeatureProbability(feature, category);
            }
        }
        return result;
    }

    /**
     * 加载训练结果
     * 
     * @param file
     * @throws IOException
     */
    public void load(File file) throws IOException {
        DataInputStream in = new DataInputStream(new FileInputStream(file));
        VARIABLE = Variable.read(in);
    }

    /**
     * 保存训练结果
     * 
     * @throws IOException
     */
    void save(File file) throws IOException {
        DataOutput out = new DataOutputStream(new FileOutputStream(file));
        VARIABLE.write(out);
    }

    /**
     * 训练一篇文档
     * 
     * @param doc
     */
    public void training(Instance doc) {
        VARIABLE.addInstance(doc);
    }
```

特征词类代码：

```java

public class Feature {
    /** 每个关键词在不同类别中出现的文档数量 */
    private Map<String, Integer> docCountMap = new HashMap<String, Integer>();
    /** 特征名称 */
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void incDocCount(String category) {
        if (docCountMap.containsKey(category)) {
            docCountMap.put(category, docCountMap.get(category) + 1);
        } else {
            docCountMap.put(category, 1);
        }
    }

    public int getDocCount(String category) {
        if (docCountMap.containsKey(category)) {
            return docCountMap.get(category);
        } else {
            return 0;
        }
    }

    public void write(DataOutput out) throws IOException {
        out.writeUTF(name == null ? "" : name);

        out.writeInt(docCountMap.size());
        for (String category : docCountMap.keySet()) {
            out.writeUTF(category);
            out.writeInt(docCountMap.get(category));
        }
    }

    public void readFields(DataInput in) throws IOException {
        this.name = in.readUTF();

        docCountMap = new HashMap<String, Integer>();
        int size = in.readInt();
        for (int i = 0; i < size; i++) {
            String category = in.readUTF();
            int docCount = in.readInt();
            docCountMap.put(category, docCount);
        }
    }

    public static Feature read(DataInput in) throws IOException {
        Feature f = new Feature();
        f.readFields(in);
        return f;
    }

    public static void main(String[] args) {
        // TODO Auto-generated method stub

    }

}
```

更多代码请看[这里](https://github.com/aluenkinglee/mlclass/tree/master/NativeBayes/src)


