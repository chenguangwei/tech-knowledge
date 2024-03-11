# NLP学习



动手学深度学习（Pytorch）内容规划

## 面向人员

- 有Python基础

- 有高数，线代，概率论基础

- 本科大二左右，或者研一

## 学习资源

- 教材：https://zh-v2.d2l.ai/

- 纸质书： https://j.youzan.com/2E82KT（最低折扣五折购买地址）

- 视频： https://space.bilibili.com/1567748478/channel/seriesdetail?sid=358497

- 笔记：https://github.com/MLNLP-World/DeepLearning-MuLi-Notes/tree/main/notes

- 竞赛：https://tianchi.aliyun.com/competition/entrance/231784/introduction?spm=5176.12281973.0.0.7c47106baWMBl3

- OpenI云环境（可直接运行）：https://openi.pcl.ac.cn/Datawhale/d2l

## 课程大纲

### 整体排期

|                     |            |          |                          |
| ------------------- | ---------- | -------- | ------------------------ |
| **学习安排**            | **预计开始时间** | **学习时长** | **分享嘉宾**<br><br>**（待定）** |
| 开营：《动手学深度学习》开场分享    | 3月18日      | 10分钟     | **李沐**                   |
| 开营：如何学深度学习          | 3月18日      | 30分钟     | **杨毅远**                  |
| **Task01：**初识深度学习   | 3月19日      | 1天       | 助教团队                     |
| 直播：环境配置讲解           | 3月19日      | 1小时      | **丁一超、骆秀韬**              |
| **Task02：**预备知识     | 3月20日      | 2天       | 助教团队                     |
| **Task03：**线性神经网络   | 3月22日      | 2天       | 助教团队                     |
| 直播：线性神经网络串讲、习题解析与答疑 | 3月23日      | 1小时      | **罗如意、陈可为**              |
| **Task04：**多层感知机    | 3月24日      | 5天       | 助教团队                     |
| 直播：多层感知机串讲、习题解析与答疑  | 3月28日      | 1小时      | **刘洋、陈安东**               |
| **Task05：**竞赛实践     | 3月29日      | 2天       | 助教团队                     |
| 直播：竞赛解读与方案分享        | 3月30日      | 1小时      | **王茂霖、王凯**               |
| **结营：优秀学习者分享**      | 3月31日      | 1小时      | 马燕鹏                      |

### 课程内容



