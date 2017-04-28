---
layout: post
title: Faster RCNN Reading Notes
---

The search for region proposals is not learnable, and running on CPU, which is computationally ineffective. To improve the performance, and to make the whole pipeline end-to-end, taking only images as input,  propose a new convolutional structure, **R**egion **P**roposal **N**etwork, which together with Fast RCNN constitute the Faster RCNN.  The weights of the ConvNet are shared between RPN and Fast RCNN.

### Region Proposal Network

+ Fully convolutional. Taking whole images as input, and outputting region proposals in different scale and aspect ratio.
+ For the last few layers (grouped as a mini-netowork) that take the output of last shared convolutional layer (so called feature map) as input:
  + **Local feature**: Each sliding window of the input is processed and mapped  to low dimensional feature vector, which is inferred informative in determining both the existance and location of b-box and to which class it belongs. The intuition behind is, even the size of each sliding window is pretty small (3 $\times$ 3 in the paper), due to the deep structure of the ConvNet, it has a broad receptive field (almost cover the whole input image) that potential to carry corresponding information.
  + **Anchor**: More specifically, each sliding window of the input is thought to represent  $k$ b-boxes centered at the center location of the window, where $k$ corresponds to proposals of different scale and aspect ratio. In the paper, 3 scales and 3 aspect ratios are considered, so $k=9$.
  + **Multitask Loss** :Therefore, following the feature vector are 2$k$ classification scores, 2 for each anchor (2-class softmax, which can alternatively be $k$ logistic regression scores) as "objectness", and $4k$ regression results (2 for the center coordinates and 2 for width and height).  The loss function is simply the sum of the entropy loss of classification task and smooth L1 loss of regression task, weighted by a balancing parameter $\lambda$.
  + **Translation-invariant**: Since for the b-box of a pair of fixed scale and aspect ratio in different location of the image, same weights are applied to do the classification and regression tasks, and the fact that RPN is an implementation of FCN, it obtains the translation-invariant property.
+ **Multi-scale Detection**:
  + Multi-scale input (image pyramid)
  + Multi-scale sliding window
  + Multi-scale anchor (what faster RCNN used)
+ **Positive-negative B-boxes**:
  + **Positive**: Has highest IoU overlap with the ground-truth (among all $k$ b-boxes centered at the same point), or has an IoU overlap higher than 0.7
  + **Negative**: Has IoU ratio lower than 0.3 for all groung-truths.
  + Others do not count; thus wouldn't influence the training.

### Pipeline

+ **Train**:
  + Sampling the positive and negative anchors that have a ratio of up to 1:1
  + New layers (mini-network) are randomly initialized while others are pretrained on ImageNet
  + **Alternating Training**: Training RPN and Fast RCNN alternatively (Finally utilized Scheme).
    1. RPN training
    2. Fast RCNN training based on trained RPN (no share weights)
    3. RPN mini-network fine-tuning (ConvNet weights from Fast RCNN and fixed)
    4. Fast RCNN fine-tuning (Only unique layers)
  + **Approximate joint training**: Combine RPN and Fast RCNN together (feed the output of RPN to Fast RCNN), and training the whole structure end-to-end in a single phase. The disadvantage is, since the region proposals (anchors) are not really accurate, the output of Fast RCNN is biased, but the corresponding loss is not included since the region proposal part is not learnable in Fast RCNN.
  + **Non-approximate joint training**: Solve the ignored gradient w.r.t the coordinates of the region proposals by an "ROI warping" layer, which backpropagates the loss to the coordinates.
  + Cross-boundary anchors are ignored, no contribution.
+ **Test**: Simultaneously compute the region proposals and feature map, and as in Fast RCNN, get a fixed length representation of each ROI by ROI pooling layer, and do classification.
