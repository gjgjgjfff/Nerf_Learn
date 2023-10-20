# paper

### 2023/10/13

* S3IM: Stochastic Structural SIMilarity and Its Unreasonable Effectiveness for Neural Fields

  * 一种即插即用的，与模型无关的损失
  * 提出问题：
    * 传统Nerf、Neus及它们的改进方法，都是计算每个单一像素点的MSEloss和PSNR，没有利用像素的集体监督，他们认为逐点质量不能反映图片在物理世界中的真实结构信息，而SSIM（一种评价指标，叫结构相似度，是基于亮度、对比度、结构三方面比较，是针对一组像素，不是一个像素）不仅反映了一组像素的结构，而且与人类视觉系统的相关性明显优于 PSNR，但SSIM并没有被用来监督Nerf的训练
  * 贡献：
    * 提出了随机结构相似性指数（S3IM），将多个随机采样的像素点作为一个整体处理，它测量两组随机采样的像素之间的相似性并从中捕获非局部结构相似性信息

  * 实施：
    * 每个batch中，随机选patch_size个像素（这一步执行M次，可以自己设定M），然后把所有这些像素去计算SSIM并取平均，用1-Avg（SSIM）作为S3IMloss。

