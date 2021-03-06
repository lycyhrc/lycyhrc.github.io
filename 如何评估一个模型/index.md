# 如何评估一个模型




## 什么是模型评估

- 需要一种在模型之间进行选择的方法：不同的模型类型，调整参数和特征
- 使用**模型评估**(**model evaluation**)来估计模型将泛化到样本外数据的程度
- 需要**模型评估指标**(**model evaluation  metric** )来量化模型性能

## 如何进行模型评估

> 最简单的模型评估方法就是留出数据集的一部分评估，且这一部分数据必须是模型以前没见过的。

具体到不同场景有不同方法：

### 一个简单的模型

将数据集分割为一个单独的训练和测试集（留出法）。

在前者上训练模型，在后者上评估模型(这里的评估指的是计算性能指标，如Precision、Accuracy、Recall、ROC AUC等)。

![](/01/20201217_3_split.png)

sklearn的[train_test_split](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.train_test_split.html)有具体的实现。

```python
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.33, random_state=42)
```

### 训练模型并调整（优化）其超参数

将数据集拆分为单独的测试和训练集。 在训练集上使用诸如k折交叉验证之类的技术来找到模型的**“最佳”超参数集**。 

如果完成了超参数调整，使用独立的测试集来获得对其性能的无偏估计。 

下面我使用一个图来说明不同之处：

![](/01/20201217_4_split-and-evaluation.png)

主要步骤为：

1. 训练数据集划分为训练子集和验证集
2. 在训练自己上进行模型训练
3. 评估验证集以优化其超参数
4. 在独立的测试集上进行测试

### 建立不同的模型并比较不同的算法

这里先介绍一下K折验证法：

在k倍交叉验证中，将数据集划分为k个相等大小的样本。 在这k个子样本中，保留了一个子样本作为用于测试模型的验证数据，其余的k-1个子样本用作训练数据。 然后，此交叉验证过程将重复k次，k个子样本中的每一个仅被用作一次验证数据。 然后可以将k个结果取平均值以产生单个估计。

<img src="https://static.packt-cdn.com/products/9781789617740/graphics/b04c27c5-7e3f-428a-9aa6-bb3ebcd3584c.png" alt="img" style="zoom:67%;" />

在建立不同的模型并比较不同的算法的场景中，我们要使用嵌套的交叉验证，嵌套交叉验证中，有一个外部k-fold交叉验证循环将数据分为训练fold和测试fold，内部的k-fold交叉验证循环通过训练层上的k-fold交叉验证选择模型。 选择模型后，然后使用test fold评估模型性能。 在确定了“最喜欢的”算法之后，我们可以采用“常规” k倍交叉验证方法（在完整的训练集上）找到其“最佳”超参数，并在独立的测试集上对其进行评估。

让我们考虑一下逻辑回归模型，以使其更清楚：使用嵌套的交叉验证训练m个不同的逻辑回归模型，m个外部折叠中的每个折叠模型都训练1个，内部折叠用于优化每个模型的超参数（例如， 结合使用网格搜索和k折交叉验证，如果您的模型稳定，则这m个模型应该都具有相同的超参数值，并根据外部测试折数报告该模型的平均性能。 

## 模型评估指标

模评估指标因机器学习任务类型的不同而有所变化，其主要有两种：

1. **回归问题：** 平均绝对误差，平均平方误差，均方根误差
2. **分类问题：** 分类准确度（其中包括了许多其他评估指标）

### 分类问题

> 从二分类角度思考，常见的一些术语包括

- True positives(TP): 本身是正例，预测**也**为正例的样本数量
- False positives(FP):本身是反例，预测为正例的样本数量
- True negatives(TN):本身是反例，预测**也**为反例的样本数量
- False negatives(FN)：本身是正例，预测为反例的样本数量

#### Confusion Matrix

是上述参数的矩阵形式的表示。可以使用可视化和代码呈现:

