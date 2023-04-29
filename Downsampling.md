Downsampling in deep neural networks refers to the process of **reducing the spatial resolution of an image or feature map** by applying a **pooling operation** or **convolutional layer with a stride greater than 1**. This operation reduces the size of the input while also increasing the receptive field of the network, **allowing it to capture more global information** and **abstract features**.

In **semantic segmentation**, downsampling is a crucial step in the network architecture. The goal of semantic segmentation is to assign a class label to each pixel in an image. To accomplish this task, the network needs to be able to **capture both local and global features** of the input image. Local features refer to the visual details and texture of small image regions, while global features capture the larger context and scene layout.

To achieve this, semantic segmentation networks often **use an encoder-decoder architecture**, where the encoder downsamples the input image to extract high-level features and the decoder upsamples the features to generate a pixel-wise segmentation map. The downsampling is typically performed using [[Convolutional Layers]] with a stride of 2 or [[Pooling Layers]] with a stride of 2 or 3. This process is repeated multiple times to further reduce the spatial resolution of the features.

The downsampling operation also helps to reduce the computational complexity of the network by reducing the number of parameters and the size of the feature maps. This allows the network to be trained efficiently on large datasets, making it possible to learn complex visual patterns and produce accurate segmentations.

Overall, downsampling is an important technique in deep neural networks for semantic segmentation, enabling the network to capture both local and global features while reducing the computational complexity of the network.

---

# From ["Pooling Layers for Convolutional Neural Networks"](https://machinelearningmastery.com/pooling-layers-for-convolutional-neural-networks/)

[Convolutional layers](https://machinelearningmastery.com/convolutional-layers-for-deep-learning-neural-networks/) in a convolutional neural network summarize the presence of features in an input image.

A problem with the output feature maps is that they are sensitive to the location of the features in the input. One approach to address this sensitivity is to down sample the feature maps. This has the effect of making the resulting down sampled feature maps more robust to changes in the position of the feature in the image, referred to by the technical phrase “_local translation invariance_.”

Pooling layers provide an approach to down sampling feature maps by summarizing the presence of features in patches of the feature map. Two common pooling methods are average pooling and max pooling that summarize the average presence of a feature and the most activated presence of a feature respectively.