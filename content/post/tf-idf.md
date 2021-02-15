---
title: "tf-idf 算法"
date: "2019-07-03 08:54:36"
tags: ["算法"]
categories: ["搜索"]
mathjax: true
description: "tf-idf 算法的相关内容"
---

tf-idf (term frequence-inverse document frequence) 词频-逆文档频率，是搜索常用的一个权重相关算法，其作用是评估一个 document 在一整个 document list 中的重要程度，下面分开来讲。

##  term frequence

tf 的意思就是一个词语在一个文件中出现的次数，对于一篇文章， tf 算法会将这篇文章进行分词并统计出现次数。这个词在这篇文章中出现的次数越多则说明越不重要，权重则越低。

其公式如下：

$$词频(tf_t,d)=\frac{单词在文档中出现的次数}{文档的总次数}$$

## inverse document frequence

先说文档频率(df)，表示文档集中包含某个词的所有文档数目。但是通常这个数目会很大，对此可以使用逆文档将其映射成小的取值范围，逆文档频率公式如下：

$$逆文档频率(idf_t)=log(\frac{文档集总数}{包含某个词的文档数})=log(\frac{N}{df_t+1})$$

# tf-idf

公式如下:

$$词频-逆文档频率(tf-idf)= 词频(tf) \times 逆文档频率(idf)$$

# 编码

```java
public class TfIdf {

    /**
     * 词频
     * 此数值越大则代表这个 term 在当前文档中越重要
     * @param doc
     * @param term
     * @return
     */
    private double tf(List<String> doc, String term) {
        double termFrequency = 0;
        for (String str : doc) {
            if (str.equalsIgnoreCase(term)) {
                termFrequency++;
            }
        }
        return termFrequency / doc.size();
    }


    /**
     * 文档频率
     * 此值会越高越说明不重要
     * @param docs
     * @param term
     * @return 存在 term 的文档数目
     */
    private int df(List<List<String>> docs, String term) {
        int n = 0;
        if (term != null && !"".equals(term)) {
            for (List<String> doc : docs) {
                for (String word : doc) {
                    if (term.equalsIgnoreCase(word)) {
                        n++;
                        break;
                    }
                }
            }
        } else {
            System.out.println("term 不能为 null 或者空字符串");
        }
        return n;
    }


    /**
     * 逆文档频率
     * 此值越小则说明当前 term 越不重要
     * @param docs
     * @param term
     * @return
     */
    private double idf(List<List<String>> docs, String term) {
        return Math.log(docs.size() / df(docs, term) + 1);
    }


    private double tfIdf(List<String> doc, List<List<String>> docs, String term) {
        return tf(doc, term) * idf(docs, term);
    }


    public static void main(String[] args) {
        List<String> doc1 = Arrays.asList("北京", "上海", "杭州");
        List<String> doc2 = Arrays.asList("北京", "深圳", "南京");
        List<String> doc3 = Arrays.asList("南京", "北京", "深圳");
        List<String> doc4 = Arrays.asList("上海", "广州", "云南");

        List<List<String>> documents = Arrays.asList(doc1, doc2, doc3, doc4);

        TfIdf tfIdf = new TfIdf();
        System.out.println("【北京】在 doc1 中的词频：" + tfIdf.tf(doc1, "北京"));
        System.out.println("【北京】在 doc4 中的词频：" + tfIdf.tf(doc4, "北京"));
        System.out.println("【北京】在 文档集 中的词频：" + tfIdf.df(documents, "北京"));
        System.out.println("【北京】的 if-idf 算法：" + tfIdf.tfIdf(doc2, documents, "北京"));
        System.out.println("【深圳】的 if-idf 算法：" + tfIdf.tfIdf(doc2, documents, "深圳"));
        /**
         * 【北京】在 doc1 中的词频：0.3333333333333333
         * 【北京】在 doc4 中的词频：0.0
         * 【北京】在 文档集 中的词频：3
         * 【北京】的 if-idf 算法：0.23104906018664842
         * 【深圳】的 if-idf 算法：0.3662040962227032
         */
    }
}
```



# 参考

《从 Lucene 到 Elasticsearch 全文检索实战》