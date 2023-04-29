#paper

# Abstract 
The low-level details and high-level semantics are both essential to the semantic segmentation task. However, to speed up the model inference, <u>current approaches almost always sacrifice the low-level details</u>, which leads to a considerable accuracy decrease. We propose to **treat these spatial details** and **categorical semantics separately** to achieve high accuracy and high efficiency for real-time semantic segmentation. To this end, we propose an efficient and effective architecture with a good trade-off between speed and accuracy, termed Bilateral Segmentation Network (BiSeNet V2). This architecture involves: 
1. a **Detail Branch**, with <u>wide channels</u> and <u>shallow layers</u> to **capture low-level details** and generate high-resolution feature representation; 
2. a **Semantic Branch**, with <u>narrow channels</u> and <u>deep layers</u> to obtain **high-level semantic context**. The Semantic Branch is <u>lightweight</u> due to reducing the channel capacity and a fast-downsampling strategy. 

Furthermore, we design a **Guided Aggregation Layer** to enhance mutual connections and fuse both types of feature representation. Besides, a **booster training strategy** is designed to improve the segmentation performance without any extra inference cost. 

Extensive quantitative and qualitative evaluations demonstrate that the proposed architecture performs favourably against a few state-of-the-art real-time semantic segmentation approaches. Specifically, for a 2,048×1,024 input, we achieve 72.6% Mean IoU on the Cityscapes test set with a speed of 156 FPS on one NVIDIA GeForce GTX 1080 Ti card, which is significantly faster than existing methods, yet we achieve better segmentation accuracy. Code and trained models will be made publicly available.

---

# Introduction
There are two main architectures as the backbone networks: 
- Dilation Backbone, removing the downsampling operations and up-sampling the corresponding filter kernels to maintain high-resolution feature representation
- Encoder-Decoder Backbone, with top-down and skip connections to recover the high-resolution feature representation in the decoder part
However, both architectures are designed for general semantic segmentation tasks with less care about the inference speed and computational cost. In the dilation backbone, the dilation convolution is time-consuming and removing down-sampling operation brings heavy computation complexity and memory footprint. Numerous connections in the encoder-decoder architecture are less friendly to the memory access cost. However, the <u>real-time semantic segmentation applications demand for an efficient inference speed</u>.

Facing this demand, based on both backbone networks, existing methods mainly employ two appraches to accelerate the model: 
- **Input Restricting**. Smaller input resolution results in less computation cost with the same network architecture. To achieve real-time inference speed, many algorithms attempt to restrict the input size to reduce the whole computation complexity; 
- **Channel Pruning**. It is a straight-forward acceleration method, especially pruning channels in early stages to boost inference speed
Although both manners can improve the inference speed to some extent, <u>they sacrifice the low-level details and spatial capacity leading to a dramatic accuracy decrease</u>. Therefore, to achieve high efficiency and high accuracy simultaneously, it is challenging and of great importance to exploit a specific architecture for the real-time semantic segmentation task.

To this end, we propose a two-pathway architecture, termed [[Bilateral Segmentation Network|Bilateral Segmentation Network (BiSeNet V2)]], for real-time semantic segmentation. One pathway is designed to <u>capture the spatial details</u> with wide channels and shallow layers, called **Detail Branch**. In contrast, the other pathway is introduced to <u>extract the categorical semantics</u> with narrow channels and deep layers, called **Semantic Branch**. The Semantic Branch simply requires a large receptive field to capture semantic context, while the detail information can be supplied by the Detail Branch. Therefore, the Semantic Branch can be made very lightweight with fewer channels and a fast-downsampling strategy. Both types of feature representation are merged to construct a stronger and more comprehensive feature representation. This conceptual design leads to an efficient and effective architecture for real-time semantic segmentation.

Specifically, in this study, we design a **Guided Aggregation Layer** <u>to merge both types of features effectively</u>. To <u>further improve the performance</u> without increasing the inference complexity, we present a **booster training strategy** with a series of auxiliary prediction heads, which can be discarded in the inference phase. Extensive quantitative and qualitative evaluations demonstrate that the proposed architecture performs favourably against state-of-the-art real-time semantic segmentation approaches.

