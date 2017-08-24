## C3D

---

#####source code：[https://github.com/facebook/C3D/tree/master/C3D-v1.1](https://github.com/facebook/C3D/tree/master/C3D-v1.1)

#####example:  [https://github.com/facebook/C3D/tree/master/C3D-v1.1/examples/c3d_ucf101_finetuning](https://github.com/facebook/C3D/tree/master/C3D-v1.1/examples/c3d_ucf101_finetuning)

主要是修改data-list,匹配上video-data-layer读入的数据形式。如下：

    
    （子路径，总帧/关键帧数，类别号）
    ApplyEyeMakeup/v_ApplyEyeMakeup_g08_c01/ 1 0
    ApplyEyeMakeup/v_ApplyEyeMakeup_g08_c01/ 17 0
    ApplyEyeMakeup/v_ApplyEyeMakeup_g08_c01/ 33 0
    ...............

frame的路径由video-data-layer里的root-folder和子路径组成。
参数说明：


    layer {
      name: "data"
      type: "VideoData"
      top: "data"
      top: "label"
      include {
    phase: TRAIN
      }
      transform_param {
    crop_size: 112 #送入网络前 resize 112x112
    mean_value: 128 #送入网络前每个pixel减去均值
    mirror: true # flipping
      }
      video_data_param {
    source: "train_01.lst" # data_list的路径
    root_folder: "/data/users/trandu/datasets/ucf101/frm/" #frame存储的路径
    new_length: 8 # 每个送入网络的cube的帧数
    sampling_rate: 2 # 取frame时的间隔数
    new_height: 128 #resize前，对帧做 random crop的h
    new_width: 171 #同上 为w
    use_image: true #使用帧图像，False时为video作为输入
    show_data: false # TRUE时可视化
    batch_size: 24  # mini_batch_size
    shuffle: true  #对data_list做shuffle
      }
    }

C3D很难收敛，建议先下载这个repo里面的pretrained model。




C3D pytorch version example:https://github.com/whitesnowdrop/c3d_pytorch




---
## Optical flow: ##

###  1，先抽光流拿到光流图像，再送入训练/抽特征：
    
####  ◇抽光流 [https://github.com/wanglimin/dense_flow](https://github.com/wanglimin/dense_flow)

conmand：
       
    ./denseFlow_gpu -f test.avi -x tmp/flow_x -y tmp/flow_x -i tmp/image -b 20 -t 1 -d 0 -s 1
    
	# f,x,y,i： 为视频，存储x轴光流场图像，存储y轴光流场图像以及帧图像的路径
	# b 光流场的最大值/最小值，即大于b的都为b，小于-b的都为-b
	# t 光流算法，0为farnback , 1为tvl1 ，2为alg_brox
	# d gpu设备号
	# s 计算光流场的间隔帧
	
	    
    

####  ◇训练光流图像的代码，主要是video_data_layer和model的第一层不一样：
   
   参考：TSN的前面处理部分：
  [ https://github.com/yjxiong/temporal-segment-networks/blob/master/models/ucf101/tsn_bn_inception_rgb_train_val.prototxt]( https://github.com/yjxiong/temporal-segment-networks/blob/master/models/ucf101/tsn_bn_inception_rgb_train_val.prototxt)
 
###  2，抽光流+直接训练/抽特征：需要自己写

比如基于pytorch，可以使用opencv的python version Optical flow wrapper：


    
    dtvl1=cv2.createOptFlow_DualTVL1()
    flowDTVL1=dtvl1.calc(frame_0,frame_1,None)
    
抽完光流，需要展开成0~255，作为光流图像来计算。



