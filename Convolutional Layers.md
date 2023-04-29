A convolutional layer is a fundamental building block of [[Convolutional Neural Networks|convolutional neural networks]] (CNNs), which are a type of deep learning model commonly used for image and video recognition tasks. A convolutional layer **applies a convolution operation** to the input data, which is typically an image or a feature map from a previous layer.

The convolution operation involves a *set of learnable filters*, also known as **kernels**, which **slide over the input data** and perform a **dot product between their weights and the local patches of the input**. The **output** of this operation is a **new feature map**, which captures local patterns or features in the input data that are relevant to the task at hand.

Each **filter** in a convolutional layer is **typically small in size**, such as 3x3 or 5x5, but can have **multiple channels to capture different aspects** of the input data. The layer can also have **multiple filters**, which allows it to **extract multiple features simultaneously**. The **output** of each filter is typically **passed through a nonlinear activation function**, such as ReLU, to introduce nonlinearity into the network and increase its expressive power.

In addition to the filters, a convolutional layer also has a **stride** parameter, which determines the *step size* of the filter as it slides over the input data. This parameter can be used to **control the spatial resolution of the output feature map**, with a smaller stride resulting in a larger feature map and a larger stride resulting in a smaller feature map.

Overall, convolutional layers are a powerful tool for extracting features from high-dimensional data such as images and video. They are widely used in state-of-the-art deep learning models for a variety of computer vision tasks, including object detection, semantic segmentation, and image classification.

---

- [How Do Convolutional Layers Work in Deep Learning Neural Networks?](https://machinelearningmastery.com/convolutional-layers-for-deep-learning-neural-networks/)