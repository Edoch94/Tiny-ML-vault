---
tags:
  - paper
---
# Abstract
Semantic segmentation requires both rich spatial information and sizeable receptive field. However, modern approaches usually compromise spatial resolution to achieve real-time inference speed, which leads to poor performance. In this paper, we address this dilemma with a novel [[Bilateral Segmentation Network|BiSeNet]]. We first design a Spatial Path with a small stride to preserve the spatial information and generate high-resolution features. Meanwhile, a Context Path with a fast downsampling strategy is employed to obtain sufficient receptive field. On top of the two paths, we introduce a new **Feature Fusion Module** to combine features efficiently. The proposed architecture makes a right balance between the speed and segmentation performance on Cityscapes, CamVid, and COCO-Stuff datasets. Specifically, for a 2048×1024 input, we achieve 68.4% Mean IOU on the Cityscapes test dataset with speed of 105 FPS on one NVIDIA Titan XP card, which is significantly faster than the existing methods with comparable performance.  

---

Three approaches to accelerate the model. 
1. try to **restrict the input size** to reduce the computation complexity by cropping or resizing. Though the method is simple and effective, the loss of spatial details **corrupts the prediction** especially around boundaries, leading to the **accuracy decrease** on both metrics and visualization. 
2. Instead of resizing the input image, some works **prune the channels** of the network to boost the inference speed, especially in the early stages of the base model. However, it **weakens the spatial capacity**. 
3. For the last case, [[04 - ENet A Deep Neural Network Architecture for Real-Time Semantic Segmentation|ENet]] proposes to **drop the last stage of the model** in pursuit of an extremely tight framework. Nevertheless, the drawback of this method is obvious: since the ENet *abandons the [[Downsampling|downsampling]] operations in the last stage*, the receptive field of the model is not enough to cover large objects, resulting in a **poor discriminative ability**. 

Overall, all of the above methods compromise the accuracy to speed, which is inferior in practice.

To remedy the loss of spatial details mentioned above, researchers widely utilize the [[U-Net architecture|U-shape structure]]. By fusing the hierarchical features of the backbone network, the U-shape structure gradually <u>increases the spatial resolution and fills some missing details</u>. However, this technique has **two weaknesses**.
1. The complete U-shape structure can <u>reduce the speed of the model</u> due to the introduction of extra computation on high-resolution feature maps. 
2. More importantly, most <u>spatial information lost</u> in the pruning or cropping <u>cannot be easily recovered</u> by involving the shallow layers. 
In other words, the U-shape technique is better to regard as a relief, rather than an essential solution.

Based on the above observation, we propose the Bilateral Segmentation Network (BiSeNet) with two parts: Spatial Path (SP) and Context Path (CP). As their names imply, the two components are devised to confront with the loss of spatial information and shrinkage of receptive field respectively. The design philosophy of the two paths is clear. 
1. For Spatial Path, we stack only three [[Convolutional Layers|convolution layers]] to obtain the 1/8 feature map, which retains affluent spatial details. 
2. In respect of Context Path, we append a [[Pooling Layers|global average pooling layer]] on the tail of [[Xception]], where the receptive field is the maximum of the backbone network. 

![[Pasted image 20230429190020.png]]

In pursuit of better accuracy without loss of speed, we also 
- research the fusion of two paths, proposing Feature Fusion Module (FFM)
- and refinement of final prediction, proposing Attention Refinement Module (ARM) 

As our following experiments show, these two extra components can further improve the overall semantic segmentation accuracy on both Cityscapes, CamVid, and COCO-Stuff benchmarks.

## Main contributions
Our main contributions are summarized as follows:
- We propose a novel approach to decouple the function of spatial information preservation and receptive field offering into two paths. Specifically, we propose a Bilateral Segmentation Network (BiSeNet) with a Spatial Path (SP) and a Context Path (CP).
- We design two specific modules, Feature Fusion Module (FFM) and Attention Refinement Module (ARM), to further improve the accuracy with acceptable cost. 
- We achieve impressive results on the benchmarks of Cityscapes, CamVid, and COCO-Stuff. More specifically, we obtain the results of 68.4% on the Cityscapes test dataset with the speed of 105 FPS.

# Architecture
![[Pasted image 20230429191935.png]]


> [!note] Meaning of "While the Spatial Path encodes affluent spatial information, the Context Path is designed to provide sufficient receptive field." 
>The Spatial Path and Context Path are two components of a neural network designed to perform semantic segmentation. The Spatial Path is responsible for capturing spatial information, while the Context Path is designed to provide a large receptive field to capture global contextual information.

