title: 基于RGB-D图像的hand pose最新文章回顾
date: 2016-01-01 14:43:31
tags:
- Hand Pose
- Paper Reading
categories:
- Research
---
当前在实验室的主要topic是RGB-D图的hand pose。这个topic在2015年开始爆发，从去年寥寥无几的几篇，到今年各种会议上超过十篇，performance提升得也很快。所以在2016年的第一天按照时间顺序整理一下自2014年起最新的hand pose文章。为了简单起见~~且配置MathJax失败~~，本文尽量用通俗的语言来描述工作，不出现数学公式。
<!--more-->
注：本文的图片均为相应文章的截图。
又注：每个标题为对应文章或project的链接。

### 1. [Latent Regression Forest: Structured Estimation of 3D Articulated Hand Posture (CVPR 2014, oral)](http://www.iis.ee.ic.ac.uk/~dtang/hand.html)
本文是ICL的工作，主要贡献在于利用非监督的手拓扑结构学习及隐变量随机森林提高了hand pose估计的精度，并发布了一个新的包含180K图像的数据集（不过标注是由算法得到，并不是很准确）。该算法基于拓扑结构，coarse-to-fine地去层级寻找隐节点对应拓扑结构的重心位置，最终找到手的各个节点位置，如下图所示：
![算法示意图](/img/hand-paper/hand-paper-cvpr1-alg.png)

该算法首先需要非监督地学习如何将手的结构进行划分，即Latent Tree Model(LTM)。基于定义好的距离度量（欧几里德距离或者测地线距离），Chow-Liu Neighbour-Jointing算法将学出一个层级的拓扑结构树，类似于一个聚类的过程，把相近的节点连在一起。欧几里德距离和测地线距离的聚类效果图如下：
![LTM结构](/img/hand-paper/hand-paper-cvpr1-ltm.png)

然后根据学到的拓扑结构，算法训练一个多级随机森林（RF）来学习层级地寻找每个隐节点的重心位置，即Latent Regression Forest(LRF)。该forest和传统RF的区别在于多了一种叫division node的节点，它根据LTM将当前拓扑结构划分为两部分子结构。另外两种节点split node和leaf node和普通RF一样。LRF的示意图如下：
![LRT示意图](/img/hand-paper/hand-paper-cvpr1-lrf.png)

训练时LRF按阶段来进行学习，每个阶段对应LTM的一个节点。从根节点开始，该阶段相当于学习一个回归出该节点重心位置偏移量的的RF，即不断对节点进行分裂，直到信息增益低于阈值。然后对该节点根据拓扑结构进行划分。之后分别对划分过的两个节点进行同样的操作（学习子节点重心到父节点的偏置），直至划分到LTM的叶节点。测试时对所有叶节点的回归结果进行投票即得到hand pose。LRF相比与传统的RF，每张图的每个节点只需要一个sample（传统RF需要在每个地方进行vote），因此计算效率更高。

由于coarse阶段所需训练数据比fine阶段要少，作者采取了逐步增加训练数据的策略，即将训练数据划分为D份（D为树的深度），逐阶段增加一份训练样本。为了防止错误累积，在每个阶段结束时，在进行划分前，使用下一阶段的训练数据作为validation set，将树继续分裂直至在validation set上的偏置variance最小，之后再进行划分。

作者在ICL数据集上比较了基于欧几里德距离、测地线距离的LTM和随机得到的LTM的回归误差，以此证明测地线距离效果最好（见下图）。从图中也可看出，从指根到指尖的误差依次递增，指尖的误差均大于20mm，这是因为层级回归的误差累计。
![LTM结果](/img/hand-paper/hand-paper-cvpr1-ltm-result.png)

在和之前方法的比较中，作者的方法也是优势明显，在这就不贴实验结果了，具体见原文。

### 2. [Realtime and Robust Hand Tracking from Depth (CVPR 2014, oral)](http://research.microsoft.com/en-us/um/people/yichenw/handtracking/index.html)
这篇是MSRA的工作，主要的贡献是提出了一个实时鲁棒的手部跟踪算法。他们基于之前的“初始化+局部优化”框架进行改进，将手部模型表示为球的组合，定义了新的代价函数，然后结合ICP和PSO算法进行优化，以及提出了基于简单手指检测的手部初始化算法。

对于手的模型，作者用简单的48个球形来表示（见下图），这样能够提高计算速度。基于该模型，作者定义了一个代价函数，由三项构成，分别是点云到模型的L1距离项、对位于点云外模型点的惩罚项和对球形相交的惩罚性。为了减少计算时间，代价函数的计算是通过对点云进行降采样来进行的。
![手模型](/img/hand-paper/hand-paper-cvpr2-hand-model.png)

