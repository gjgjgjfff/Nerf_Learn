在citynerf里面加入深度？参考那两篇卫星图像的，先考虑加入稀疏深度，~~DS-Nerf+citynerf+Nerf-w~~    NerfingMVS+citynerf+Nerf-w

# 一、整体流程：

- 先复现一下最好的citynerf，看看和原论文差多少
- 先真正看懂两篇论文，画出大致pipline
- ~~看懂两篇论文对应的关键代码，包括DS-nerf中的ray_depth是什么？深度是怎么计算的？~~     DS-Nerf放弃，换成NerfingMVS
- 因为citynerf是在mipnerf基础上改进的，有radii这一项，不知道能不能加深度，先尝试改通citynerf+NerfingMVS代码
- citynerf+NerfingMVS跑一下，看看效果
- 最后再加上Nerf-w跑一下，看看效果

# 二、执行情况：

### 1.复现出最好的citynerf：

|阶段|下采样倍数|PSNR|
|:--:|:--:|:--:|
|stage0|2|训练：最大22.785，测试：22.050|
|stage0|3|训练：最大23.510，测试：22.874|
|stage1|3|训练：最大23.502，测试：22.943|

### 2.用NerfingMVS跑大场景

|阶段|下采样倍数|PSNR|
|:--:|:--:|:--:|
|stage0|3|训练：最大22.863，测试：19.386|
|stage3|3|训练：最大20.377，测试：17.896|
|test(用最高的高度拍摄的59张图片)|3|训练：最大27.249，测试：25.502|

- 结论：不能把不同高度拍摄的图片一次加入网络求出的深度作为深度监督，需要分开计算，怎么分开？

### 3.看懂两篇论文，画出大致pipline

### 4.用哪篇论文的深度？

- ~~DS-Nerf是最简单的，但是不同高度拍摄的图片进去后有bds有问题，而且是稀疏深度，效果肯定没有稠密深度好，而且不适合360度环绕拍摄的情况，所以先放弃~~
  - ~~stage0单独放进去可以的~~
  - ~~stage1单独放进去不行，应该是要求同一高度拍摄的图片才行~~
- NerfingMVS
  - 先跑起来源代码
  - 跑一下56Leonard看看输出的深度图是什么样子？还行
  - 跑一下56Leonard全部图片看看PSNR能有多少，用near_bound和far_bound，下采样4倍：test：17.89，train：最大20.3
  - 跑一下56Leonard stage0看看PSNR有多少，用near_bound和far_bound，下采样3倍：test：19.38，train：最大22.86
  - 跑一下56Leonard stage0看看PSNR有多少，用bds求出来的near和far，下采样3倍：test：，train：最大
- Dense depth priors，自己的数据怎么生成深度，再其次
- monosdf
- satnerf、spsnerf，最后选

### 5.怎么在citynerf的基础上加入深度

- citynerf中的near/far具体代表什么？换算成世界坐标是多少米？还是什么东西？NerfingMVS中的深度值具体代表什么？换算成世界坐标是多少米？还是什么东西？ 只要都是用colmap跑出来的，深度也是用colmap稠密重建监督的，单位就一样
- 按理来说，用bds跑出的near/far要包含所有的深度值，但用不同高度拍摄的图片跑出来不是这样，所以不能不同高度拍摄的图片一次性求深度，那咋办？分开加？

# 三、看论文/代码时的思考：

### 1.citynerf

- raw2output需不需要改？因为现在是所有输出头一起做loss，用不用给他们分开做loss？
- 把citynerf的深度图可视化出来？

### 2.DS-Nerf

- 看到一个很好的可以放在论文中的话 [这里](https://github.com/dunbar12138/DSNeRF/issues/18)

![83f33bb174dbdf232aa30d35e85bbb07](D:\SCUT\note\CV\Nerf系列\idea\citynerf+深度+Nerf-w\83f33bb174dbdf232aa30d35e85bbb07.png)

### 3.NerfingMVS

- 原文的任务是深度估计，所以是用Nerf在给定视角合成的RGB图像与GTRGB图对比，然后对预测的深度滤波得到更好的深度图。那么，我们的任务是新视图合成，可以用GT的深度图和Nerf估计的深度图进行对比，然后对预测的RGB图进行滤波得到更好的RGB图吗？
- bds应该是点的bbox，和深度的最大值最小值应该差不多

# 四、能想到的卖点

### 1.深度标签往往是很难获得的，需要大量的人力物力，这个网络不需要GT深度

### 2.室外大规模场景还没有用过深度监督
