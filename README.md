# pm2.5-forecast-in-time-and-space

数据：
1. 沈阳市11个雾霾监测站点的经纬度，及20130101-20151231期间监测站pm2.5值，时间粒度为天；
2. 污染源为沈阳14个发电厂、热电厂、制药厂、热力厂、炼焦厂、化工厂，数据包括污染源的经纬度，及20130101-20151231期间污染源几个烟囱周围的颗粒物浓度，时间粒度为天；
3. 沈阳市天气情况，包括是否采暖、是否降雨、平均气温（°C）、平均相对湿度（%）、湿度分类、风速（公里/小时）、风速（m/s）、风向、风向分类。

特征处理：
1. 部分特征二值化编码(OneHotEncoder()(输入必须是2-D array，无法直接对字符串型的类别变量编码), LabelEncoder()和LabelBinarizer()(输入被限定为 1-D array))
    对于字符串型类别变量，方法一：先用LabelEncoder() 转换成连续的数值型变量，再用OneHotEncoder()二值化；方法二：直接用LabelBinarizer()进行二值化。
2. 异常值和缺失值处理，因数据中这两种情况较少，所以直接删除样本。
3. 归一化（回归）
4. 离散化（回归）
5. 正则化（回归）

特征筛选：
1. Filter—考虑自变量和目标变量之间的关联。  
&emsp;对于连续型的变量之间的相关性，可以采用相关系数来评估，比如皮尔逊相关系数。  
&emsp;对于类别型的可以采用假设检验的方式，比如卡方检验。  
&emsp;（对于连续型的自变量和二元的离散因变量，利用WOE，IV，通过WOE的变化来调整出最佳的分箱阀值，通过IV值，筛选出有较高预测价值的自变量。）  
&emsp;R平方，一个变量的变化有百分之多少可以有另外一个变量来解释。还有互信息、信息增益等等。
2. Wrapper方式，主要考虑的是离线和在线评估是否增加一个特征，通过选定模型评估的指标(AUC、MAE、MSE)来评价对特征增加和去除后模型的好坏，通常有前向和后向两种特征选择方式。
3. Embedded方式，通过分类学习器本身对特征自动的进行刷选，比如逻辑回归中的L1 L2惩罚系数，决策树中的基于最大熵的信息增益选择特征。

## 回归
&emsp;以污染源颗粒物浓度/到站点距离（线性、平方或指数）及天气情况作为特征，建立回归模型。  
&emsp;由于特征间存在多重共线性的问题，采用岭回归，alpha从0到1粒度为0.01取，发现0.03最好，但是在测试集上的损失和普通回归模型差别不大，与真实值误差达到33.6；  
&emsp;lasso的效果和普通回归模型差别不大。

## GBR建模
&emsp;用前三天污染源的颗粒物浓度、天气情况、污染源与监测站的经纬度差值作为特征。  
&emsp;GBR相比于回归模型在特征工程方面更有优势。但GBR在模型特征的自定义上没有回归模型好。  
&emsp;用GridSearchCV寻找最优参数，当n_estimators=300, learning_rate=0.1, max_depth=9, subsample=0.8, max_features=0.6时，在测试集上的损失最小，与真实值误差为8.3；

&emsp;缺点：因为每次迭代都依赖之前树的信息，所以不能并行化，训练速度慢

## 多层感知器 MLP
只有一个输出层，误差为45

有两层或三层（隐藏层+输出层），效果提升不明显，误差为43，比普通线性回归还差

不管怎么调参或者加dropout层防止过拟合，损失都收敛到相同的值。

## GBR + 回归
&emsp;根据GBR跑出的叶子信息将训练集和测试集重新编码，得到的特征放进回归，岭回归，lasso预测。
&emsp;误差：普通回归 28， 岭回归 9.5， lasso 15。（因为GBR重新编码的样本存在多重共线性，普通回归效果较差，岭回归最好。岭回归与lasso的效果差别是因为空间内球面和菱面与目标函数相切的位置不同。）