The main contributions are summarized as follows:
- We propose an efficient and effective **two-pathway architecture**, termed Bilateral Segmentation Network, for real-time semantic segmentation, which treats the spatial details and categorical semantics separately.
- For the Semantic Branch, we design a **new lightweight network** based on <u>depth-wise convolutions</u> to enhance the receptive field and capture rich contextual information.
- A **booster training strategy** is introduced to further improve the segmentation performance without increasing the inference cost.
- Our architecture achieves impressive results on different benchmarks of Cityscapes (Cordts et al., 2016), CamVid (Brostow et al., 2008a), and COCO-Stuff (Caesar et al., 2018). More specifically, we obtain the results of 72.6% mean IoU on the Cityscapes test set with the speed of 156 FPS on one NVIDIA GeForce GTX 1080Ti card.

![[Pasted image 20230429202624.png]]
Illustration of different backbone architectures.
- (a) is the dilation backbone network, which removes the downsampling operations and upsampling the corresponding convolution filters. It has heavy computation complexity and memory footprint.
- (b) is the encoder-decoder backbone network, which adds extra top-down and lateral connections to recover the high-resolution feature map. These connections in the network are less friendly to the memory access cost. 
- To achieve high accuracy and high efficiency simultaneously, we design the (c) Bilateral Segmentation backbone network. This architecture has two pathways, Detail Branch for spatial details and Semantic Branch for categorical semantics. The detail branch has wide channels and shallow layers,while the semantic branch has narrow channels and deep layers, which can be made very lightweight by the factor ($\lambda$, e.g., 1/4).

A preliminary version of this work was published: [[01 - BiSeNet Bilateral Segmentation Network for Real-time Semantic Segmentation]]
We have extended our conference version as follows. 
- We <u>simplify the original structure</u> to present an efficient and effective architecture for real-time semantic segmentation. We <u>remove the time consuming cross-layer connections</u> in the original version to obtain a more clear and simpler architecture.
- We re-design the overall architecture with <u>more compact network structures</u> and <u>well-designed components</u>. Specifically, we **deepen the Detail Path** to encode more details. We design **light-weight components** based on the depth-wise convolutions for the **Semantic Path**. Meanwhile, we propose an **efficient aggregation layer** to enhance the mutual connections between both paths.
- We conduct comprehensive ablative experiments to elaborate on the effectiveness and efficiency of the proposed method. 
- We have significantly improved the accuracy and speed of the method in our previous work, i.e., for a 2048 × 1024 input, achieving 72.6% Mean IoU on the Cityscapes test set with a speed of 156 FPS on one NVIDIA GeForce GTX 1080Ti card.

# Architecture

![[Pasted image 20230429204034.png]]
> [!Abstract] Overview of the Bilateral Segmentation Network. 
> There are mainly three components: 
> - two-pathway backbone in the purple dashed box, 
> - the aggregation layer in the orange dashed box, 
> - and the booster part in the yellow dashed box. 
> 
> The two-pathway backbone has a 
> - Detail Branch (the blue cubes) 
> - and a Semantic Branch (the green cubes). 
> 
> The three stages in **Detail Branch** have C1 , C2 , C3 channels respectively. The channels of corresponding stages in **Semantic Branch** can be made lightweight by the factor λ(λ < 1). The last stage of the Semantic Branch is the output of the **Context Embedding Block**. Meanwhile, numbers in the cubes are the feature map size ratios to the resolution of the input. 
> 
> In the Aggregation Layer part, we adopt the bilateral aggregation layer. Down indicates the **downsampling** operation, Up represents the **upsampling** operation, ϕ is the Sigmoid function, N and means element-wise product. Besides, in the booster part, we design some auxiliary segmentation heads to improve the segmentation performance without any extra inference cost.

## Core concepts