跟踪的过程即是从上一帧或者手指检测得到的初始手势出发，对代价函数进行局部优化。由于计算代价函数需要降采样，这导致优化函数不光滑且存在很多局部最小点，传统的PSO算法并不能解决这个问题。作者提出了一种ICP和PSO相结合的策略，即每次更新PSO前先将模型利用ICP朝着目标点云迭代几次，这样做显著地提高了收敛速度，如下图所示。算法细节在此处略去，ICP算法见[这篇BMVC2008的文章](http://www.vision.ee.ethz.ch/en/publications/get_abstract.cgi?procs=559&mode=&lang=dj)，PSO算法见[这篇BMVC2011的文章](http://users.ics.forth.gr/~argyros/research/kinecthandtracking.htm)。
![ICP-PSO结果](/img/hand-paper/hand-paper-cvpr2-icp-pso.png)

为了获得更好的初始化结果，作者提出了在XY平面和Z方向分别进行手指检测的算法（见下图）：
- **XY-手指：**在XY平面上，从点云中心开始，迭代寻找距离变换最远的点作为指尖候选点。然后以这些指尖候选点往回生长直到长度或宽度超过手指大小。对于每段候选手指，通过计算长度、宽度和长宽比来对其是否为手指进行分类。
- **Z-手指：**在Z方向上寻找深度的极小点作为指尖候选点，然后从这些指尖候选点向后生长直到深度超过手指指骨深度。最后以指尖点为圆心的作两个同心圆，候选手指在两个区域的面积比大于给定阈值则判断为手指。
![手指检测](/img/hand-paper/hand-paper-cvpr2-finger.png)

在获取手指之后，首先假设检测到的手指为直的（2个DOF），其他手指为弯的（DOF忽略不计），然后对于每个检测到的手指及去除手指的手掌利用PCA找到其大致方向作为约束，最后利用Inverse Kinematics算法计算出大致的hand pose。（由于手指对应关系未知，需要对每一种可能均进行计算，然后取误差的最小值）

在自己的数据集上，作者对比了之前BMVC2011的算法和自己算法的性能，证明该算法误差约为10mm，且速度在CPU上为25fps：
![结果](/img/hand-paper/hand-paper-cvpr2-result.png)

### 3. [Real-Time Continuous Pose Recovery of Human Hands Using Convolutional Networks (TOG 2014)](http://cims.nyu.edu/~tompson/NYU_Hand_Pose_Dataset.htm)
本工作出自NYU的LeCun组，是第一篇采用CNN来进行hand pose estimation的文章。作者的方法是先用RF将手分割出来，然后用CNN得到各个关键点的heat map，最后用Inverse Kinematics得到3D的hand pose，效果如下图所示。同时，本文release了一个数量很大的数据集NYU，是目前用的比较多的hand pose数据集之一。
![效果](/img/hand-paper/hand-paper-tog1-result-track.png)

对于手部分割，作者训练了一个二分类的RF，用于区分手和背景。该RF使用的特征是MSR常用的pixel difference特征，训练数据则是采集的图像，其中被试的手涂成红色方便后期分割。数据及RF的结果如下图：
![分割结果](/img/hand-paper/hand-paper-tog1-rf.png)

为了训练CNN，作者用多个深度摄像头采集了不同姿态的手部图像，并用之前的PSO算法离线得到ground truth。这样做的好处是离线算法可以配合很复杂的手部模型因此得到更准确的结果，而且使用多个摄像头可以解决因自遮挡而引起的pose不确定的情况。为了使hand pose更容易学，作者并不是直接回归pose，而是回归出每个对应点的heat map图像。该图像代表的是图中各点为关键点的概率，可以从ground truth由2D的高斯分布得到，如下图：
![heat map](/img/hand-paper/hand-paper-tog1-heat-map.png)

作者使用了multi-scale的CNN，将原图根据平均深度尺缩并归一化后经过不同程度的降采样，三路输入到CNN，得到heat map，CNN的结果如下面三张图所示：
![CNN结构1](/img/hand-paper/hand-paper-tog1-cnn1.png)
![CNN结构2](/img/hand-paper/hand-paper-tog1-cnn2.png)
![CNN结构3](/img/hand-paper/hand-paper-tog1-cnn3.png)

在得到heat map之后，作者通过LM算法在heat map上拟合2D的高斯模型得到每个关键点的位置，然后通过Inverse Kinematics算法从二维的位置得到3D的信息。在NYU数据集上2D的估计误差如下表，其中图像大小为640x480。可以看出，指尖误差最大。另外，算法运行速度在GPU上达到30fps。
![结果](/img/hand-paper/hand-paper-tog1-result.png)

### 4. [Accurate, Robust, and Flexible Real-time Hand Tracking (CHI 2015)](http://research.microsoft.com/apps/pubs/default.aspx?id=238453)
MSR的工作，发在HCI的顶会CHI上。本文的主要贡献是提出了一个实用的手部跟踪系统，该系统具有如下特点：
- **Accurate：**对于不同的手势均有准确的重建结果。
- **Robust：**系统能从各种瞬时跟踪错误中恢复。
- **Flexible：**这是该系统与其他系统最不同的地方。该系统只需要一个深度摄像头，对于手的位置（最远在几米范围也能work）、方位等均没有特殊要求。效果见下图：
![算法效果](/img/hand-paper/hand-paper-chi1-result.png)

该系统的流程并不复杂，主要思想是产生一系列候选手的图像，然后和实际图像进行比较，筛选、演化出最终的pose。算法如下图所示，主要包括手部ROI提取、重新初始化（Reinitialization）和模型拟合三部分。
![算法流程](/img/hand-paper/hand-paper-chi1-alg.png)

**1. 手部ROI提取：**首先利用传统方法来得到初始的框，比如detector或者上一帧的手的位置。然后利用RF分类器对该区域附近的像素进行手臂/手的二分类，该分类器在生成的数据上进行训练。最后搜索邻域，找到包含手概率和最大的框。

**2. 重新初始化：**Reinitialization是算法最关键的一步，文中采用了两级RF的方法，第一级预测全局旋转所在的bin，第二级预测具体的pose，这样的设计使得第二级中pose的variance大大减少。和之前基于RF的hand pose预测方法不同，二级的RF只是用于产生pose的分布，在下一个阶段将利用该分布来产生一系列不同的hypothesis。该RF也在合成的数据上训练，好像现在基于合成数据的训练已经是主流的做法了。

**3. 模型拟合：**模型拟合采用的是PSO算法与基因算法相结合的方法。首先定义golden能量函数为对应位置深度的L1截断距离的和，然后生成多个粒子（pose和粒子状态），使用基因算法的策略，对一些能量比较高的粒子进行随机变换。最终经过很多代迭代之后再选出最低能量的粒子作为最终的pose。由于相比于经典的PSO算法多了随机更迭的过程，所以效果更好。

作者定性对比了他们系统与Leap Motion及3Gears的效果，证明他们的系统更能处理一些复杂的姿态。在Dexter1数据集上，本文的算法比ICCV2013年的[算法](http://handtracker.mpi-inf.mpg.de/projects/handtracker_iccv2013/)好很多：
![实验结果](/img/hand-paper/hand-paper-chi1-dexter1.png)

### 5. [Hands Deep in Deep Learning for Hand Pose Estimation (CVWW 2015)](https://cvarlab.icg.tugraz.at/projects/hand_detection/)
水会的文章，用multi-scale和multi-stage的CNN强刷hand pose的回归，即先用第一阶段的网络回归出粗略的hand pose位置，再在第二阶段在这些位置附近取patch回归pose的偏置。没什么内容，直接看下图的网络结构就好：
![第一阶段结构](/img/hand-paper/hand-paper-cvww1-cnn1.png)
![第二阶段结构](/img/hand-paper/hand-paper-cvww1-cnn2.png)

作者在NYU和ICL上都做了实验，把之前文章的performance给刷下去了，见下图。（吐槽一句，作者为了憋出创新点提出了Pose Prior的东西，还argue说加了这个比不加要好，但实际上只是因为相比于没加prior的结构增加了一层，导致fitting能力变强了。）
![结果](/img/hand-paper/hand-paper-cvww1-result1.png)

作者也给出了平均关节误差的图，可以看出，指尖误差最大，在NYU上超过了20mm，ICL上超过10mm。此外，由于网络本身规模不大，运行速度能在CPU上达到500fps。
![具体误差](/img/hand-paper/hand-paper-cvww1-result2.png)

### 6. [Combining Discriminative and Model Based Approaches for Hand Pose Estimation (FG 2015)](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=7163141)
这篇工作首次在hand pose上将discriminative的方法与模型拟合的方法结合在一起。首先利用RF得到part的分类结果，然后利用类似ICP的迭代方法，每次对于每个点云的点，在模型对应part上找最近的点，找到对应关系之后梯度下降优化变换，然后再重新寻找最近的点，如此迭代直至收敛。算法流程如下图所示。由于该算法的performance只比CVPR2014的LRF好一点点，所以这里就不贴结果了，详见原文。
![算法流程](/img/hand-paper/hand-paper-fg1-algo.png)

### 7. [Cascaded Hand Pose Regression (CVPR 2015)](http://research.microsoft.com/en-us/people/yichenw/)
MSRA的这篇工作主要拓展了经典的2D的cascade pose regression框架（见Dollar的CVPR2010[文章](http://vision.ucsd.edu/~pdollar/research.html#PoseEstimation)）。文章主要有两个创新点，一是优化了pixel difference（或pose indexed features）特征，二是针对手的特殊结构提出了层级cascade regression的方法。文章还发布了自己的声称更challenging的数据集。

算法首先需要将手进行归一化，以增加invariance特性。之前的工作都是用3D平移变换，这里的创新点在于用了3D的刚性变换（多了三维的旋转），对于手掌和手指分别采用不同的局部坐标系，坐标轴可以根据hand pose关键点的位置来确定：
![归一化](/img/hand-paper/hand-paper-cvpr3-normalize.png)

然后对归一化后的关键点提取3D pose indexed特征。为了使特征具有一定的3D不变性，和之前只用深度对特征偏置进行归一化不同，作者首先产生随机的3D偏置，然后利用相机参数将其变换到图像坐标来提取特征。

最后算法进行层级的cascade RF回归。和之前的cascade算法的唯一不同点在于，层级算法首先估计手掌的位置，之后将其固定，并分别估计各个手指的位置。这是因为手掌的位置相对于手指更加稳定，因此更容易估计，而且手掌位置的变化将会显著影响手指的估计，同时手指之间的估计是基本相互独立的。层级与全局cascade算法的区别见下图。算法的初始pose根据点云的PCA方向及mean pose来确定。
![层级算法](/img/hand-paper/hand-paper-cvpr3-cr.png)

作者给出了ICL数据集上的结果，比CVPR2014那篇LRF在指尖上好了一些，但指尖误差仍然大于10mm。同时，可以看出，手掌的收敛速度明显比手指要快，采用层级的方法加速了他们的收敛。
![算法结果ICL](/img/hand-paper/hand-paper-cvpr3-result1.png)

在作者自己的数据集上，明显效果比ICL的数据集要差，这说明他们数据集更难了。同时，他们也证明了层级以及3D特征的效果。实验结果见下图。此外，该算法由于使用了RF，运行速度在CPU上大于300fps。
![算法结果MSRA](/img/hand-paper/hand-paper-cvpr3-result2.png)

### 8. [Fast and Robust Hand Tracking Using Detection-Guided Optimization (CVPR 2015)](http://handtracker.mpi-inf.mpg.de/projects/FastHandTracker/)
MPII这个组一直在做基于tracking的hand pose算法，发过一篇ICCV两篇CHI。这篇文章也利用了part classification的结果。他们的主要贡献是提出了一种新的detection驱动的tracking框架。作者借鉴了之前基于GMM的配准算法的思想，把手模型和深度图均表示成GMM，这样优化能量更加光滑。为了结合detection的结果，作者把part classification的结果作为能量项的约束也进行优化。此外，作者使用了late fusion的方法，实际上就是对不同约束下得到的结果进行筛选得到最好的结果，这样做比PSO要更快（在CPU下50fps）。算法流程图如下：
![算法流程](/img/hand-paper/hand-paper-cvpr4-alg.png)

对于深度图，首先用四叉树（quadtree）将其分解成深度相似的区域，然后每个区域用一个2D高斯模型来表示，均值是中心位置，标准差则和区域大小成正比，每个区域的深度用所有像素的深度平均值来表示。对于手的模型，每个关节用一个3D高斯模型来表示。为了能够计算模型与深度图的相似性，作者在计算时将手的3D高斯模型利用投影变换投到图像平面上，然后再计算2D高斯模型的相似性。

在优化时，作者定义了一个深度姿态（depth-only）的目标函数，由四项构成：
- **深度相似性项：**计算深度图与加权的高斯模型的类相关系数（即逐项相乘的积分和），其中权值在深度差小于阈值时为线性递减，当超过阈值时则为0。
- **冲突惩罚项：**计算模型的类自相关系数（即各项相乘的积分和，不包括重复的项）。
- **关节限制惩罚项：**对超过预设角度范围的参数进行惩罚。
- **平滑惩罚项：**计算时域上参数的差来进行惩罚。

基于detection的优化（detection-guided）仅在深度相似性项略有不同。首先利用RF对手进行part分类，然后给每个高斯模型安排一个label，对于相似性项的权值，当label不同时也设为0。为了结合这两种目标函数，作者首先利用相邻两帧的hand pose随机插值得到几个候选初始化pose，然后一部分用depth-only优化，一部分用detection-guided优化，最后选取一个效果最好的作为hand pose。此外，作者还给出了一种自适应用户手型的策略，即搜索固定范围的高斯模型参数，然后取能量最小时的参数作为该手型的模型。

作者在Dexter 1数据集上进行了实验，取得了19.6mm的平均误差，见下图。。
![结果](/img/hand-paper/hand-paper-cvpr4-result.png)

### 9. [First-Person Pose Recognition using Egocentric Workspaces (CVPR 2015)](http://www.cs.cmu.edu/~deva/papers/ego_workspaces.pdf)
这篇工作针对第一人称（egocentric）的3D上肢pose（手臂+手）识别，提出了一个简单有效的4行代码实现的算法。首先在训练样本中提取perspective-aware深度特征，然后将pose进行聚类把问题转化为多分类问题，最后用线性SVM对特征进行分类。

为了获取训练数据，作者利用商用软件产生具有真实3D场景的包含手的图像，利用这些数据进行训练。作者将3D空间进行spherical binning（中心在相机原点），从而使深度图量化来作为特征，这样做的好处是同一个bin永远对应图像中相同面积的patch。binning的过程见下图：
![bin过程](/img/hand-paper/hand-paper-cvpr5-bin.png)

在binning之后，对于每一幅图像建立二值化的特征b[u, v, w]，即若原图中z[u, v] = w，则有b[u, v, w] = 1，对于被遮挡的像素也设为1。之后再对b进行降采样以减少计算量。计算feature的过程见下图：
![特征](/img/hand-paper/hand-paper-cvpr5-feature.png)

为了将原问题转换成多分类问题，作者将训练数据按照pose进行聚类，将每一类平均的pose来代表该类。然后训练线性的one-vs-all SVM对特征进行分类。提取特征和SVM分类可以合为一步，因为计算多分类的分数只是单纯一个累加权值的过程，代码见下：
![代码](/img/hand-paper/hand-paper-cvpr5-code.png)

和之前基于HOG特征的方法以及基于正交voxel的方法相比，作者的简单方法准确率提升了20个点（见下图）。关于其他细节请见原文。
![结果](/img/hand-paper/hand-paper-cvpr5-result.png)

### 10. [Hybrid One-Shot 3D Hand Pose Estimation by Exploiting Uncertainties (BMVC 2015, oral)](http://lrs.icg.tugraz.at/research/hybridhape/)
该工作提出了一种混合data-driven和model-based方法的框架，首先利用RF得到关键点的分布，然后再对这些分布进行模型拟合得到最终的hand pose。

对于data-driven部分，文章采用了MSR之前human pose的[做法](http://research.microsoft.com/en-us/projects/vrkinect/)，即训练的时候在RF存储一系列joint的mode，然后测试的时候用这些mode来进行voting，详见[这篇文章](http://research.microsoft.com/apps/pubs/default.aspx?id=154457)。基于这些mode，作者定义了一个模型拟合的能量函数，简单来说就是mode权值越高、距离越近则能量越高。然后作者用PSO来求解模型参数得到hand pose。

作者在ICL上与LRF比较，在NYU与CNN比较，performance均显著好于之前的方法，见下图。
![ICL结果](/img/hand-paper/hand-paper-bmvc1-result1.png)
![NYU结果](/img/hand-paper/hand-paper-bmvc1-result2.png)

### 11. [Rule Of Thumb: Deep Derotation for Improved Fingertip Detection (BMVC 2015)](http://www.cs.technion.ac.il/~twerd/HandNet/)
这篇文章提出了一种指尖检测的预处理方法，并release了一个指尖数据集。作者argue说之前的CNN的方法都需要augment数据，但他们的方法不需要（实际上augment数据是很cheap的）。

作者首先使用一个CNN来预测手的转向，然后利用该旋转矩阵把图像转正之后再使用传统的RF或者CNN来得到指尖的位置。作者的评估指标比较奇怪，用的是detection里的mP和mAP，阈值取了15mm（估计是用普通的指标比不过其他方法）。结果见下表。比较奇怪的是，RF的方法居然比CNN的要好，感觉是训练得不好。
![结果](/img/hand-paper/hand-paper-bmvc2-result.png)

### 12. [Training a Feedback Loop for Hand Pose Estimation (ICCV 2015, oral)](https://cvarlab.icg.tugraz.at/pubs/oberweger_iccv15.pdf)
这篇工作是前面那篇CVWW的CNN工作的升级版。作者使用的是data-driven的方法，但借鉴了model-based方法的思想，即通过CNN来学习模型产生和获取模型间差异，从而得到更准确的结果。简单来说，作者训练了三个CNN，一个用来给出粗略的初始hand pose，一个用来从hand pose得到深度图，最后一个从生成的深度图和实际的深度图得到pose的偏置。算法在生成深度图和更新pose之间迭代，主要流程如下图。之所以设计这个迭代的过程，而不是利用最小化合成图和原图差别来获取pose，是因为该目标函数的局部极值过多，容易收敛到非实际的pose上，而且合成的图无法模拟实际图的噪声，作者也在实验部分验证了该方法的不可取。
![算法流程](/img/hand-paper/hand-paper-iccv1-algo.png)

对于初始化的CNN，作者用了单层卷积层加上全连接层。作者也尝试使用CVWW中的复杂网络来初始化，发现performance并没有提升多少（0.5mm），为了保证速度所以选择了该简单的网络。

对于合成深度图的CNN，这个是这篇文章的一大亮点。作者采用了和估计pose相反的结构，即先全连接，然后不断地上采样、卷积，直至得到原图大小的feature map，结构见下图：
![合成CNN](/img/hand-paper/hand-paper-iccv1-synthetic.png)

合成的效果看着还不错（见下图，主要误差在边缘，因为原图边缘有噪声），但应该比较难训练，作者利用了一个layer-wise的方法逐层训练初始化，即类似DNN的训练方法，只是每一层的训练为有监督的，把目标深度图降采样作为label（需要加入一个临时的卷积层将多个feature map合成一个）。而且，作者的实验也表明，在标注噪声较大的ICL数据集还不work，合成的深度图会有伪影。
![合成结果](/img/hand-paper/hand-paper-iccv1-synthetic-result.png)

对于回归pose偏置的CNN，作者使用了类似Siamese网络的共享参数结构，同时用stride而不是max pooling来进行降采样，因为空间不变性并不是该任务所需要的。作者并不是直接使用L2误差作为目标函数，而是使用一个截断误差，即当误差下降到原误差的0.6倍则没有惩罚，否则惩罚超出的误差量。这样做的好处是网络更加容易收敛了，同时保证每次更新都能使误差下降（作者在实验部分也证明了这一点）。训练的时候，首先利用初始化CNN输出的pose以及加入高斯噪声的pose作为训练数据。在训练一段时间后加入本网络得到的pose，以及从当前所有样本误差采样得到的误差加入到pose中（这与加入高斯噪声不同，可以帮助纠正初始化误差比较大的pose）。网络结构如下图所示：
![更新CNN](/img/hand-paper/hand-paper-iccv1-update.png)

文章只在NYU上做了实验，结果从原来state-of-the-art的平均20+mm误差下降到16.5mm，实验结果见下图，其他细节见原文。
![结果](/img/hand-paper/hand-paper-iccv1-result.png)

### 13. [Opening the Black Box: Hierarchical Sampling Optimization for Estimation Human Hand Pose (ICCV 2015, oral)](http://www.iis.ee.ic.ac.uk/~dtang/iccv_2015_camready.pdf)
这篇文章是ICL的新工作，主要贡献在于将之前模型拟合方法里的能量函数进行改进，使得搜索hand pose的过程可以从全pose同时遍历变为层级的遍历（从掌心到指尖），这样避免了迭代，可以更加准确且有效率地获取最优的hand pose。算法的主要流程如下：
![算法流程](/img/hand-paper/hand-paper-iccv2-algo.png)

相对于CHI 2015那篇基于golden能量函数的文章，作者提出了更有效率的silver能量函数。它基于pose的层级结构（见下图，将pose参数分成四个集合，逐级求各个参数），利用确定该参数之后所有可能的后继关节点的信息来计算。具体来说，该能量函数分为两项：
- **B(x)：**投影到图像坐标的节点和轮廓之间的距离。这一项使得后继节点尽可能落在手的轮廓内。
- **D(x)：**后继关节点和图像中对应点深度的截断差，类似于hinge loss。

基于silver能量，作者提出了层级采样优化算法（HSO），见下图。按照层级结构，算法利用RF在当前点预测出的分布采样得到一些候选点，并计算他们的能量函数，从中选出最好的点，进入下一层，直到获取所有关节点。其中，RF的叶节点存的是参数的分布。为了正确处理角度的分布，在训练角度参数回归时首先回归3D的偏置，在叶节点聚类的时候再将其转换为角度，这样避免了gimbal lock伪影以及相同角度不同视角下的appearance不同的问题。
![HSO算法](/img/hand-paper/hand-paper-iccv2-hso.png)

作者在三个数据集上做了实验，包括NYU、ICL和MSHD（CHI 2015的数据集）。首先作者评估了M和N对算法性能的影响，发现层级算法能有效地降低误差。和state-of-the-art相比，HSO均outperform之前的算法，除了在NYU上大阈值的时候比TOG的略差（作者argue说图像噪声太多，导致missing pixel，这使得silver能量有时候不work），见下图。同时，作者的算法在CPU上达到50fps。
![结果](/img/hand-paper/hand-paper-iccv2-result.png)

### 14. [3D Hand Pose Estimation Using Randomized Decision Forest with Segmentation Index Points (ICCV 2015)](http://www.dabi.temple.edu/~hbling/publication/hand-pose-iccv15.pdf)
该工作基于CVPR 2014那篇LRF进行了一个优化，就是把原来用非监督方式学习得到的LTM融进了整个RF训练的框架中变成Segmentation Index Points(SIP)，即不再提前确定分裂方式，而是在division节点处对关节点按照测地线距离进行聚类来进行划分。这样做的好处是，不同的手势会有不同的划分结果，能够更好地handle那些view-point变换或标注错误的情况。但缺点是训练时计算量变大了，而且节点处需要存储更多信息。和LRF的区别见下图：
![算法区别](/img/hand-paper/hand-paper-iccv3-algo.png)

算法仅在ICL上与LRF及更早的一些算法做了比较，效果比他们的要好一些（见下图）。其他细节请见原文。
![结果](/img/hand-paper/hand-paper-iccv3-result.png)

### 15. [A Collaborative Filtering Approach to Real-Time Hand Pose Estimation (ICCV 2015)](https://engineering.purdue.edu/cdesign/wp/a-collaborative-filtering-approach-to-real-time-hand-pose-estimation/)
这篇文章的idea比较新，是将hand pose估计类比为推荐系统里的cold-start问题。具体来说，数据集里面深度图的信息对应推荐系统里用户的元数据，而它们的hand pose则代表用户评分。基于这些假设，hand pose的估计相当于对新用户估计他的评分。实际上，这个方法相当于一种变形的最近邻算法，用多个exemplar来估计出hand pose。算法流程如下：
![算法流程](/img/hand-paper/hand-paper-iccv4-algo.png)

为了提供训练集，作者利用手模型来生成数据，生成时对参数加上限制进行随机采样。基于数据集，作者采用OPTICS和DBSCAN两种方法相结合进行聚类得到1000+代表性的exemplar。对于每个exemplar，它的元数据用pose与15个手语手势pose（见下图）的欧几里德距离来表示。而在数据库检索时，算法使用了形状描述子距离：计算深度图的每个FAST特征点的BRIEF特征，样本间的距离即是所有配对的特征的汉明距离的平均值。
![手语](/img/hand-paper/hand-paper-iccv4-asl.png)

对于给定的深度图，首先利用固定阈值分割出手，然后根据重心算出平移的参数，之后利用形状描述子距离算出k近邻（k为距离小于固定阈值在[32, 64]范围内的exemplar数）。最后利用联合矩阵分解和填充算法（JMFC）得到hand pose。令k近邻的pose矩阵为P，和手语手势距离矩阵为D，该算法简单来说就是假设P和D都是由同样的latent变量得到的，即P约等于AC，D约等于AB（其中A、B、C均为矩阵）。把该问题转化为无约束最优化问题，最后用经典的ALS算法进行求解即可。具体推导见原文。

实验方面，作者在自己合成的数据集和自己采集的真实数据集以及进行了测试，证明基于JMFC的方法要优于最近邻（NN）的方法，同时k近邻的结果比使用全部的exemplar（JMFC-full）要好：
![结果1](/img/hand-paper/hand-paper-iccv4-result1.png)

在CVPR 2015的MSRA数据集上，作者的方法比cascade regression的方法略好（其实我并没有看出好多少）：
![结果2](/img/hand-paper/hand-paper-iccv4-result2.png)

### 16. [Depth-based Hand Pose Estimation: Methods, Data, and Challenges (ICCV 2015)](http://www.ics.uci.edu/~jsupanci/#HandData)
该文章主要对hand pose的数据集和方法进行了review（CVPR 2015之前，感觉文章还没一年就已经有点过时了，可见该领域发展之快），并提出了一个比较难的带场景的数据集以及一种简单粗暴好使的NN算法，在很多数据集上均有不错的结果。

作者把目前的测试数据集归为三类：只有articulation变化的，加上viewpoint变化的，以及加上背景聚类变化的（见下表）。文章主要用了ICL（因为用的人多）、NYU（因为variation大且标注准确）、UCI-EGO（因为难）以及自己采集的数据集。对于训练数据，作者分成了四类：真实数据+人工标注（ICL），真实数据+自动标注（NYU），半合成数据（即在真实数据上进行augment）和合成数据（UCI-EGO）。作者也利用libhand生成了大量数据用于训练。
![数据集](/img/hand-paper/hand-paper-iccv5-dataset.png)

在本文里，作者比较了一系列不同的算法，主要focus on在single-frame的方法，且主要为data-driven方法（作者自己实现了基于part model的算法、基于Hough Forest的方法、类似Kinect里的part分类+聚类的方法等）：
![方法](/img/hand-paper/hand-paper-iccv5-methods.png)

作者实现了一种基于NN的算法，主要利用了volumetric exemplar。该方法将训练数据里的手构造成一个个binary voxel grid，被遮挡的设为1，其他为0。测试的时候对于图中每一个voxel grid用最近邻的方法找到汉明距离最小（即相同数值最多）的exemplar作为hand pose。为了加速，作者提出了两种策略：跳过全0或全1的voxel；直接计算2D的L1距离来替代3D的逐像素比较。此外，还可以用近似NN算法进行加速。volume相比于2D window的好处是不用关心尺度问题因为是在真实坐标下，同时该方法也能将背景分开，见下图：
![NN算法](/img/hand-paper/hand-paper-iccv5-volume.png)

对于评估指标，作者主要使用了关节的最大误差。作者发现，20mm接近人类分辨的水平，50mm表示大致正确的hand pose，100mm表示正确的detection结果：
![评估](/img/hand-paper/hand-paper-iccv5-evaluation.png)

作者在不同的数据集评估了各个算法，发现目前在非背景聚类的数据集上各方法已经基本work了。在ICL这种没有viewpoint变化的数据集上，RF的和CNN的效果都可以（85%小于50mm，100%小于100mm），而在NYU上只有CNN比较work（96%小于50mm，100%小于100mm）。但在有背景聚类的两个数据集上，目前效果仍然一般。比较神奇的是，作为baseline的NN算法在这几个数据集上均有不错的结果，除了在NYU上比CNN差了一截（但在mean error上也和CNN接近）。具体结果见下图：
![结果ICL](/img/hand-paper/hand-paper-iccv5-icl.png)
![结果NYU](/img/hand-paper/hand-paper-iccv5-nyu.png)
![结果test](/img/hand-paper/hand-paper-iccv5-test.png)
![结果UCI](/img/hand-paper/hand-paper-iccv5-uci.png)
此外，作者对他们的方法进行了跨数据集测试，发现效果很差（见下图）。这说明目前很多方法实际上都是依赖于训练集分布的。
![cross结果](/img/hand-paper/hand-paper-iccv5-cross.png)

### 总结
hand pose是少数目前CV领域方法较丰富~~没有完全被CNN攻占~~的topic（相比于classification、detection、segmentation、human pose），目前performance也还有提升的空间（指尖点误差仍然比较大）。关于算法，之后用得比较多的分类器仍会是RF和CNN，目前一个大趋势是从data-driven的方法慢慢向着混合的方法发展，因为单独一种方法都有其局限性；另一个趋势是大家发现基于exemplar的方法效果也不错。此外大家也越来越关注于实时的性能，因为毕竟hand pose的主要应用便是HCI，但目前很多算法对于硬件还是有比较大的要求（比如GPU），感觉还不能直接用于移动端。另外，开一下脑洞，多手的tracking以及基于RGB的hand pose（可能很久以前有人做，但是现在荒废了）目前也还是一块空白的区域。以及hand pose和hand gesture结合的方法好像也比较少，不像human pose与action结合那么多。