## Spatial Path
The Spatial Path is responsible for encoding spatial information, meaning it focuses on capturing details about the local regions of the image. This path typically consists of several convolutional layers and pooling layers, which downsample the input image while preserving spatial information. The sentence states that the Spatial Path encodes "affluent" spatial information, suggesting that this path is designed to capture as much spatial detail as possible.

We propose a Spatial Path to preserve the spatial size of the original input image and encode affluent spatial information. The Spatial Path contains three layers. Each layer includes a convolution with stride = 2, followed by batch normalization and ReLU. Therefore, this path extracts the output feature maps that is 1/8 of the original image. It encodes rich spatial information due to the large spatial size of feature maps. Figure 2(a) presents the details of the structure.

## Context Path
The Context Path, on the other hand, is designed to provide a "sufficient receptive field". A receptive field refers to the area in the input image that influences the output of a given neuron in the network. In semantic segmentation, it is important to have a large receptive field in order to capture global contextual information, such as the overall structure and layout of the image. The Context Path typically consists of several convolutional layers with large kernel sizes, which increase the receptive field of the network. The sentence suggests that the Context Path is specifically designed to provide a receptive field that is large enough to capture the necessary contextual information for the segmentation task.

In this work, the lightweight model, like [[Xception]], can downsample the feature map fast to obtain large receptive field, which encodes high level semantic context information. Then we add a global average pooling on the tail of the lightweight model, which can provide the maximum receptive field with global context information. Finally, we combine the up-sampled output feature of global pooling and the features of the lightweight model. In the lightweight model, we deploy U-shape structure to fuse the features of the last two stages, which is an incomplete U-shape style. 

## Network
With the Spatial Path and the Context Path, we propose BiSeNet for real-time semantic segmentation.

We use the pre-trained Xception model as the backbone of the Context Path and three convolution layers with stride as the Spatial Path. And then we fuse the output features of these two paths to make the final prediction. It can achieve real-time performance and high accuracy at the same time. First, we focus on the practical computation aspect. 
- Although the Spatial Path has large spatial size, it only has three convolution layers. Therefore, it is **not computation intensive**. 
- As for the Context Path, we use a **lightweight model** to down-sample rapidly. Furthermore, these two paths compute concurrently, which considerably increase the efficiency. 
Second, we discuss the accuracy aspect of this network. In our paper, the Spatial Path encodes rich spatial information, while the Context Path provides large receptive field. They are complementary to each other for higher performance.

- **Attention refinement module**: In the Context Path, we propose a specific Attention Refinement Module (ARM) to refine the features of each stage. ARM employs [[Pooling Layers|global average pooling]] to capture global context and computes an [[attention vector]] to guide the feature learning. This design can refine the output feature of each stage in the Context Path. It integrates the global context information easily without any up-sampling operation. Therefore, it demands negligible computation cost.

- **Feature fusion module**: The features of the <u>two paths are different in level of feature representation</u>. Therefore, we **can not simply sum up these features**. The spatial information captured by the Spatial Path encodes mostly rich detail information. Moreover, the output feature of the Context Path mainly encodes context information. In other words, the <u>output feature of Spatial Path is low level</u>, while the <u>output feature of Context Path is high level</u>. Therefore, we propose a specific **Feature Fusion Module to fuse these features**. Given the different level of the features, 
	1. we first <u>concatenate the output features</u> of Spatial Path and Context Path. 
	2. And then we utilize the <u>batch normalization</u> to balance the scales of the features. 
	3. Next, we <u>pool the concatenated feature</u> to a feature vector and compute a weight vector, like [[SENet]]. This weight vector can re-weight the features, which amounts to feature selection and combination. 

- **Loss function**: In this paper, we also utilize the auxiliary loss function to supervise the training of our proposed method. We use the principal loss function to supervise the output of the whole BiSeNet. Moreover, we add two specific auxiliary loss functions to supervise the output of the Context Path, like deep supervision. All the loss functions are Softmax loss, as Equation 1 shows. Furthermore, we use the parameter $\alpha$ to balance the weight of the principal loss and auxiliary loss, as Equation 2 presents. The $\alpha$ in our paper is equal to 1. The joint loss makes optimizer more comfortable to optimize the model.
$$
\text{loss}=\frac{1}{N}\sum_i{L_i}=\frac{1}{N}\sum_i{-\log{(\frac{e^{p_i}}{\sum_i{e^{p_i}}})}}
$$
where $p$ is the output prediction of the network.
$$
L(X;W)=l_p(X;W)+\alpha\sum_{i=2}^K{l_i(X;W)}
$$
where $l_p$ is the principal loss of the concatenated output. $X_i$ is the output feature from stage $i$ of Xception model. $l_i$ is the auxiliary loss for stage $i$. The $K$ is equal to 3 in our paper. The $L$ is the joint loss function. Here, we only use the auxiliary loss in the training phase.
