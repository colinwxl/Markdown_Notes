# NMF方法及实例
[Original Site](https://blog.csdn.net/JinbaoSite/article/details/73928729)
## 简介
1. 负矩阵分解（Non-negative Matrix Factorization ，NMF）是在矩阵中所有元素均为非负数约束条件之下的矩阵分解方法。 
2. 基本思想：给定一个非负矩阵V，NMF能够找到一个非负矩阵W和一个非负矩阵H，使得矩阵W和H的乘积近似等于矩阵V中的值。 
$$V_{n\times m} = W_{n\times k}H_{k\times m}$$

<img src="https://t1.picb.cc/uploads/2018/08/28/JMD2A0.jpg" style="width:400px;height:350px;"></img>
W矩阵：基础图像矩阵，相当于从原矩阵V中抽取出来的特征 
H矩阵：系数矩阵。

3. NMF能够广泛应用于图像分析、文本挖掘和语音处理等领域。 
4. 矩阵分解优化目标：最小化W矩阵H矩阵的乘积和原始矩阵之间的差别，目标函数如下： 
$$argmin\frac{1}{2}\parallel X-WH\parallel^2=\frac{1}{2}\sum\limits_{i,j}(x_{ij}-WH_{ij})^2$$

5. 基于KL散度的优化目标，损失函数如下： 
$$argminj(W,H)=\sum\limits_{i,j}(X_{ij}ln\frac{X_{ij}}{WH_{ij}}-X_{ij}+WH_{ij})$$

## 实例一：图像处理
已知Olivetti人脸数据共400个，每个数据是64*64大小。由于NMF分解得到的W矩阵相当于从原始矩阵中提取的特征，那么就可以使用NMF对400个人脸数据进行特征提取。 
通过设置k的大小，设置提取的特征的数目。在本实验中设置k=6，随后将提取的特征以图像的形式展示出来。

```python
import matplotlib.pyplot as plt
from sklearn import decomposition
from sklearn.datasets import fetch_olivetti_faces
from numpy.random import RandomState

n_row, n_col = 2, 3
n_components = n_row*n_col
image_shape = (64, 64)
dataset = fetch_olivetti_faces(shuffle=True, random_state=RandomState(0))
faces = dataset.data

def plot_gallery(title, images, n_col=n_col, n_row=n_row):
    plt.figure(figsize=(2. * n_col, 2.26 * n_row))
    plt.suptitle(title, size=16)
    for i, comp in enumerate(images):
        plt.subplot(n_row, n_col, i+1)
        vmax = max(comp.max(), -comp.min())
        plt.imshow(comp.reshape(image_shape), cmap=plt.cm.gray,interpolation='nearest', vmin=-vmax, vmax=vmax)
        plt.xticks(())
        plt.yticks(())
    plt.subplots_adjust(0.01, 0.05, 0.99, 0.94, 0.04, 0.)

estimators = [('Eigenfaces - PCA using randomized SVD', decomposition.PCA(n_components=6, whiten=True)), ('Non-negative components - NMF', decomposition.NMF(n_components=6, init='nndsvda', tol=5e-3))]

for name, estimator in estimators:
    print("Extracting the top %d %s..." % (n_components, name))
    print(faces.shape)
    estimator.fit(faces)
    components_ = estimator.components_
    plot_gallery(name, components_[:n_components])

plt.show()
```

## 实例二：NMF-based推荐算法
[Original Site](https://github.com/chenzomi12/NMF-example)
在例如Netflix或MovieLens这样的推荐系统中，有用户和电影两个集合。给出每个用户对部分电影的打分，希望预测该用户对其他没看过电影的打分值，这样可以根据打分值为其做出推荐。用户和电影的关系，可以用一个矩阵来表示，每一列表示用户，每一行表示电影，每个元素的值表示用户对已经看过的电影的打分。

```python
#coding:utf-8
import matplotlib.pyplot as plt
plt.rcParams['font.sans-serif']=['SimHei']

from sklearn.decomposition import NMF
import numpy as np

items = ['希特勒回来了', '死侍', '房间', '龙虾', '大空头', '极盗者', '裁缝', '八恶人', '实习生', '间谍之桥']
users = ['五柳君', '帕格尼六', '木村静香', 'WTF', 'airyyouth', '橙子c', '秋月白', 'clavin_kong', 'olit', 'You_某人', '凛冬将至', 'Rusty', '噢！你看！', 'Aron', 'ErDong Chen']
RATE_MATRIX = np.array(
    [[5, 5, 3, 0, 5, 5, 4, 3, 2, 1, 4, 1, 3, 4, 5],
     [5, 0, 4, 0, 4, 4, 3, 2, 1, 2, 4, 4, 3, 4, 0],
     [0, 3, 0, 5, 4, 5, 0, 4, 4, 5, 3, 0, 0, 0, 0],
     [5, 4, 3, 3, 5, 5, 0, 1, 1, 3, 4, 5, 0, 2, 4],
     [5, 4, 3, 3, 5, 5, 3, 3, 3, 4, 5, 0, 5, 2, 4],
     [5, 4, 2, 2, 0, 5, 3, 3, 3, 4, 4, 4, 5, 2, 5],
     [5, 4, 3, 3, 2, 0, 0, 0, 0, 0, 0, 0, 2, 1, 0],
     [5, 4, 3, 3, 2, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1],
     [5, 4, 3, 3, 1, 0, 0, 0, 0, 0, 0, 0, 0, 2, 2],
     [5, 4, 3, 3, 1, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1]]
)

# nmf_model为NMF的类，users_dis为W矩阵，items_dis为H矩阵
nmf_model = NMF(n_components=2) # 设有2个主题
items_dis = nmf_model.fit_transform(RATE_MATRIX)
users_dis = nmf_model.components_

print('用户的主题分布：')
print(users_dis)
print('电影的主题分布：')
print(items_dis)

# 把电影主题分布矩阵和用户分布矩阵画出来
import matplotlib.pyplot as plt
plt1 = plt
plt1.plot(items_dis[:, 0], items_dis[:, 1], 'ro')
plt1.draw()#直接画出矩阵，只打了点，下面对图plt1进行一些设置

plt1.xlim((-1, 3))
plt1.ylim((-1, 3))
plt1.title(u'the distribution of items (NMF)')#设置图的标题

count = 1
zipitem = zip(items, item_dis)#把电影标题和电影的坐标联系在一起

for item in zipitem:
    item_name = item[0]
    data = item[1]
    plt1.text(data[0], data[1], item_name, horizontalalignment='center', verticalalignment='top', wrap=True)
plt1.show()

users_dis = users_dis.T #把转置用户分布矩阵
plt2 = plt
plt2.plot(users_dis[:, 0], users_dis[:, 1], 'ro')
plt2.xlim((-1, 3))
plt2.ylim((-1, 3))
plt2.title(u'the distribution of user (NMF)')#设置图的标题

zipuser = zip(users, users_dis)#把电影标题和电影的坐标联系在一起
for user in zipuser:
    user_name = user[0]
    data = user[1]
    plt2.text(data[0], data[1], user_name, horizontalalignment='center', verticalalignment='top', wrap=True)
plt2.show()

# 推荐 对于NMF的推荐很简单 1.求出用户没有评分的电影，因为在numpy的矩阵里面保留小数位8位，判断是否为零使用1e-8（后续可以方便调节参数），当然你没有那么严谨的话可以用 = 0。 2.求过滤评分的新矩阵，使用NMF分解的用户特征矩阵和电影特征矩阵点乘。 3.求出要求得用户没有评分的电影列表并根据大小排列，就是最后要推荐给用户的电影id了。

rec_mat = np.dot(items_dis, users_dis.T)
filter_matrix = RATE_MATRIX < 1e-8
print('重建矩阵，并过滤掉已经评分的物品：')
rec_filter_mat = (filter_matrix * rec_mat).T
print(rec_filter_mat)

rec_user = '凛冬将至'  # 需要进行推荐的用户
rec_userid = users.index(rec_user)  # 推荐用户ID
rec_list = rec_filter_mat[rec_userid, :]  # 推荐用户的电影列表
print('推荐给凛冬将至用户的电影：')
print([items[i] for i in np.nonzero(rec_list)[0]])

# 误差
a = NMF(n_components=2)  # 设有2个主题
W = a.fit_transform(RATE_MATRIX)
H = a.components_
print(a.reconstruction_err_)

b = NMF(n_components=3)  # 设有3个主题
W = b.fit_transform(RATE_MATRIX)
H = b.components_
print(b.reconstruction_err_)

c = NMF(n_components=4)  # 设有4个主题
W = c.fit_transform(RATE_MATRIX)
H = c.components_
print(c.reconstruction_err_)

d = NMF(n_components=5)  # 设有5个主题
W = d.fit_transform(RATE_MATRIX)
H = d.components_
print(d.reconstruction_err_)
```