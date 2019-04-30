# End-to-end Lane Detection

This repo contains the implementation of our paper [End-to-end Lane Detection through Differentiable Least-Squares Fitting](https://arxiv.org/abs/1902.00293v1) by Bert De Brabandere\*, Wouter Van Gansbeke\*, Davy Neven, Marc Proesmans and Luc Van Gool.

## Setup and design

This repository compares two methods to achieve higher accuracy for lane detection applications. The former is the conventional segmentation approach and the latter will tackle this problem in an end-to-end manner. The segmentation approach depends on the cross-entropy loss in order to learn the road markings by attention. However this approach is not necessarily the most accurate. Since the final line coordinates are desired, a complete end-to-end method should achieve better results.

This end-to-end architecture consist of two parts. The first part is an off the shelf network to predict weight maps. These weights are applied to a mesh grid which are used for the last step. The last step can be considered as the final layer of the network which solves a weighted system of equations to calculate the final curve parameters of the road markings. The amount of weight maps depends on the amount of lane lines the network ought to detect. This means that a direct regression can be performed to the desired curve coordinates. After all, backpropagation through the whole architecture is now possible.

Finally we show results for egolane detection. The implementation proofs that this direct optimization is indeed possible and can achieve better results than the conventional segmentation approach. Moreover, this module can be applied to a broad range of tasks since the first stage of the architecture can be chosen freely. The extention to other domains such as object detection is considered as future work which also shows the effectiveness of this architecture.

## Requirements

I just updated the code to the most recent version of Pytorch (=pytorch 1.0.1) with python 3.7.
The other required packages are: opencv, scikit-learn, torchvision, numpy, matplotlib, json/ujson and pillow.

## Run Code

The TuSimple dataset has been used for this experiment. I will try to make the train-validation split available as well as the ground truth annotations to save the user this work.
In the file Labels/Curve_parameters.json, the coefficients of the second degree polynomials are shown for the multiple lane lines in a bird's eye view perspective for the a subset of the data. (three zeros means that the lane line is not present in the image)

To run the code (training phase):

` python main.py --image_dir /path/to/image/folder --gt_dir /path/to/ground_truth/folder --end_to_end True`

Flags:
- Set flag "end_to_end" to True to regress towards the final lane line coefficients directly.
- See `python main.py --help` for more information.

The weight maps will be computed but be aware that the appearance of the weight maps is architecture dependent. Augmenting this method with a line type branch in a shared encoder setup, results in: 

![end_to_end](https://user-images.githubusercontent.com/9694230/51836593-12459400-2301-11e9-9d1b-37cbe936f8cc.gif)

## Results Egolane Detection 

| Method | Model | Area metric | Area<sup>2</sup> loss|
| --- | --- | --- | --- | 
| Segmentation | ERFNet | 1.603e-3 normalized | 2.733e-5  normalized |
| End to end | ERFNet | 1.437e-3 normalized| 1.912e-5 normalized | 
| **Gain** | ERFNet |1.66e-4\*(19.1mx38.2m)/2<sup>1</sup> = **0.06 m<sup>2</sup>** | 8.21e-6 normalized |

(<sup>1</sup> Based on 3.7 m standard lane width in the US)


## Discussion

Practical discussion for multi lane detection:

- Instance segmentation: We primarily want to focus on our differentiable least squares module from our paper. This module is compatible with whatever method you choose. See it as an extra layer of the network to make lane detection completely end-to-end. Hence, an instance segmentation method can be combined with our method.

- To detect multiple lanes more robustly, the mask in the Networks/LSQ_layer.py file can be exploited.

- Continual learning setup: A possibility is to focus first on egolanes and add more lane lines during training. This makes the task more difficult over time. This will improve the convergence, since the features of the first lane lines can help to detect the later ones.

- Pretrainig: When a high number of lane lines are desired to be detected, the supervision could be be too weak (depending on the initialization and the network). Pretraining using a few segmentation labels is good way too alleviate this problem.

- Proxy segmentation task: You could also combine our method with a proxy segmentation task in a shared encoder architecture. This can have some benefits (i.e. good initialization for the weight maps), although this makes it more complex.
