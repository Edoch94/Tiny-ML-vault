A fully convolutional network (FCN) is a type of neural network architecture designed for **tasks** that **require pixel-level predictions**, such as semantic segmentation, image deblurring, and image super-resolution. Unlike traditional neural networks that consist of fully connected layers, **FCNs consist entirely of [[Convolutional Layers|convolutional layers]]**, which makes them better suited for handling input images of varying sizes.

In an FCN, the input image is passed through a series of convolutional layers, **which extract features from the image at multiple scales**. The output of these convolutional layers is then passed through a series of **upsampling layers**, which increase the spatial resolution of the feature map. Finally, the upscaled feature map is passed through a convolutional layer that produces the desired output.

One key aspect of FCNs is that they are <u>designed to preserve the spatial information of the input image</u> throughout the network. This is achieved by using upsampling layers that increase the spatial resolution of the feature map, rather than using pooling layers that reduce the resolution. Additionally, skip connections can be used to connect the output of a layer to the input of a later layer, which allows the network to preserve spatial information across multiple scales.

The main advantage of using FCNs over traditional neural networks for pixel-level prediction tasks is that **they can handle input images of any size**, as long as the size is a multiple of the stride of the convolutional layers. This makes them particularly <u>useful for tasks that require processing of high-resolution images or videos</u>.

FCNs have been used in a wide range of applications, including object detection, semantic segmentation, image restoration, and generative modeling. They have achieved state-of-the-art performance in many of these tasks and are widely used in computer vision research and industry.