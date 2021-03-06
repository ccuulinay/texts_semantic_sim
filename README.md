---
# 文本语义相似度建模

---
问题：对于给到的2句描述，设计一套算法识别他们是否语义上是一致的。  

数据描述：
- 每一行数据包括
    - 问题1
    - 问题2
    - 是否语义一致，0表示不一致，1表示一致。所以是一个二分类问题。
- 训练集包括（0：50220，1：11266），测试集包括（0：25169，1：5575）。两类数据比例约为1：5，分布不均衡。  

数据增强：
- 在数据探索，数据清洗，分词的过程中，需要对中文文字数据进行处理
    - 删除停用词。
    - 替换个别常见错别字词。
    - 补充部分常见的特征未登录词作为补充词典。
    
---
## 思路：  
机器学习分类算法或者神经网络分类。
- 机器学习分类算法，作为建模，需要对输入语句处理，并进行特征抽取和特称工程（选择和表示）。
- 神经网络则需要通过表示学习方法将输入的文字表示为词向量（或字向量），并选择网络结构。

---
## 评价标准选择：
- 准确率，accuracy
- F1-score
- ROC_AUC_SCORE

---
## Day1
### 机器学习分类模型建模
1. 数据预处理，删除停用词，jieba分词，纠错。
2. 利用gensim建立word2vec模型。
3. 构建特征
    - q1和q2的分别的长度
    - 长度差
    - q1和q2的分别有效分词个数
    - 共有词数
    - 基于fuzzywuzzy的匹配
    - 基于fuzzywuzzy的非完全匹配
    - 基于fuzzywuzzy的忽略顺序匹配
    - 基于fuzzywuzzy的去重子集匹配
    - Word Mover Distance
    - Normalized Word Mover Distance
    - 基于gensim的词向量计算的cosine/曼哈顿/jaccard等各种距离
4. 去除null及inf值的行
5. 选用支持向量机SVC和XGBoost构建两个分类模型。

#### 在测试集效果：
##### 未处理数据不均衡：
###### SVC:  
F1 score: 0.06675862068965517  
Accuracy: 0.8165799175883757  
混淆矩阵：  
            
| cm |  Pred 0| Pred 1|
|----|  ----  | ----  |
|Actual 0| 14940  | 162 |
|Actual 1| 3221   | 121 |

###### XGBoost:  
F1 score: 0.23973880597014927  
Accuracy: 0.8232487529819996  
混淆矩阵：  
            
| cm |  Pred 0| Pred 1|
|----|  ----  | ----  |
|Actual 0| 14670  | 432 |
|Actual 1| 2828   | 514 |

##### 用重抽样将label1的训练样例增加到与label0是1：1：
###### SVC:  
F1 score： 0.21446563512054695  
Accuracy： 0.763283452613316  
混淆矩阵：  
            
| cm |  Pred 0| Pred 1|
|----|  ----  | ----  |
|Actual 0| 14670  | 432 |
|Actual 1| 2828   | 514 |

###### XGBoost:  
F1 score： 0.3841292134831461  
Accuracy： 0.8098026458468879  
ROC_AUC_SCORE: 0.6219581174762067  
混淆矩阵：  
            
| cm |  Pred 0| Pred 1|
|----|  ----  | ----  |
|Actual 0| 13842  | 1260 |
|Actual 1| 2248   | 1094 |

---

## Day2&3
### 神经网络分类模型建模
1. 采用tencent ai lab的预训练词向量。
2. 数据预处理。
3. 建立神经网络。
    - 基于双向lstm编码的孪生网络，采用对数负曼哈顿距离作为衡量两个输入的相似度衡量；
    - 基于双向gru编码的，并计算两个输入的对数负曼哈顿距离，cosine距离，L1距离，乘积，合并后作为MLP的输入；  


###### manhattan_lstm_distance:  
F1 score: 0.1351220300630354  
Accuracy: 0.8259497788186313  
ROC AUC SCORE: 0.5336348419215253  
混淆矩阵：  
            
| cm |  Pred 0| Pred 1|
|----|  ----  | ----  |
|Actual 0| 24975  | 194 |
|Actual 1| 5157   | 418 |

###### manhattan_lstm_distance (upsampled):  
F1 score: 0.4581174031847734  
Accuracy: 0.8129391100702577  
ROC AUC SCORE: 0.6662370055554496  
混淆矩阵：  
            
| cm |  Pred 0| Pred 1|
|----|  ----  | ----  |
|Actual 0| 22562  | 2607 |
|Actual 1| 3144   | 2431 |

###### nn with concate L1 distance, cosine distance, exp neg manhattan distance, and multiply and then MLP. (upsampled & 2 epoch):  
F1 score: 0.3597253080185821  
Accuracy: 0.5875618006765547  
ROC AUC: SCORE 0.6075543781436592  
混淆矩阵：  
            
| cm |  Pred 0| Pred 1|
|----|  ----  | ----  |
|Actual 0| 22562  | 2607 |
|Actual 1| 3144   | 2431 |



---

## TODO
### 神经网络练习
- 测试语义相似度的深度学习模型的预训练的词向量（fasttext/腾讯AI LAB等）,作为编码输入文本的编码输入。
- 利用上述作为输入，建立并测试LSTM/BiLSTM/TextCNN/孪生网络（Siamese Network）等，并验证效果。
- 测试不同用来衡量编码输入后的相似的度量。比如差值，点积，或其他组合。
### 机器学习练习
- 调优模型。
- 对于解决样本不平衡，还可以尝试将label为1的数据的问题1/2位置置换，来构造新的数据，达到数据增强。
- 同时应该再深入探索数据的分布。例如，删除停用词后的分词，如果顺序和组合（即排列）完全相同，可认为是同一个问句，也可以利用在增强数据。