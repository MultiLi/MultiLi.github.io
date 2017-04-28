---
layout: post
title: R-CNN Reading Notes
---

### Components
+ **Region Proposal**:
  + Selective search for region proposals
  + Reduce the number of proposals with non-maximum suppression
  + Warped each proposal to fit the ConvNet
  + Hard coded, not learnable
+ **ConvNet Feature Extractor**:
  + Pre-trained on ImageNet
  + Trained with loosely-filtered b-box area
  + Aim at extracting feature from raw pixels (4096-d feature vector)
+ **Linear SVM Classifier**:
  + Class-specific
  + Take the feature extracted by the ConvNet as input
  + Aim at categorizing the feature of each region proposal to corresponding class
  + Trained with high IOU b-box area
+ **B-box Regressor**:
  + Class-specific
  + Take the feature, corresponding b-box coordinates, extracted by ConvNet as input
  + Coordinates in log scale
  + Trained to optimize the selective search result and increase the IOU with ground truth

### Pipeline
+ **Train**
  + Pretrain the ConvNet on ImageNet (with a redundant softmax layer removed later)
  + Region proposal and preprocessing (warping to fit the ConvNet)
  + Fine-tune the ConvNet on training set (with a redundant softmax layer removed later)
  + Train the Linear SVM on training set (class-specific, on top of the ConvNet)
  + Train the b-box regressor based on the features and the ground truth (based on the region proposal and feature extractor, class-specific)
+ **Test**
  + Region proposal and preprocessing (Selective search)
  + Feature Extraction (ConvNet, dependent on region proposal)
  + Classification (Linear SVM, dependent on all 2 above)
  + B-box refinement (B-box regressor, dependent on all 3 above)
