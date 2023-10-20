- 针对citynerf的不同高度的图片，不能用传统的load_llff的方法得到near/far，因为传统方法near/far是根据bds得到的，bds是colmap重建的所有稀疏3D点中，距离所有相机中心最近和最远得到的，这个距离是相对于相机中心的距离，比如，现在有离物体很近的相机也有离物体很远的相机，而所有重建的点中，离近相机中最近的是0.01，离远相机最远的是3，那么最终得到的near是0.01，far是3，则在采样点时，所有光线都在离相机原点的0.01~3的范围内采样(那么就有问题，离远相机0.01的地方可能是离物体很远的空气，离近相机3的地方可能已经到地底下了)，所以只能用他的把整个地球视作球体的方法？
- 正常的数据每条光线的near和far是不变的，而citynerf的near和far是变化的，所以citynerf的bbox不能根据near/far计算得到

### 1.环境

- 在一卡zbz上装citynerf的环境和tcnn库，暂时就用torch-ngp

- 2023/9/28：一卡服务器上tcnn库安装完成，cmake version 3.22.5，用gcc/g++10能成功 ，https://github.com/NVlabs/instant-ngp/issues/119

### 2.接下来做什么

- citynerf+tcnn的哈希编码
  - 2023/9/28：citynerf+哈希编码改完并跑通。但mipnerf的输入是6维的，tcnn的哈希编码输入只能是2或3维，所以需要把mipnerf的东西去掉，虽然速度明显提升，原来20w个iter要40h，现在只要4h，而且收敛很快，2w个iter就收敛，但效果变差很多，不知道是mipnerf没去干净还是去掉后有问题。
  - 2023/9/29：看懂mipnerf，能不能把mip_embedding用tcnn.encoding替换？这样就可以不用去掉citynerf中mipnerf的部分。不行！原理不一样，不能直接把mip_embedding换成tcnn.encoding，也就是说没办法加入ngp的哈希编码。---卒！

* 2023/10/9：看懂mipnerf360，看看他的Distortion Loss和proposal Net，这些都可以用起来，不过mipnerf360又是另一个框架...Distortion Loss可以参考DVGO-V2里面的torch_efficient_distloss，或者ngp-pl里的Nerf_loss，或者grid-nerf里好像也加了distloss，或者multinerf里  ---看不懂loss，加进去有问题，只收敛到了局部最优解
* 2023/10/15：
  
  * 原始nerf+ngp的hashencoding跑citynerf，但跑崩了（zbz一卡下BungeeNeRF/logs/train_hash_stage0_Nerf）
    * 感觉是射线没有打到物体上？就直接用nerf跑citynerf，看看是hashencoding的问题还是换nerf没换好的问题？
  * 看来是hashencoding的问题，用2层64的Nerf+hashencoding跑一下，2x64是ngp的标配，还是跑崩了...（zbz一卡下BungeeNeRF/logs/test_2_64）

* 2023/10/16：
  * 原始nerf跑citynerf，效果变好，应该是因为用的MLP层数变多

第一阶段（zbz一卡下BungeeNeRF/logs/train_stage0_Nerf），20w iter

|          | PSNR （越大越好） | SSIM（越大越好） | lpips（越小越好） | 时间（单卡3090） |
| -------- | ----------------- | ---------------- | ----------------- | ---------------- |
| 原文     | 23.441            | 0.743            | 0.225             | 9h               |
| 原始Nerf | 24.788            | 0.815            | 0.152             | 9h               |

* 2023/10/17：
  * 在原始nerf基础上改成hash编码，看看是哪的问题，也是不行，说明我代码没有改对
  * 把Nerf的forword改写成torch-ngp中network的样子，再试试，还是不行，应该不是网络的问题，是render那些的问题？
  * 现在思路，在torch-ngp上改citynerf，主要有两个工作，一个是读数据，一个是采样点，先暂时放下这个
* 2023/10/18：
  * 原始Nerf加上平面特征，效果没啥变化，但速度变慢（zbz一卡下BungeeNeRF/logs/train_stage0_Nerf_plane）

第一阶段（zbz一卡下BungeeNeRF/logs/train_stage0_Nerf_plane），20w iter

|                   | PSNR （越大越好） | SSIM（越大越好） | lpips（越小越好） | 时间（单卡3090） |
| ----------------- | ----------------- | ---------------- | ----------------- | ---------------- |
| 原文第一阶段      | 23.441            | 0.743            | 0.225             | 9h               |
| 原始Nerf+平面特征 | 24.721            | 0.811            | 0.152             | 11h              |

* 2023/10/19：
  * 给sigma的MLP加上平面特征训全部图片，（zbz一卡下BungeeNeRF/logs/train_all_Nerf_plane），这么做的出发点是将原来的串行训练改为并行训练，避免多个stage速度成倍增加

|                                         | PSNR （越大越好） | SSIM（越大越好） | lpips（越小越好） | 时间（单卡3090） |
| --------------------------------------- | ----------------- | ---------------- | ----------------- | ---------------- |
| 原文第四阶段                            | 24.091            | 0.770            | 0.174             | 9+18+35+55h      |
| 原始Nerf+sigma的MLP平面特征训练全部图片 | 23.588            | 0.730            | 0.211             | 13h              |

* 2023/10/20：
  * 给sigma和color的MLP加上平面特征用一个MLP训全部图片，（zbz一卡下BungeeNeRF/logs/train_all_Nerf_plane2）

|                                                | PSNR （越大越好） | SSIM（越大越好） | lpips（越小越好） | 时间（单卡3090） |
| ---------------------------------------------- | ----------------- | ---------------- | ----------------- | ---------------- |
| 原文第四阶段                                   | 24.091            | 0.770            | 0.174             | 9+18+35+55h      |
| 原始Nerf+sigma和color的MLP平面特征训练全部图片 | 23.661            | 0.735            | 0.206             | 13.5h            |
