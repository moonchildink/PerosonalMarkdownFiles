# NLA-SLR论文阅读

## Related works

![image-20240130210200792](C:\Users\29236\AppData\Roaming\Typora\typora-user-images\image-20240130210200792.png)

1.   Sign Language Recognition:在VKNet(Video-Keypoint Network)中使用S3D(在I3D基础上的改进，将部分3DCNN调整为2DCNN，使得在算力要求与性能之间取得了较好的平衡)，
2.   Word Representation Learning: 基于word2vec，使用fastText，to pre-compute gloss(word) representations.
3.   Vision-Language Models ：using CLIP,to exploit the implicit knowledge included in glosses(sign lables),which is distinct from previous works on vision-language modeling.
4.   Multi-label Classification: 实际的手语动作可能对应多种不同的含义，因此，本论文中使用了glosses中的语言信息来区分VISigns()

## Methodology

作者提出了Natural Language-assisted sign language recognition网络，简称NLA-SLR. 该网络主要有以下构成：data pre-processing which generates video-keypoints pairs as inputs; a video-keypoint network which takes video-keypoints pairs of various temporal receptive fields as inputs for vision feature extraction; a head network containing a language-aware label smoothing branch and inter-modality mixup branch. 



### DataPreprocessing

给定视频$V \in \mathbb{R}^{T \times H_V \times W_V\times 3} \ $ with  $T = 64$ frames and a spatial resolution of $H_v = W_V = 224$，使用在COCO-WholeBody上训练的网络HRNet来预测每帧的63个关键点。这一关键点使用热力图$K_t \in \mathbb{R}^{H_K\times W_K \times K}$来表示，其中$H_K = W_K = 112$表示高和宽，$K = 63$表示关键点的数量。使用高斯函数来生成伴有热力图的元素。对于所有的帧，重复这一过程来生成heatmap，and feed it into VKNet to extract more robust vision features.



### Video-KeyPoint Network

![image-20240130221516233](C:\Users\29236\AppData\Roaming\Typora\typora-user-images\image-20240130221516233.png)

VKNet 有两个子网络构成：VKNet-32和VKNet-64，分别使用$(V_{32},K_{32})$和$(V_{64},K_{64})$作为输入。两个子网络的网络结构相同，都拥有two-stream architecture ocnsisting of a video encoder and a keypoint encoder. 在该网络中，使用具有5个blocks的S3D网络作为视频/关键点编码器，VKNet使用了两个在B1到B4中具有*bidirectional lateral connections*的S3D网络，并产生video feature $f^V_{32}(f^V_{64})$ and keypoint feature $f^K_{32}(f^K_{64})$ as output of VKNet-32 and VKNet-64. The final feature   $f$ is the **concatenation** of $f_{32}$ and $f_{64}$.

### Head Network

![image-20240130222658083](C:\Users\29236\AppData\Roaming\Typora\typora-user-images\image-20240130222658083.png)

#### Language-Aware Lable Smoothing

传统的label smoothing技术最初作为一种正则化技术被引入，特别的，给定一个训练样本，该样本属于$b$-th classes，label smoothing替代one-hot label