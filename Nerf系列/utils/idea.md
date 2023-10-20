# 我大概要做一个什么事？

**稀疏视角+大规模场景**

## 1.点云分支+Nerf分支

> Grid-Nerf就是在TensoRF的基础上加了Nerf分支...很小的改动

那我能不能在Point-Nerf的基础上加上Nerf分支，模拟Grid-Nerf，分成点云分支和Nerf分支？

遇到问题：Point-Nerf代码没有注释且网上说bug很多，也不知道怎么把Point-Nerf用在自己数据集上。

## 2.稀疏视角(深度先验--sfm稀疏深度图)+大规模场景

- ~~DS-Nerf+Grid-Nerf~~  

遇到问题：img2poses跑不出来，内存不够

遇到问题：loda_llff中的load_colmap_depth函数中，有以下限制条件的时候，在rubble和Residence数据集上，深度全部被过滤掉了

```
if depth < bds_raw[id_im-1,0] * sc or depth > bds_raw[id_im-1,1] * sc: # 原来的
  continue
```

但是去掉限制条件感觉又不对..所以深度是怎么从colmap的稀疏点云中得到的？

## 3.稀疏视角(深度先验--深度补全密集深度图)+大规模场景

看到一篇解释深度在Nerf中应用的总结的论文Digging into Depth Priors for Outdoor Neural Radiance Fields，大致意思是深度补全>双目深度估计>单目深度估计，密集深度图>稀疏深度图， 密集视角>稀疏视角，但是在密集视角中加入深度不一定会使得结果更好，有可能变差...

## ~~4.Grid-Nerf~~

1. Nerf分支根本没必要，去掉，然后在Grid分支中加入原本在Nerf分支中的Embedding
2. Grid分支也是分为粗采样和细采样，即sample_pdf

## 5.自适应采样？

WaH-NeRF

## 6.不同的方法解决不同高度拍摄的大场景图片重建

即不死盯着mega/switch-nerf，而是改进citynerf，在citynerf里面加入深度？参考那两篇卫星图像的，先考虑加入深度，DS-Nerf+NerfingMVS+Nerf-w