| 深度学习介绍            | [深度学习介绍_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1J54y187f9?p=1)                                                                       | 13      | [1. 引言 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_introduction/index.html)                                             |                                                                                                                                                                         |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 环境安装配置            | [03 安装【动手学深度学习v2】_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV18p4y1h7Dr/?vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                     | 16      | [安装 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_installation/index.html)                                                |                                                                                                                                                                         |
| 数据操作<br>          | [数据操作_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1CV411Y7i4?p=1&vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                              | 18      | [2.1. 数据操作 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_preliminaries/ndarray.html)                                      | [ndarray slides (d2l.ai)](https://courses.d2l.ai/zh-v2/assets/notebooks/chapter_preliminaries/ndarray.slides.html#/)                                                    |
| 数据预处理             | [数据预处理实现_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1CV411Y7i4?p=3&vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                           | 7       | [2.2. 数据预处理 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_preliminaries/pandas.html)                                      | [pandas slides (d2l.ai)](https://courses.d2l.ai/zh-v2/assets/notebooks/chapter_preliminaries/pandas.slides.html#/)                                                      |
| 线代<br>            | [05 线性代数【动手学深度学习v2】_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1eK4y1U7Qy/?vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                   | 30      | [2.3. 线性代数 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_preliminaries/linear-algebra.html)                               | [linear-algebra slides (d2l.ai)](https://courses.d2l.ai/zh-v2/assets/notebooks/chapter_preliminaries/linear-algebra.slides.html#/)                                      |
| 矩阵计算              | [06 矩阵计算【动手学深度学习v2】_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1eZ4y1w7PY/?vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                   | 12      | [2.4. 微积分 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_preliminaries/calculus.html)                                      |                                                                                                                                                                         |
| 自动求导              | [07 自动求导【动手学深度学习v2】_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1KA411N7Px/?vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                   | 18      | [2.5. 自动微分 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_preliminaries/autograd.html)                                     | [autograd slides (d2l.ai)](https://courses.d2l.ai/zh-v2/assets/notebooks/chapter_preliminaries/autograd.slides.html#/)                                                  |
| 线性回归+基础优化算法       | [08 线性回归 + 基础优化算法【动手学深度学习v2】_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1PX4y1g7KC/?vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)          | 22      | [3.1. 线性回归 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_linear-networks/linear-regression.html)                          | [linear-regression slides (d2l.ai)](https://courses.d2l.ai/zh-v2/assets/notebooks/chapter_linear-networks/linear-regression.slides.html#/)                              |
| 线性回归的从零开始实现       | [线性回归的从零开始实现_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1PX4y1g7KC?p=3&vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                       | 15      | [3.2. 线性回归的从零开始实现 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_linear-networks/linear-regression-scratch.html)           | [linear-regression-scratch slides (d2l.ai)](https://courses.d2l.ai/zh-v2/assets/notebooks/chapter_linear-networks/linear-regression-scratch.slides.html#/)              |
| 线性回归的简洁实现         | [线性回归的简洁实现_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1PX4y1g7KC?p=4&vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                         | 7       | [3.3. 线性回归的简洁实现 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_linear-networks/linear-regression-concise.html)             | [linear-regression-concise slides (d2l.ai)](https://courses.d2l.ai/zh-v2/assets/notebooks/chapter_linear-networks/linear-regression-concise.slides.html#/)              |
| Softmax 回归+损失函数   | [Softmax 回归_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1K64y1Q7wu?p=1&vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                        | 16      | [3.4. softmax回归 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_linear-networks/softmax-regression.html)                    |                                                                                                                                                                         |
| 图像分类数据集           | [图片分类数据集_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1K64y1Q7wu?p=3&vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                           | 9       | [3.5. 图像分类数据集 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_linear-networks/image-classification-dataset.html)            | [image-classification-dataset slides (d2l.ai)](https://courses.d2l.ai/zh-v2/assets/notebooks/chapter_linear-networks/image-classification-dataset.slides.html#/)        |
| Softmax 回归的从零开始实现 | [Softmax 回归从零开始实现_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1K64y1Q7wu?p=4&vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                  | 18      | [3.6. softmax回归的从零开始实现 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_linear-networks/softmax-regression-scratch.html)     | [softmax-regression-scratch slides (d2l.ai)](https://courses.d2l.ai/zh-v2/assets/notebooks/chapter_linear-networks/softmax-regression-scratch.slides.html#/)            |
| Softmax 回归的简洁实现   | [Softmax 回归简洁实现_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1K64y1Q7wu?p=5&vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                    | 4       | [3.6. softmax回归的从零开始实现 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_linear-networks/softmax-regression-scratch.html)     | [softmax-regression-concise slides (d2l.ai)](https://courses.d2l.ai/zh-v2/assets/notebooks/chapter_linear-networks/softmax-regression-concise.slides.html#/)            |
| 感知机               | [感知机_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1hh411U7gn?p=1&vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                               | 14      | [4.1. 多层感知机 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/mlp.html)                                |                                                                                                                                                                         |
| 多层感知机             | [多层感知机_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1hh411U7gn?p=2&vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                             | 22      | [4.1. 多层感知机 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/mlp.html)                                | [mlp slides (d2l.ai)](https://courses.d2l.ai/zh-v2/assets/notebooks/chapter_multilayer-perceptrons/mlp.slides.html#/)                                                   |
| 多层感知机的从零开始实现      | [代码实现_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1hh411U7gn?p=3&vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                              | 8       | [4.2. 多层感知机的从零开始实现 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/mlp-scratch.html)                 | [mlp-scratch slides (d2l.ai)](https://courses.d2l.ai/zh-v2/assets/notebooks/chapter_multilayer-perceptrons/mlp-scratch.slides.html#/)                                   |
| 多层感知机的简洁实现        | [代码实现_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1hh411U7gn?p=3&vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                              |         | [4.2. 多层感知机的从零开始实现 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/mlp-scratch.html)                 | [mlp-concise slides (d2l.ai)](https://courses.d2l.ai/zh-v2/assets/notebooks/chapter_multilayer-perceptrons/mlp-concise.slides.html#/)                                   |
| 模型选择              | [模型选择_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1kX4y1g7jp?p=1&vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                              | 18      | [4.4. 模型选择、欠拟合和过拟合 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/underfit-overfit.html)            | [underfit-overfit slides (d2l.ai)](https://courses.d2l.ai/zh-v2/assets/notebooks/chapter_multilayer-perceptrons/underfit-overfit.slides.html#/)                         |
| 过拟合和欠拟合           | [过拟合和欠拟合_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1kX4y1g7jp?p=2&vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                           | 16      | [4.4. 模型选择、欠拟合和过拟合 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/underfit-overfit.html)            | [underfit-overfit slides (d2l.ai)](https://courses.d2l.ai/zh-v2/assets/notebooks/chapter_multilayer-perceptrons/underfit-overfit.slides.html#/)                         |
| 权重衰退              | [12 权重衰退【动手学深度学习v2】_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1UK4y1o7dy/?vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                   | 25      | [4.5. 权重衰减 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/weight-decay.html)                        | [weight-decay slides (d2l.ai)](https://courses.d2l.ai/zh-v2/assets/notebooks/chapter_multilayer-perceptrons/weight-decay.slides.html#/)                                 |
| Dropout           | [13 丢弃法【动手学深度学习v2】_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Y5411c7aY/?vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                    | 25      | [4.6. 暂退法（Dropout） — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/dropout.html)                     | [dropout slides (d2l.ai)](https://courses.d2l.ai/zh-v2/assets/notebooks/chapter_multilayer-perceptrons/dropout.slides.html#/)                                           |
| 数值稳定性             | [数值稳定性_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1u64y1i75a?p=1&vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                             | 15      | [4.8. 数值稳定性和模型初始化 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/numerical-stability-and-init.html) | [numerical-stability-and-init slides (d2l.ai)](https://courses.d2l.ai/zh-v2/assets/notebooks/chapter_multilayer-perceptrons/numerical-stability-and-init.slides.html#/) |
| 模型初始化和激活函数        | [模型初始化和激活函数_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1u64y1i75a?p=2&vd_source=4cc9f0023dddad3ba6f7edc17eb3040d)                        | 24      | [4.8. 数值稳定性和模型初始化 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh-v2.d2l.ai/chapter_multilayer-perceptrons/numerical-stability-and-init.html) | [numerical-stability-and-init slides (d2l.ai)](https://courses.d2l.ai/zh-v2/assets/notebooks/chapter_multilayer-perceptrons/numerical-stability-and-init.slides.html#/) |
| 二手车交易价格预测竞赛实战     | [赛事地址：https://tianchi.aliyun.com/competition/entrance/231784/information](https://tianchi.aliyun.com/competition/entrance/231784/information) |         |                                                                                                                                                  | [Datawhale 零基础入门数据挖掘-Baseline_天池notebook-阿里云天池 (aliyun.com)](https://tianchi.aliyun.com/notebook/95422)                                                                 |
| 二手车交易价格预测竞赛实战     | [赛事地址：https://tianchi.aliyun.com/competition/entrance/231784/information](https://tianchi.aliyun.com/competition/entrance/231784/information) |         |                                                                                                                                                  | [Datawhale 零基础入门数据挖掘-Baseline_天池notebook-阿里云天池 (aliyun.com)](https://tianchi.aliyun.com/notebook/95422)                                                                 |
| 结营：优秀学习者分享        | 直播嘉宾：马燕鹏、优秀学习者                                                                                                                                | 预计60min |                                                                                                                                                  |                                                                                                                                                                         |

## 直播

- 内容
  
  - 预热：AI：从小白到入门，超详细人工智能成长路径分享
  
  - 活动开营：邀请李沐老师开场分享，杨毅远分享如何学深度学习
  
  - 直播1：环境配置讲解
  
  - 直播2：线性神经网络串讲、习题解析与答疑
  
  - 直播3：多层感知机串讲、习题解析与答疑
  
  - 直播4：竞赛解读与方案分享
  
  - 活动结营：邀请优秀学习者分享

- 直播形式
  
  - 小鹅通主平台 + 视频号/B站同步推流
