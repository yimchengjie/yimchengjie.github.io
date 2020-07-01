---
title: 贝叶斯分类器的Java实现
categories:
  - 自然语言处理
tags:
  - 自然语言处理
  - NLP
toc: true
date: 2020-03-30 20:38:54
---
## 贝叶斯分类器的Java实现

--------------

```java
package com.ycj.publicOpinionAnalysisSystem.nlp.utility;

import com.hankcs.hanlp.classification.tokenizers.*;
import javax.xml.crypto.dom.DOMCryptoContext;
import java.io.Serializable;
import java.util.*;

/**
 * @Author: yanchengjie
 */
public class BayesClassifier implements Serializable {

    private static final long serialVersionUID = 8338342007460341994L;

    // 语料库总词数
    private Long corpusAllWordsNum = new Long(0);

    // 语料库总文档数
    private Long corpusAllDocNum = new Long(0);

    // 语料库各分类文档数
    private Map<String, Long> corpusClassDocNumMap = new HashMap<>();

    // 语料库各分类词数
    private Map<String, Long> corpusClassWordsNumMap = new HashMap<>();

    // 语料库各分类先验概率
    private Map<String, Double> corpusClassPriorProbabilityMap = new TreeMap<>();

    // 各分类词频树
    private Map<String, TreeMap<String, Integer>> corpusClassWordFrequency = new HashMap<>();

    // 语料库包含特征词的文档总数, 一篇文档包含多次计作一次
    private TreeMap<String, Integer> wordExistInAllDocNumTree;

    // 特征映射
    private Map<String, TreeMap<String, Double>> corpusClassWordFeature = new HashMap<>();

    // IDF映射
    private TreeMap<String, Double> corpusClassWordIDF = new TreeMap<>();


    // 分词器
    private ITokenizer tokenizer = new BigramTokenizer();

    // 中文停用词表
    private TreeSet<String> stopWordsTree;

    // 平滑系数
    private static Double ALPHA = 1.0;

    // 语料库各分类文档集合,String[]表示分词后一篇文档(语料加载元数据)
    private Map<String, Vector<String[]>> corpusClassDocSet;


    public BayesClassifier(Map<String, Vector<String[]>> corpusClassDocSet, TreeSet<String> stopWordsTree) {
        this.stopWordsTree = stopWordsTree;
        this.corpusClassDocSet = corpusClassDocSet;
        // 构建 语料库包含特征词的文档数Tree
        this.wordExistInAllDocNumTree = createWordExistInAllDocNumTree(corpusClassDocSet);
        if (corpusClassDocSet != null) {
            for (Map.Entry<String, Vector<String[]>> entry : corpusClassDocSet.entrySet()) {
                // 构建 各分类下文档数
                this.corpusClassDocNumMap.put(entry.getKey(), Integer.valueOf(entry.getValue().size()).longValue());
                // 构建 总文档数
                this.corpusAllDocNum += Integer.valueOf(entry.getValue().size()).longValue();
                // 构建 各分类的词频树
                TreeMap<String, Integer> wordFrequencyTree = createWordFrequencyTree(entry.getValue());
                this.corpusClassWordFrequency.put(entry.getKey(), wordFrequencyTree);
            }
            if (corpusClassWordFrequency != null) {
                // 构建 语料库各分类词数
                for (Map.Entry<String, TreeMap<String, Integer>> entry : corpusClassWordFrequency.entrySet()) {
                    String className = entry.getKey();
                    TreeMap<String, Integer> corpusTree = entry.getValue();
                    Long corpusNum = new Long(0);
                    Iterator<Map.Entry<String, Integer>> iterator = corpusTree.entrySet().iterator();
                    while (iterator.hasNext()) {
                        Map.Entry<String, Integer> wordNum = iterator.next();
                        // 去停用词
                        if (stopWordsTree.contains(wordNum.getKey())) {
                            iterator.remove();
                            continue;
                        }
                        corpusNum += wordNum.getValue().longValue();
                    }
                    corpusClassWordsNumMap.put(className, corpusNum);
                }
            }
            // 构建 语料库总词数
            for (Map.Entry<String, Long> entry : corpusClassWordsNumMap.entrySet()) {
                corpusAllWordsNum += entry.getValue();
            }
            // 构建 先验概率
            /*for (Map.Entry<String, Long> entry : corpusClassDocNumMap.entrySet()) {
                corpusClassPriorProbabilityMap.put(entry.getKey(), entry.getValue().doubleValue() / corpusAllDocNum);
            }*/
            // 构建 先验概率
            for (Map.Entry<String, Long> entry : corpusClassWordsNumMap.entrySet()) {
                corpusClassPriorProbabilityMap.put(entry.getKey(), Math.log(entry.getValue().doubleValue() / corpusAllWordsNum));
            }
            // 构建 特征映射 条件概率
            for (Map.Entry<String, TreeMap<String, Integer>> entry : corpusClassWordFrequency.entrySet()) {
                String className = entry.getKey();
                TreeMap<String, Integer> corpusTree = entry.getValue();
                TreeMap<String, Double> featureTree = new TreeMap<>();
                for (Map.Entry<String, Integer> wordNum : corpusTree.entrySet()) {
                    Double featureValue = (wordNum.getValue().doubleValue() + ALPHA) / (Integer.valueOf(corpusTree.size()).doubleValue()
                            + corpusClassWordFrequency.size() * ALPHA);
                    /*Double featureValue = (wordNum.getValue().doubleValue() + ALPHA) / (corpusClassWordsNumMap.get(className).doubleValue()
                                                                                        + corpusClassWordsNumMap.size() * ALPHA);*/
                    // Double featureValue = (wordNum.getValue().doubleValue() + ALPHA) / (corpusClassDocNumMap.get(className).doubleValue() + corpusClassDocNumMap.size() * ALPHA);
                    // 取对数
                    featureValue = Math.log(featureValue);
                    featureTree.put(wordNum.getKey(), featureValue);
                }
                corpusClassWordFeature.put(className, featureTree);
            }
            // 构建 IDF映射
            for (Map.Entry<String, Integer> entry : wordExistInAllDocNumTree.entrySet()) {
                Double idf = Math.log((ALPHA + corpusAllDocNum.doubleValue()) / (ALPHA + wordExistInAllDocNumTree.get(entry.getKey()).doubleValue()));
                idf = Math.log(idf);
                corpusClassWordIDF.put(entry.getKey(), idf);
            }
        }
    }

    /*public BayesClassifier(Map<String, TreeMap<String, Integer>> corpusClassWordFrequency, Map<String, Long> corpusEachClassDocNum, TreeSet<String> stopWordsTree) {
        this.corpusClassWordFrequency = corpusClassWordFrequency;
        this.corpusEachClassDocNum = corpusEachClassDocNum;
        this.stopWordsTree = stopWordsTree;
        if (corpusEachClassDocNum != null) {
            corpusEachClassDocNum.entrySet();
            for (Map.Entry<String, Long> entry : corpusEachClassDocNum.entrySet()) {
                corpusAllDocNum += entry.getValue();
            }
        }
        if (corpusClassWordFrequency != null) {
            // 计算语料库各分类总词数
            for (Map.Entry<String, TreeMap<String, Integer>> entry : corpusClassWordFrequency.entrySet()) {
                String className = entry.getKey();
                TreeMap<String, Integer> corpusTree = entry.getValue();
                Long corpusNum = new Long(0);

                Iterator<Map.Entry<String, Integer>> iterator=corpusTree.entrySet().iterator();
                while (iterator.hasNext()){
                    Map.Entry<String, Integer> wordNum= iterator.next();
                    // 去停用词
                    if (stopWordsTree.contains(wordNum.getKey())){
                        iterator.remove();
                        continue;
                    }
                    corpusNum += wordNum.getValue().longValue();
                    // 合并词数 构建全词频树
                    if (!corpusWordFrequency.containsKey(wordNum.getKey())){
                        corpusWordFrequency.put(wordNum.getKey(),wordNum.getValue());
                    }else {
                        corpusWordFrequency.replace(wordNum.getKey(),wordNum.getValue()+corpusWordFrequency.get(wordNum.getKey()));
                    }
                }
                corpusEachClassWordsNumMap.put(className, corpusNum);
            }
            // 计算语料库总词数
            for (Map.Entry<String, Long> entry : corpusEachClassWordsNumMap.entrySet()) {
                corpusAllWordsNum += entry.getValue();
            }
            // 计算先验概率
            for (Map.Entry<String, Long> entry : corpusEachClassWordsNumMap.entrySet()) {
                corpusClassPriorProbabilityMap.put(entry.getKey(), entry.getValue().doubleValue() / corpusAllWordsNum);
            }
            // 计算TF-IDF
            for (Map.Entry<String, TreeMap<String, Integer>> entry : corpusClassWordFrequency.entrySet()) {
                String className = entry.getKey();
                TreeMap<String, Integer> corpusTree = entry.getValue();
                TreeMap<String, Double> tfIDFTree = new TreeMap<>();
                for (Map.Entry<String, Integer> wordNum : corpusTree.entrySet()) {
                    Double alpha=1.0;
                    Double tf = (wordNum.getValue().doubleValue()+alpha) / (corpusEachClassWordsNumMap.get(className).doubleValue()+corpusEachClassDocNum.size()*alpha);
                    //Double idf = Math.log((1 + corpusAllDocNum) / corpusWordFrequency.get(wordNum.getKey()).doubleValue());
                    //Double tf_idf = tf * idf;
                    //Double tf_idf = tf;
                    tfIDFTree.put(wordNum.getKey(), tf);
                }
                corpusClassWordTFIDF.put(className, tfIDFTree);
            }
        }
    }*/

    private static TreeMap<String, Integer> createWordFrequencyTree(Vector<String[]> docSet) {
        if (docSet == null) throw new NullPointerException("docSet为空");
        TreeMap<String, Integer> wordFrequencyTree = new TreeMap<>();
        Iterator<String[]> iterator = docSet.iterator();
        while (iterator.hasNext()) {
            String[] strings = iterator.next();
            for (int i = 0; i < strings.length; i++) {
                if (wordFrequencyTree.containsKey(strings[i])) {
                    wordFrequencyTree.replace(strings[i], wordFrequencyTree.get(strings[i]) + 1);
                } else {
                    wordFrequencyTree.put(strings[i], 1);
                }
            }
        }
        return wordFrequencyTree;
    }

    private static TreeMap<String, Integer> createWordExistInAllDocNumTree(Map<String, Vector<String[]>> corpusClassDocSet) {
        if (corpusClassDocSet == null) throw new NullPointerException("corpusClassDocSet为空");
        TreeMap<String, Integer> wordExistInAllDocNumTree = new TreeMap<>();
        for (Map.Entry<String, Vector<String[]>> entry : corpusClassDocSet.entrySet()) {
            Vector<String[]> docSet = entry.getValue();
            Iterator<String[]> iterator = docSet.iterator();
            while (iterator.hasNext()) {
                String[] strings = iterator.next();
                List<String> stringDifferent = new ArrayList<>(strings.length);
                for (int i = 0; i < strings.length; i++) {
                    if (!stringDifferent.contains(strings[i])) stringDifferent.add(strings[i]);
                }
                for (int i = 0; i < stringDifferent.size(); i++) {
                    if (wordExistInAllDocNumTree.containsKey(stringDifferent.get(i))) {
                        wordExistInAllDocNumTree.replace(stringDifferent.get(i), wordExistInAllDocNumTree.get(strings[i]) + 1);
                    } else {
                        wordExistInAllDocNumTree.put(stringDifferent.get(i), 1);
                    }
                }
            }
        }
        return wordExistInAllDocNumTree;
    }


    public Double featureExtraction(String classification, String word) {
        Double featureValue = corpusClassWordFeature.get(classification).get(word);
        if (featureValue == null) {
            featureValue = ALPHA / (corpusClassWordsNumMap.get(classification).doubleValue() + corpusClassWordsNumMap.size() * ALPHA);
            featureValue = Math.log(featureValue);
            corpusClassWordFeature.get(classification).put(word, featureValue);
        }
        return featureValue;
    }

    public Double idfExtraction(String word) {
        Double idfValue = corpusClassWordIDF.get(word);
        if (idfValue == null) {
            idfValue = Math.log((ALPHA + corpusAllDocNum.doubleValue()) / ALPHA);
            idfValue = Math.log(idfValue);
            corpusClassWordIDF.put(word, idfValue);
        }
        return idfValue;
    }

    public Map<Double, String> calculateProbability(String[] strings) {
        Map<String, Integer> tempForTF = new HashMap<>();
        for (int i = 0; i < strings.length; i++) {
            if (tempForTF.containsKey(strings[i])) {
                tempForTF.replace(strings[i], tempForTF.get(strings[i]) + 1);
            } else {
                tempForTF.put(strings[i], 1);
            }
        }
        Map<Double, String> probabilityMap = new HashMap<>();
        for (Map.Entry<String, Double> entry : corpusClassPriorProbabilityMap.entrySet()) {
            Double probability = corpusClassPriorProbabilityMap.get(entry.getKey());
            Double feature;
            Double idf;
            for (int i = 0; i < strings.length; i++) {
                try {
                    if (stopWordsTree.contains(strings[i])) continue;
                    feature = featureExtraction(entry.getKey(), strings[i]);
                    idf = idfExtraction(strings[i]);
                    probability = probability
                            + feature + idf
                            + Math.log(tempForTF.get(strings[i]).doubleValue() / Integer.valueOf(strings.length).doubleValue());
                    //if (probability == 0.0 || probability < Double.MIN_VALUE) probability = Double.MIN_VALUE;
                } catch (NullPointerException e) {
                    e.printStackTrace();
                    continue;
                }
            }
            probabilityMap.put(probability, entry.getKey());
        }
        return probabilityMap;
    }

    public String classify(String text) {
        String[] strings = tokenizer.segment(text);
        Map<Double, String> probabilityMap = calculateProbability(strings);
        Double maxProbability = -Double.MAX_VALUE;
        for (Map.Entry<Double, String> entry : probabilityMap.entrySet()) {
            Double probability = entry.getKey() + Double.MIN_VALUE;
            System.out.println("分类:"+entry.getValue()+" 概率:"+probability);
            if (probability > maxProbability) {
                maxProbability = probability;
            }
        }
        //System.out.print("概率:" + (maxProbability - Double.MIN_VALUE) + ",");
        String result = probabilityMap.get(maxProbability - Double.MIN_VALUE);
        if (result == null) {
            result = "";
        }
        return result;
    }

    public String classify(String[] strings) {
        Map<Double, String> probabilityMap = calculateProbability(strings);
        Double maxProbability = -Double.MAX_VALUE;
        for (Map.Entry<Double, String> entry : probabilityMap.entrySet()) {
            Double probability = entry.getKey() + Double.MIN_VALUE;
            System.out.println("分类: 【"+entry.getValue()+"】 概率:"+probability);
            if (probability > maxProbability) {
                maxProbability = probability;
            }
        }
        //System.out.println("概率:" + (maxProbability - Double.MIN_VALUE) + ",");
        String result = probabilityMap.get(maxProbability - Double.MIN_VALUE);
        if (result == null) {
            result = "";
        }
        return result;
    }

}

```