### Detail Branch
The Detail Branch is responsible for the spatial details, which is low-level information. The key concept of the Detail Branch is to use **wide channels** and **shallow layers** for the spatial details. Besides, the feature representation in this branch has a large spatial size and wide channels. Therefore, it is better not to adopt the residual connections, which increases the memory access cost and reduce the speed.

### Semantic Branch
The Semantic Branch is designed to **capture high-level semantics**. This branch has low channel capacity, while the spatial details can be provided by the Detail Branch. In contrast, in our experiments, the Semantic Branch has a ratio of $\lambda$ ($\lambda<1$) channels of the Detail Branch, which makes this branch lightweight. Actually, the Semantic Branch <u>can be any lightweight convolutional model</u>. Meanwhile, the Semantic Branch adopts the **fast downsampling strategy** to promote the level of the feature representation and enlarge the receptive field quickly. High-level semantics require large receptive field, therefore, the Semantic Branch employs the **global average pooling** to embed the global contextual response.

### Aggregation Layer
The feature representation of the Detail Branch and the Semantic Branch is complementary, one of which is unaware of the information of the other one. Thus, an Aggregation Layer is designed to **merge both types of feature representation**. Due to the fast-downsampling strategy, the spatial dimensions of the **Semantic Branch's output are smaller** than the Detail Branch. We need to <u>upsample</u> the output feature map of the Semantic Branch <u>to match the output of the Detail Branch</u>. There are a few manners to fuse information, e.g., simple summation, concatenation and some well-designed operations. We have experimented different fusion methods with consideration of accuracy and efficiency. At last, we **adopt the bidirectional aggregation method**.

## Network

### Detail Branch
The instantiation of the Detail Branch in Table 1 contains three stages, each layer of which is a convolution layer followed by batch normalization and activation function. The first layer of each stage has a stride s = 2, while the other layers in the same stage have the same number of filters and output feature map size. Therefore, this branch extracts the output feature maps that are 1/8 of the original input.

### Semantic Branch
In consideration of the large receptive field and efficient computation simultaneously, we design the Semantic Branch, inspired by the philosophy of the lightweight recognition model, e.g., Xception, MobileNet, ShuffleNet. Some of the key features of the Semantic Branch are as follows.

#### Stem Block
Stem Block adopts a fast-downsampling strategy. This block has two branches with different manners to downsample the feature representation. Then both feature response of two branches is concatenated as the output.

It uses two different downsampling manners to shrink the feature representation. And then the output feature of both branches are concatenated as the output.

#### Context Embedding Block
This block uses the global average pooling and residual connection to embed the global contextual information efficiently. 
We design a Context Embedding Block with the global average pooling to embed the global contextual information.

#### Gather-and-Expansion Layer
The Gather-and-Expansion Layer consists of:
- a 3×3 convolution to efficiently aggregate feature responses and expand to a higher-dimensional space; 
- a 3 × 3 depth-wise convolution performed independently over each individual output channel of the expansion layer; 
- a 1×1 convolution as the projection layer to project the output of depth-wise convolution into a low channel capacity space. When stide = 2, we adopt two 3 × 3 depth-wise convolution, which further enlarges the receptive field, and one 3 × 3 separable convolution as the shortcut.

### Bilateral Guided Aggregation
The Detail Branch is for the low-level, while the Semantic Branch is for the high-level. Therefore, simple combination ignores the diversity of both types of information, leading to worse performance and hard optimization. Based on the observation, we propose the Bilateral Guided Aggregation Layer to <u>fuse the complementary information from both branches</u>. This layer employs the contextual information of Semantic Branch to guide the feature response of Detail Branch. With different scale guidance, we can capture different scale feature representation, which inherently encodes the multi-scale information. Meanwhile, this guidance manner enables efficient communication between both branches compared to the simple combination.

### Booster Training Strategy
To further improve the segmentation accuracy, we propose a booster training strategy. As the name implies, it is similar to the rocket booster: it can enhance the feature representation in the training phase and can be discarded in the inference phase. Therefore, it increases little computation complexity in the inference phase.