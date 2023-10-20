# Grid-Nerf就是TensoRF+Nerf分支

Grid Branch可以理解为：自己定义了一个辐射场，每个点包括密度特征和外观特征，我通过得到每条光线上的采样点的密度和外观特征，然后用这些特征得到用于体渲染的体密度和rgb值

## Bug

- 没有训练时下采样的步骤。航拍数据集分辨率太大了，每个像素一条光线的话，CPU资源根本不够，所以必须在训练时下采样，我是按照TensoRF的写法直接把长宽都/scale，还有生成all_rgbs那块也要对图片resize，focal也/scale，也不知道对不对？应该对吧？
- Grid-Nerf的train_dataset和test_dataset中，train包含test的数据，已改正。

## 实验

- LandMark_new中 
  - log/Residence_0
    - far = 10
    - 按照原文生成的json和train_idx，test_idx，且收集时train+=test，这时train_idx本身重复了两次，和test_idx也全部重复，train_idx只用了少量图片(约二百多张，然后重复两次，就是四百多张)，test_idx就更少了(6张)
    - lb = [-5.5, -5, -6] ub = [5.5, 5, 6]
    - downsample_train = 8
    - PSNR 4999 mean 17.8
  - log/Residence_all_json
    - far = 10
    - 把生成所有的pose然后统一划分train_idx和test_idx，这时train_idx本身没有重复，和test_idx也没有重复，train_idx用了大部分图片(2065张)，test_idx也用了不少图片(517张)
    - lb = [-5.5, -5, -6] ub = [5.5, 5, 6]
    - downsample_train = 8
    - PSNE 4999 mean 12.4
  - log/Residence_all_json_1
    - far = 200
    - 和log/Residence_all_json一样
    - lb = [-5.5, -5, -6] ub = [5.5, 5, 6]
    - downsample_train = 8
    - PSNE 4999 mean 12.1
  - log/Residence_all_json_2
    - far = 200
    - 和log/Residence_all_json一样
    - lb = [-55, -50, -60] ub = [55, 50, 60]
    - downsample_train = 8
    - PSNE 4999 mean 16.2
  - log/Residence_all_json_3
    - far = 100
    - 和log/Residence_all_json一样
    - lb = [-55, -50, -60] ub = [55, 50, 60]
    - downsample_train = 8
    - PSNE 4999 mean 16.1
  - log/Residence_all_json_4
    - far = 200
    - 和log/Residence_all_json一样
    - lb = [-55, -50, -60] ub = [55, 50, 60]
    - focal x10 
    - downsample_train = 8
    - PSNE 4999 mean 14
  - log/Residence_all_json_5
    - far = 200
    - 和log/Residence_all_json一样
    - lb = [-70, -70, -70] ub = [70, 70, 70]
    - downsample_train = 8
    - PSNE 4999 mean 15.9
  - log/Residence_all_json_6
    - far = 200
    - 和log/Residence_all_json一样
    - lb = [-30, -30, -30] ub = [30, 30, 30]
    - downsample_train = 8
    - PSNE 4999 mean 16.3
  - log/Residence_all_json_7
    - far = 200
    - 和log/Residence_all_json一样
    - lb = [-15, -15, -15] ub = [15, 15, 15]
    - downsample_train = 8
    - PSNE 4999 mean 16.7
  - log/Residence_all_json_8
    - far = 200
    - 和log/Residence_all_json一样
    - lb = [-15, -15, -15] ub = [15, 15, 15]
    - downsample_train = 8
    - pose / 8
    - PSNE 4999 mean 15.4
  - log/Residence_all_json_9
    - far = 200
    - 和log/Residence_all_json一样
    - lb = [-15, -15, -15] ub = [15, 15, 15]
    - downsample_train = 8
    - resMode = [1,2,4]
    - PSNE 4999 mean 15.4
  - 会不会是下采样了8倍导致精度大幅度损失？用Residence_sparse_gap10来试下，下采样2倍。log/Residence_sparse_gap10_all_json_0
    - far = 200
    - 把生成所有的pose然后统一划分train_idx和test_idx，这时train_idx本身没有重复，和test_idx也没有重复，train_idx用了205张图片，test_idx用了52张
    - lb = [-55, -50, -60] ub = [55, 50, 60]
    - downsample_train = 2
    - PSNE 4999 mean 14.8
  - 会不会是下采样了8倍导致精度大幅度损失？用Residence_sparse_gap10来试下，下采样8倍。log/Residence_sparse_gap10_all_json_1
    - far = 200
    - 和Residence_sparse_gap10_all_json_0一样
    - lb = [-55, -50, -60] ub = [55, 50, 60]
    - downsample_train = 8
    - PSNE 4999 mean 15.1
  - 会不会是经过colmap再下采样导致pose不准确，所以先下采样8倍，再经过colmap。 log/Residence_sparse_gap10_downsample8_all_json_0
    - far = 200
    - 和Residence_sparse_gap10_all_json_0一样
    - lb = [-55, -50, -60] ub = [55, 50, 60]
    - PSNE 4999 mean 14.9

放弃！
