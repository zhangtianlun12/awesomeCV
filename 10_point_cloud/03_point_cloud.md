# Point cloud

<h3> Keywords </h3>

__`dat.`__: dataset &emsp; | &emsp; __`cls.`__: classification &emsp; | &emsp; __`rel.`__: retrieval &emsp; | &emsp; __`sem.`__: semantic segmentation
__`ins.`__: instance segmentation &emsp; |__`det.`__: detection &emsp; | &emsp; __`tra.`__: tracking &emsp; | &emsp; __`pos.`__: pose &emsp; | &emsp; __`dep.`__: depth
__`reg.`__: registration &emsp; | &emsp; __`rec.`__: reconstruction &emsp; | &emsp; __`auto`__: autonomous driving
__`oth.`__: other, including normal-related, correspondence, mapping, matching, alignment, compression, generative model...

Statistics: :fire: code is available or the paper is very important

## Table of Contents

- [survey](#survey)
- [benchmark](#benchmark)
- [RGB-D](#RGB-D)
- [Augment](#Augment)
- [classification](#classification)
- [3D-object-detection](#3D-object-detection)
- [Segmentation](#Segmentation)
- [6D-Pose-Estimation](#6D—Pose-Estimation)
- [Geodesic-topology](#Geodesic-topology)

---

## survey

survey/review/overview

- [2017][a review of point clouds segmentation and classification algorithms](https://www.int-arch-photogramm-remote-sens-spatial-inf-sci.net/XLII-2-W3/339/2017/isprs-archives-XLII-2-W3-339-2017.pdf)

- 论文从点云采集方式(image-derived,LiDAR,RGB-D,SAR)，点云数据集，点云分割方法（edge-based,regin growing,model fitting,unsupervised clustering-based），点云语义分割方法等，是个人认为是点云语义分割入门材料。

- 点云语义分割综述.
  - [2019][A Review of Point Cloud Semantic Segmentation](https://arxiv.org/pdf/1908.08854.pdf)

- 点云对齐领域的论文综述。
  - [2019][Target-less registration of point clouds: A review](https://arxiv.org/pdf/1912.12756.pdf)

- 3D检测框恢复6D姿态估计。
  - [2020][A Review on Object Pose Recovery: from 3D Bounding Box Detectors to Full 6D Pose Estimators](https://arxiv.org/pdf/2001.10609.pdf)

- 自动驾驶中的点云综述,加拿大滑铁卢大学提出。包括数据集，评价方法，分类，目标检测和分割。对点云的处理有框架性的概括。:star: :star: :star: :star: :star:
  - [Deep Learning for LiDAR Point Clouds in Autonomous Driving: A Review](https://arxiv.org/pdf/2005.09830.pdf)

---

## benchmark

- <https://paperswithcode.com/task/3d-part-segmentation/latest>
- <https://paperswithcode.com/area/computer-vision/3d>
- <https://paperswithcode.com/task/3d-object-detection>
- <http://kaldir.vc.in.tum.de/scannet_benchmark/>
- <https://yochengliu.github.io/files/Report_JIANGMEN_2019-08.pdf>

---

## RGB-D

- CVPR2019论文，出自于大名鼎鼎李飞飞组，提出模型，一个用于估计RGB-D图像中已知目标6D姿态的通用框架
（类似于视频处理的two-stream，分别处理RGB图像和深度图像,DenseFusion融合两路特征）。在YCB-Video
和LineMOD数据集验证测试。论文中6自由度指李群SE(3)（包括旋转和平移），目标是求相机的运动姿态。
  - [CVPR2019][DenseFusion: 6D Object Pose Estimation by Iterative Dense Fusion](https://arxiv.org/pdf/1901.04780.pdf)

---

## Augment

- 香港中文大学提出的，点云增强方法。
  - 点云稀疏性，单点云规模一般较大，需要做增强？构建大规模场景数据集，相比ImageNet的困难在哪里？
  - PointAugment framework中，feedback的理论依据？ #TODO
  - 论文只是在ModelNet40做验证，是否过拟合，有更复杂一些数据集可验证性能？
  - [2020][CVPR][PointAugment: an Auto-Augmentation Framework for Point Cloud Classification](https://arxiv.org/pdf/2002.10876.pdf)
  - <https://github.com/liruihui/PointAugment>

## classification

- 斯坦福大学提出，点云领域的经典论文，用于解决点云分类，语义分割和目标识别(分类和分割任务共用backbone)。
PointNet之前的方法集中在点云投影二维平面，点云划分Voxel等方式。而本文直接对点云操作。点云具有无序，局部相关性，平移不变性(旋转，平移)三个特征。论文同时提出两个结论：(1)PointNet的网络结构能够拟合任意的连续集合函数，(2)PointNet能够总结出表示某类物体形状的关键点，基于这些关键点PointNet能够判别物体的类别。
这样的能力决定了PointNet对噪声和数据缺失的鲁棒性。
  - 两次用到T-Net,T-Net是3*3的矩阵，对每个点云做旋转对齐。实验分析T-Net没有明显作用。
  - 缺点卷积计算是conv1d，单独对每个点云做卷积，没有利用点云的周围特征。在背景噪声复杂时，不具有鲁棒性。
  - 所有点云提取全局特征，而不是局部特征。
  - 点云存储位置相邻并不意味真实空间相邻。
  - [CVPR2017][PointNet: Deep Learning on Point Sets for 3D Classification and Segmentation](https://arxiv.org/pdf/1612.00593.pdf)[__`cls.`__ __`seg.`__]  :star: :star: :star: :star:

- 斯坦福大学提出PointNet升级版PointNet++。主要解决两个问题：a对点云如何分组，b.如何提取局部特征。模型主要包括
  - sampling layer(farthest point sampling {FPS} algorithm):选择N个中心点。
  - grouping layer：Ball query生成N个局部区域，参数包括：中心点的数量K，球的半径。
  - PointNet layer：输入网络之前的点云球体，坐标会更新为球中心的相对坐标，类似Batch Norm。
  - hierarchical structure，PointNet++的backbone，由sampling&grouping&PointNet交叠组成。
  - 非均匀点云的处理，论文使用Multi-scale grouping (MSG) and Multi-resolution grouping (MRG)多尺度处理。MSG+DP相对单尺度SSG，有更好的鲁棒性。
  - [NIPS2017][PointNet++: Deep Hierarchical Feature Learning on Point Sets in a Metric Space](https://arxiv.org/pdf/1706.02413.pdf)[__`cls.`__ __`seg.`__] 

- 俄勒冈州立大学机器人技术与智能系统（CoRIS）研究所的研究者提出了PointConv，基于2D卷积推导出3D点云表达式，将3D卷积看做由局部点3D坐标的非线性函数(包括权重和密度)，
可以高效的对非均匀采样的3D点云数据进行卷积操作，该方法在多个数据集(ModelNet40、ShapeNet和ScanNet)上实现state-of-art。
主要贡献：
  - 1、提出逆密度重+权重的卷积操作PointConv，近似拟合3D连续卷积。
  - 2、通过改变求和顺序，提出了高效PointConv。
  - 3、将PointConv扩展到反卷积PointDeconv，以获得更好的分割结果。
  - 论文已经开源tensorflow和pytorch源代码，可用于评估性能。
  - [CVPR2019] [PointConv: Deep Convolutional Networks on 3D Point Clouds](https://arxiv.org/abs/1811.07246)
  - [[tensorflow](https://github.com/DylanWusee/pointconv)] [__`cls.`__ __`seg.`__] :fire:
  - [[pytorch](https://github.com/DylanWusee/pointconv)

- 论文提出ShufflePointNet，基于二维分组卷积和论文ShuffleNet,在三维点云的应用。
  - [2019.09][Go Wider: An Efficient Neural Network for Point Cloud Analysis via Group Convolutions](https://arxiv.org/pdf/1909.10431.pdf)

- 法国国家航空航天中心提出。
  - [2019][ConvPoint: continuous convolutions for point cloud processing](https://arxiv.org/pdf/1904.02375.pdf)

---

## 3D-object-detection

- 香港中文大学提出，基于Point cloud->3D Box的3D目标检测方法，基本原理类似2D RCNN结构，两阶段方式：stage-1 基于bottom-up，对点云数据分割前景和背景，生成3D建议候选框，stage-2
网络将全局语义特征和局部空间特征结合起来，对3D候选框进行优化。

  - stage-1 bottom-up strategy依据：3D场景目标之间没有压盖，3D语义掩码可以直接从3D bounding box 标注中获取，所以3D目标检测问题可以转换为3D语义分割问题。而在2D目标检测中，边界框只能为2D语义分割提供弱监督。
  - PointRCNN的backbone基于pointnet++/VoxelNet，stage-1包括Foreground point segmentation和Bin-based 3D bounding box generation分支，分布完成前景/背景分割和3D bounding box。Foreground point segmentation使用focal loss来解决室外场景中，由前景点过少带来的类别不平衡问题。
  - stage-2结合Semantic Features，Foreground Mask，3D RoIs，生成Local Spatial Points和Semantic Features，最终完成3D bounding box优化。

  - [2019][CVPR][PointRCNN: 3D Object Proposal Generation and Detection from Point Cloud](https://arxiv.org/pdf/1812.04244.pdf)
  - https://github.com/sshaoshuai/PointRCNN

- Facebook何凯明等人提出VoteNet,直接基于点云的3D目标检测模型(无image输入，话说何凯明开始多领域作战)。点云一般是稀疏性，直接
做检测具有较高难度。论文基于PointNet++,提出VoteNet，由deep point set networks 和 Hough voting组成。论文在ScanNet和
SUN RGB-D具有良好表现。 CNN在3D object classification ,3D object detection和3D semantic segmentation均已有所表现，
下一个战场应该是3D Instance Segmentation.

- 
  - [Deep Hough Voting for 3D Object Detection in Point Clouds](https://arxiv.org/pdf/1904.09664.pdf)

- 
  - [2019.07][Going Deeper with Point Networks](https://arxiv.org/pdf/1907.00960.pdf)
  
- 港中文的贾佳亚教授实验室作品，思想类似于类似于RCNN在二维目标检测方法，采用两阶段方法：第一阶段基于生成粗略预测结果，第二阶段结合Raw 点云生成精确检测结果。
  2D目标检测的更多trick比如Cascade RPN什么时候用在三维上？拭目以待。
  
  - [2019.09][Fast Point R-CNN](https://arxiv.org/abs/1908.02990) [__`det.`__ __`aut.`__]
  
- 二维图像的YOLO算法应用于三维点云检测和跟踪。

  - [2019][Complexer-YOLO: Real-Time 3D Object Detection and Tracking on Semantic Point Clouds](https://arxiv.org/abs/1904.07537) [[pytorch](https://github.com/AI-liu/Complex-YOLO)] [__`det.`__ __`tra.`__ __`aut.`__] :fire:

- 商汤，香港中文大学联合实验室提出，论文将Grid-based(Voxel-based)和Point-based结合，通过两个阶段策略：the voxel-to-keypoint 3D scene encoding，the keypoint-to-grid RoI feature abstraction
实现3D的目标检测。
  - 话说串行的优势在哪里，为什么不用并行？
  - [PV-RCNN: Point-Voxel Feature Set Abstraction for 3D Object Detection](https://arxiv.org/pdf/1912.13192.pdf)

---

- RGB-D Image Analysis and Processing,chapter 3

  - [RGB-D image-based Object Detection: from Traditional Methods to Deep Learning Techniques](https://arxiv.org/pdf/1907.09236.pdf)

---

## Segmentation

- CVPR2018论文，巴黎东部大学和巴黎圣母大学提出一种点云分割方法：1.将三维点云首先分成简单几何体组成的简单形状，即为超点(superpoints)。 2.由具有丰富属性的超边连接相邻的超点形成的超点图(superpoints graph, SPG)
3.由graph CNN对superpoints提取上下文信息，分类并得到语义标签。
  - superpoints基本思路类似于图像分割领域的超像素Superpixel。
  - [2018][CVPR][Large-scale Point Cloud Semantic Segmentation with Superpoint Graphs](https://arxiv.org/pdf/1711.09869.pdf)  
  - https://github.com/loicland/superpoint_graph

- CVPR2018,上海交通大学卢策吾团队MVIG实验室提出PointSIFT。论文基于SIFT思路推广到3D点云，编码点云8个方向信息，自适应合适的表征尺度。论文设计的PointSIFT module可集成在PointNet系列框架中。
网络模型的基本架构有三部分组成，PointSIFT模块，Set abstraction (SA,PointNet++下采样过程)和feature propagation(FP，PointNet++上采样过程)。
  - 论文借助SIFT思想表达三维信息，更多的2D方法可以用于3D CNN，比如PointSIFT类似encoder-decoder的架构，各个feature level之间连接可以更好的特征融合。
  - 3D点云由于稀疏性和无序性，不能直接使用CNN，如何提取高效特征并借助于2D语义分割开放的研究思路，不失为好的研究方向。
  -[2018][CVPR][PointSIFT: A SIFT-like Network Module for 3D Point Cloud Semantic Segmentation](https://arxiv.org/pdf/1807.00652.pdf)[__`sem.`__]:fire:          

- 
  - [SqueezeSeg: Convolutional Neural Nets with Recurrent CRF for Real-Time Road-Object Segmentation from 3D LiDAR Point Cloud]

- 澳大利亚阿德莱德大学,腾讯优图，香港中文大学等(沈春华项目组)，提出ASIS,联合语义分割和实例分割的点云分割方法。实例分割主要通过学习的instance embeddings实现(Metric Learning)。
  个人感觉instance segmentation包含semantic segmentation，前者可直接推导后者。联合学习或联合损失函数可以更好网络训练？
  
  - [CVPR2019][Associatively Segmenting Instances and Semantics in Point Clouds](https://arxiv.org/pdf/1902.09852.pdf)[__`sem.`__.__`ins.`__]:fire:

- PointNet系列论文只能在small-scale点云实现语义分割，对于large scale只能通过切分方式，这种切分会让一个物体变成两个物体。论文任务传统降采样方法代价昂贵，特征学习模块代价高，特征感受野受限。
  - 论文提出随机降采样策略：
  - 论文提出有效的局部特征聚合模块使得网络模型适用于增加大规模点云感受野。
  - [2020][CVPR][RandLA-Net: Efficient Semantic Segmentation of Large-Scale Point Clouds](https://arxiv.org/pdf/1911.11236.pdf)
  - <https://github.com/QingyongHu/RandLA-Net>

---

## 6D-Pose-Estimation

- CVPR2019论文，浙江大学提出6D Pose Estimation，输入2D图片和3D模型特征数据，在3D空间中检测目标的位置和姿态，应用之一是实现AR中目标的运动估计。

  - [CVPR][2019][PVNet: Pixel-wise Voting Network for 6DoF Pose Estimation](https://arxiv.org/pdf/1812.11788.pdf)

---

## Geodesic-topology

- CVPR2019 oral,旷视西雅图研究院提出的基于测地距离的点云分析深度网络GeoNet，个人理解主要针对不连续点云建立拓扑逻辑关系，可用于点云上采样、法向量估计、网格重建及非刚性形状分类等。
  - [2019][CVPR][GeoNet: Deep Geodesic Networks for Point Cloud Analysis](https://arxiv.org/pdf/1901.00680.pdf)

---

## 待阅读

pvnet,SqueezeSeg ，20190723分享
https://zhuanlan.zhihu.com/p/44809266

VoteNet层次理解
Unstructured point cloud semantic labeling using deep segmentation networks
Generalizing discrete convolutions for unstructured point clouds
Point Cloud Oversegmentation with Graph-Structured Deep Metric Learning
PointPillars: Fast Encoders for Object Detection from Point Cloud
RepNet: Weakly Supervised Training of an Adversarial Reprojection Network for 3D Human Pose Estimation	cvpr2019
SGPN: Similarity Group Proposal Network for 3D Point Cloud Instance Segmentation

Deep hough voting for 3d object detection in point clouds

3124,L3-Net: Towards Learning based LiDAR Localization for Autonomous Driving,Weixin Lu (Baidu ADU)

SDRSAC: Semidefinite-Based Randomized Approach for Robust Point Cloud Registration without Correspondences,Huu Minh Le (Queensland University of Technology)*
PointNetLK: Robust & Efficient Point Cloud Registration using PointNet,Hunter M Goforth (Carnegie Mellon University)*

,Object Tracking by Reconstruction with View-Specific Discriminative Correlation Filters,Ugur Kart (Tampere University of Technology)*

Learning Depth with Convolutional Spatial Propagation Network，Xinjing Cheng（baidu）2018，arkiv
