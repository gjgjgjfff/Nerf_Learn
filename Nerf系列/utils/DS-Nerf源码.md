## 实验

- 以10为间隔取Resdience的图片，跑一下DS-Nerf。
  > 遇到问题：有的图片没有特征点，从而在images.txt中没有它们，没有pose，这在py文件处理colmap文件时，可能会出现序列index越界的问题。
  > 
  > 解决：把那些没有特征点的图片删掉重跑
  > 遇到问题：loda_llff中的load_colmap_depth函数中，有以下限制条件的时候，在rubble和Residence_sparse_gap10数据集上，深度全部被过滤掉了
  > 
  > ```
  > if depth < bds_raw[id_im-1,0] * sc or depth > bds_raw[id_im-1,1] * sc: # 原来的
  >   continue
  > ```
  > 
  > 解决：去掉限制条件后，跑起来了，但在Residence_sparse_gap10上效果很差，所以应该设置怎样的深度限制条件？

- 四卡/八卡 服务器都不能用来跑img2poses，内存不够

## 疑惑

- 如何从colmap得到深度值？
![截图](f29cbabde65f023c184c14cac04320d6.png)