![Confusion Matrix](https://miro.medium.com/max/759/1*yYctsCAlkQHixEHYy0dHPw.png)

```python
from sklearn.metrics import confusion_matrix
y_actu = [1, 0, 1, 1, 0, 1, 1, 1, 1, 0, 1, 1]
y_pred = [0, 0, 1, 1, 0, 1, 1, 0, 1, 0, 1, 1]
confusion_matrix(y_actu, y_pred)
## array([[3, 0],
##       [2, 7]])
```

输出一个Numpy矩阵

- **TP=3：有3个0(正例),预测出3个0（正例）**
- FP=2：有7个1(反例)，预测出2个0（正例）
- **TN=7：有7个1(反例)，预测出7个1（正例）**
- FN=0：有3个0(正例),预测出0个1（正例）

```python
import pandas as pd
y_actu = pd.Series([1, 0, 1, 1, 0, 1, 1, 1, 1, 0, 1, 1], name='Actual')
y_pred = pd.Series([0, 0, 1, 1, 0, 1, 1, 0, 1, 0, 1, 1], name='Predicted')
df_confusion = pd.crosstab(y_actu, y_pred)
df_confusion
```

| Predicted |    0 |    1 |
| --------: | ---: | ---: |
|    Actual |      |      |
|         0 |    3 |    0 |
|         1 |    2 |    7 |

将得到一个（带有标签的）Pandas DataFrame，进一步可得到所有统计和

```python
df_confusion = pd.crosstab(y_actu, y_pred, rownames=['Actual'], colnames=['Predicted'], margins=True)
df_confusion
```

| Predicted |    0 |    1 |  All |
| --------: | ---: | ---: | ---: |
|    Actual |      |      |      |
|         0 |    3 |    0 |    3 |
|         1 |    2 |    7 |    9 |
|       All |    5 |    7 |   12 |

我们可以可视化这一张图表试试

```python
import matplotlib.pyplot as plt
import numpy as np
def plot_confusion_matrix(df_confusion, title='Confusion matrix', cmap=plt.cm.gray_r):
    plt.matshow(df_confusion, cmap=cmap) # imshow
    #plt.title(title)
    plt.colorbar()
    tick_marks = np.arange(len(df_confusion.columns))
    plt.xticks(tick_marks, df_confusion.columns, rotation=45)
    plt.yticks(tick_marks, df_confusion.index)
    #plt.tight_layout()
    plt.ylabel(df_confusion.index.name)
    plt.xlabel(df_confusion.columns.name)

plot_confusion_matrix(df_confusion)
```

![](/01/20201217_1_confusion-matrix.png)

#### Accuracy

评判模型最常用的指标，实际上并不是性能的明确指标。当数据集不平衡时，此情况不适用。

以癌症检测模型为例。患癌症的几率非常低。假设在100名患者中，有90人没有患癌症，剩下的10人实际上患有癌症。我们不想错过一个患有癌症却未被发现的病人(false negative)。检测每个人是否患有癌症的准确率为90%。这个模型在这里什么也没做，只是对所有100个预测都给出了癌症的概率。此情况需要更好的替代方案。

#### Precision

正实例占**预测的正实例**总数的百分比分母(TP + FP)是对整个给定数据集做的正例的模型预测。
$$
\dfrac{TP }{TP+FP }
$$

#### Recall

正实例占**实际正实例**总数的百分比。分母(TP + FN)是数据集中出现的积极实例的实际数量。
$$
\dfrac{TP }{TP+FN }
$$

#### Specificity

反实例在实际总反实例中所占的百分比。分母（TN + FP）是数据集中存在的否定实例的实际数量。 它类似于召回，但变化是负面的。一种衡量类之间是否分离的度量。
$$
\dfrac{TN }{TN+FP }
$$

#### F1-score

是精度和召回率的调和平均数。因此，如果正面预测确实是正面的(precision)，并且没有错过正面预测并预测它们是负面的(recall)，那么模型在F1得分上表现得很好。
$$
\frac{2}{\frac{1}{precision} + \frac{1}{recall}}=\frac{2*precision*recall}{precision+recall}
$$
该方法的一个缺点在于precision和recall都被同等重视，但是在实际应用中，我们可能有指标偏好性，因此F1分数可能不是精确的指标。因此，加权F1得分（ weighted-F1 score ）或查看PR或ROC曲线都可以提供帮助。



我们可以使用以下代码生成confusionmatrix，再利用公式计算以上数据以上数据

```python
df_conf_norm = df_confusion / df_confusion.sum(axis=1)
df_conf_norm
```

| Predicted |        0 |        1 |   All |
| --------: | -------: | -------: | ----: |
|    Actual |          |          |       |
|         0 | 0.500000 | 0.000000 | 0.125 |
|         1 | 0.333333 | 0.388889 | 0.375 |
|       All | 0.833333 | 0.388889 | 0.500 |

使用classification_report可以得到上述值。

```python
from sklearn.metrics import classification_report, confusion_matrix
y_actu = [1, 0, 1, 1, 0, 1, 1, 1, 1, 0, 1, 1]
y_pred = [0, 0, 1, 1, 0, 1, 1, 0, 1, 0, 1, 1]
print(classification_report(y_actu, y_pred))
```

![](/01/20201217_2_confusion-matrix-report.png)

#### PR曲线

它是不同阈值在precision和recall之间的曲线。下图中，有6个预测变量，分别显示了它们在各种阈值下的精确调用曲线。 图的右上方（黑色曲线）是我们获得高精度和召回率的理想空间。 AUC就是曲线下的面积。 其数值越高越好。

![Image for post](https://miro.medium.com/max/741/1*FPT_ykB5W226a-GW3g8l5A.png)

#### ROC 曲线

ROC针对各种阈值绘制了针对 True Positive Rate(TPR)和False Positive Rate(FPR)的图表。随着TPR的增加，FPR也跟着增加。如图左所示，我们想要四个类别更靠近左上角的阈值。如图右所示，在给定的数据集上比较不同的预测变量也变得容易了，您可以根据手头的应用选择阈值。
$$
TPR = Recall = \dfrac{TP}{TP+FN}
$$

$$
FPR = 1 - Specificity = \dfrac {FP}{FP+TN}
$$

![Image for post](https://miro.medium.com/max/1400/1*3eRQjKTr18QOuozyl3daMQ.png)

## 模型评估案例

### 场景1:评估一个简单模型

### 场景2: 训练模型并调整（优化）其超参数

## 参考

1. https://towardsdatascience.com/various-ways-to-evaluate-a-machine-learning-models-performance-230449055f15
2. https://www.ritchieng.com/machine-learning-evaluate-classification-model/
