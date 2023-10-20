# Nerf源码

### 1.光线生成

- Nerf中是求出了每个像素的rays_o和rays_d。对于每张图片都有其对应的相机外参pose(即那个4x4的矩阵)，Nerf中，对于每个pose，都生成rays_o和rays_d，都是HxWx3，每张图片的rays_o是相同的，rays_d是根据小孔成像原理求解的像素平面上一点到相机光心的方向，然后利用相机外参转置矩阵将相机坐标转换为世界坐标，之后把所有图片的所有像素的rays_o和rays_d都保持起来，训练的时候是每个像素每个像素的取。

### 2.读入llff数据

- [参考](https://zhuanlan.zhihu.com/p/593204605)