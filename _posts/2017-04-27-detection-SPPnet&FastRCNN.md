---
layout: post
title: 2.SPPnet and Fast RCNN Reading Notes
---

### Novel Idea of SPPnet
Replace the regular pooling layer, in which the size of the kernel is fixed and the output depends on the input, with a **S**patial **P**aramid **P**ooling (SPP) layer, in which, on the contrary, the size of the kernel depends on the input and the output is fixed. Consequently, the input size of the final fully-connected layer can be determined in advance, thus the network is empowered to deal with arbitrary-size input.
### Spatial Paramid Pooling Layer
+ Fixed output size
+ Multi-scale pooling
+ Size of Receptive field proportional to the input

### General Improvements of Fast RCNN
+ **Arbitraty-size input**:
  Inspired by SPPnet, replace the last pooling layer of ConvNet with an ROIPooling layer (a special case of SPP layer in which there is only 1 scale) to avoid the warping phase.
+ **Single pass through ConvNet**:
  + In RCNN, each obtained region proposal is fed into the ConvNet separately.
  + In Fast RCNN, the whole image passes the ConvNet only once, and the region proposals are directly applied to the shared feature map. And then the pooling result of each ROI is fed to the following fully-connected layers.
+ **End-to-end Training**:
  + Do the classification work with the softmax layer on top of ConvNet rather than training an extra Linear SVM classifier.
  + Put the b-box regressor following the ROIPooling layer, so that it can be trained together with the rest parts instead of as a single phase.
  + The ConvNet is no longer used solely as a feature extractor. Together with the 2 sibling fully-connected parts on top (softmax classification and b-box regressor), it can produce the prediction all at once (classication of each input region proposal and the b-box refinement).
+ **Acceleration**:
  + Single pass through ConvNet (discussed in detail above).
  + SVD the fully-connected layer \\(W \approx U \sum_t V^T\\) , so that the number of parameter is reduced from $u\times v$ to $t(u+v)$.
  + End-to-end (discussed in detail above), no extra storage needed to keep the extracted features.
  + To speed up the backpropagation, introducing a hierarchical sampling strategy, in which N images are firstly sampled and then $\frac{R}{N}$ ROIs in each image (R, the number of ROIs, here corresponds to the batch size in machine learning terminology).

### Pipeline
+ **Train**
  + For each image, conducting selective search and series of post-processing and filter methods to get some region proposals.
  + The Fast RCNN takes as input images and associated region proposals, where the region proposals are directly applied to the feature map, and is supervised by the class label and actual location of each region proposal.
  + Cross entropy loss for the classification task, and L1 loss for regression.
+ **Test**
  + Fast RCNN directly outputs the prediction of classication and the b-box location of each image at once.
  + NMS used to refine the prediction
