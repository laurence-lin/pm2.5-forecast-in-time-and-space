# pm2.5-forecast-in-time-and-space

数据：
1. 沈阳市11个雾霾监测站点的经纬度，及20130101-20151231期间监测站pm2.5值，时间粒度为天；
2. 污染源为沈阳14个发电厂、热电厂、制药厂、热力厂、炼焦厂、化工厂，数据包括污染源的经纬度，及20130101-20151231期间污染源几个烟囱周围的颗粒物浓度，时间粒度为天；
3. 沈阳市天气情况，包括是否采暖、是否降雨、平均气温（°C）、平均相对湿度（%）、湿度分类、风速（公里/小时）、风速（m/s）、风向、风向分类。

## 回归
以污染源颗粒物浓度/到站点距离（线性、平方或指数）及天气情况作为特征，建立回归模型。

由于特征间存在多重共线性的问题，采用岭回归，alpha从0到1粒度为0.01取，发现0.03最好，但是在测试集上的损失和普通回归模型差别不大，与真实值误差达到53；

lasso的效果和普通回归模型差别不大。

## GBR建模：
用前三天污染源的颗粒物浓度、天气情况、污染源与监测站的经纬度差值作为特征。

GBR相比于回归模型在特征工程方面更有优势。但GBR在模型特征的自定义上没有回归模型好。

迭代次数300，学习率0.1，每棵树的最大深度为9时，在测试集上的损失最小，与真实值误差为13；
