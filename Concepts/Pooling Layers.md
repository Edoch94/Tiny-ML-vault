Pooling layers are a type of layer commonly used in [[Convolutional Neural Networks|convolutional neural networks]] (CNNs) for **down-sampling the spatial resolution of feature maps**. The main goal of pooling is to **reduce the spatial size** of the feature maps while retaining the most important information.

A pooling layer operates by dividing the input feature map into non-overlapping regions, or by using a sliding window approach, and **applying a pooling operation to each region**. The most common types of pooling operations are **max pooling** and **average pooling**. In max pooling, the maximum value of each region is selected as the output, while in average pooling, the average value is taken.

The pooling operation effectively **reduces the spatial size of the feature map by a factor** determined by the size of the pooling region and the stride of the pooling operation. For example, if the pooling region has size 2x2 and the stride is 2, then the spatial resolution of the output feature map will be reduced by a factor of two in both dimensions.

Pooling layers can be used for several *reasons*:
1. They **reduce the spatial resolution** of the feature maps, which can help 
	- to *reduce the computational cost* of the network 
	- and *avoid overfitting*.
2. They can help to **increase the receptive field** of the network
	- allowing it to capture *more global features and patterns*.
3. They can **introduce a degree of translation invariance** to the network by 
	- selecting the most relevant features in each region.

Pooling layers are typically used in conjunction with convolutional layers in CNNs. The *[[Convolutional Layers|convolutional layers]]* are responsible for *extracting features* from the input data, while the *pooling layers* are responsible for *down-sampling* the features and increasing the robustness of the network. By stacking multiple convolutional and pooling layers, the network can learn hierarchical representations of the input data, starting from low-level features and gradually building up to high-level features that are more relevant to the task at hand.