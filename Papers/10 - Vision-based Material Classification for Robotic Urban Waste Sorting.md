#paper

In contrast to the preceding, the current work focuses on much more complex multi-object images, solving a three-fold problem related to object identification (bounding box specification), localization (masking), and material type attribution (classification). This triplet makes our solution applicable in industrial conditions where several potentially overlapping recyclables shown in the very same image need to be identified, localized, and classified.

Key components deployed in the robot cabin are described in the following.
- Conveyor belt
- Waste feeder:
- Camera
- Robot
- Vacuum gripper

This article presents an integrated robotic system for recyclable sorting that is composed of two main parts: 
- a robotic manipulator for the physical separation of waste to different bins, depending on the material type
- a vision-based material detection and categorization module

The majority of robotic systems for recyclable sorting rely on a pick-and-place (PnP) process to select objects and physically transfer them to the appropriate material bin. Capitalizing on delta robots, we developed a composite system that performs fast recyclable sorting by adopting the PnP approach. Given that recyclables do not require fine manipulations, the gripping of objects is often implemented with the use of vacuum technology.

Generally speaking, there are two main technologies to generate a vacuum
- Venturi generators
- Blowers

CNNs trained with deep learning algorithms have been widely applied in demanding computer vision applications. Interestingly, CNNs have also been used to perform waste classification, although they were applied to a relatively small data set with single-item images and without any type of occlusion. In contrast, the present work targets "instance segmentation" to accomplish multipleobject identification and labeling. To this end, the well-known [[Mask Regional CNN|R-CNN]] open source network was employed since it has been particularly successful at similar tasks. Mask R-CNN provides a scalable means for categorizing recyclables into a high number of classes. However, this cannot be done incrementally. Any time a class is added, a full retraining of the model is assumed, according to the specification of the new data set.



# Data collection and processing procedure
According to the PDF file, there are only two open source datasets available for public use, namely TrashNet and Taco. Both cover waste classification for outdoor cases rather than for demanding industrial setups, where objects transferred on conveyor belts can be strongly deformed, dirty, and piled on top of one another. The present work's approach to develop the data set differs from the techniques in the works mentioned previously. You can find this information on page 6 of the PDF file.
The authors of the article did not use the already existing datasets, TrashNet and Taco, because both cover waste classification for outdoor cases rather than for demanding industrial setups, where objects transferred on conveyor belts can be strongly deformed, dirty, and piled on top of one another. The present work's approach to develop the data set differs from the techniques in the works mentioned previously. Therefore, they developed their own dataset using a different approach to better suit their needs. You can find this information on page 6 of the PDF file.

The authors of the article followed a semiautomated procedure to develop the data set for training Mask R-CNN. They collected 400 images for each of the material types considered in the study, namely, aluminum, paper and cardboard, polyethylene terephthalate (PET) bottles, and nylon. Then they followed a manual procedure to collect red-green-blue images. All images had the same size of 800 x 800 pixels and depicted a single recyclable against a black background. It is noted that to have a representative data set of recyclables images, they also considered deformed and dirty materials. You can find this information on page 7 of the PDF file.

1. First, they identified masks describing regions of the identified objects. They called this the Basic data set. 
2. Then, they applied step two to gather 2,000 images of each material type, applying randomly three geometric transformations (translation, rotation, and scaling) to the image and the mask. They called this the Synthetic Single data set. 
3. Finally, following the third processing step, they developed a data set that artificially described complex cases of multiple and overlapping objects, and changed the background. This was the Synthetic Complex data set. In particular, they developed 16,000 images for the training set and 5,600 for the validation set.

# Training process
- We used a public [[Mask Regional CNN|Mask R-CNN]] implementation that was trained on the data sets in the “Vision-Based Material Categorization” section, using 30% of the images for model learning validation and the remaining 70% for model training. 
- To increase the image processing frame rate, we incorporated [[ResNet-50]] for feature extraction across entire photos.
- Moreover, a [[Region Proposal Network|RPN]] was employed to scan images in a sliding window manner to identify areas that contained objects. The RPN employed a set of boxes, called anchors, with predefined image locations and scales (there were five scales and three aspect ratios in our implementation) to figure out the size and location of an object on the feature map. Specifying the correct anchor span is critical for obtaining successful classification results. To estimate recyclable objects’ bounding boxes, masks, and material types, dedicated subnetworks known as heads work on identified regions of interest to shape the final output. After extensive experimentation with the validation set, we identified the Mask R-CNN training parameters that worked effectively.
- Before using Mask R-CNN for recyclable identification and classification, a learning process was applied to improve the applicability of the deep neural network to the problem. To bootstrap the learning process, a network pretrained on the [[Common Objects in Context data set]] was used. This approach speeds up learning and ensures the minimum quality of the results in a reasonable training time. Following this, only the Mask R-CNN heads are typically trained for a problem, keeping the general object feature extraction mechanism unchanged. To adapt Mask R-CNN to the classification task, we developed and tested four customized implementations: Net 1, Net 2, Net 3, and Net 4. They were trained on a high-performance computing infrastructure with TensorFlow GPU support.

Mask R-CNN training is based on a complex loss function that is calculated as the weighted **sum of different partial losses** at every training state. The partial losses considered in the current implementation are described in the following.
- **mrcnn_bbox_loss** corresponds to the success at *localizing the bounding boxes* of objects that belong to a given class. This loss metric is increased when the object categorization is correct but the bounding box localization is not precise.
- **mrcnn_class_loss** describes the loss due to the improper categorization of an object identified by the RPN. This provides an indication of how well Mask R-CNN *recognizes each class of an identified object*. This metric is increased when an object is detected in an image but misclassified.
- **mrcnn_mask_loss** is the metric that summarizes the success at implementing masks across a processed image that correspond to the *pixel area covered by the identified objects*. The training data set includes masks for all objects of interest in input images, thus providing the ground truth for the evaluation of masks predicted by Mask R-CNN.
- **rpn_bbox_loss** corresponds to the localization accuracy of the RPN, in other words, *how well the RPN identifies individual objects*. High values indicate miscalculations when an object is detected but the bounding box needs to be corrected.
- **rpn_class_loss** is an RPN performance metric that is *increased when an object is undetected* at the final output and decreased when the RPN successfully identifies an item.

To assess the performance of each trained network, we used a typical evaluation procedure that considered three metrics, namely, average precision (AP), average recall (AR), and the F1 score (F1)

# Implementations
- Net 1: trained on the Basic data set
- Net 2: trained on the Synthetic Single data set
- Net 3: trained on the Synthetic Complex data set
- Net 4: trained on the Synthetic Complex data set, after applying translation, rotation, and scale (in the ranges (- 0.2, 0.2), (- 45, 45), and (0.8, 1.2), respectively). Additionally, the training process was extended by using 100 learning steps in each epoch.