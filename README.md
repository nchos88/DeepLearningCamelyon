# Deep Learning Pipeline for Camelyon 2016 dataset

    Weizhe Li, Weijie Chen
    

## 0 - Preparation
### 0.1 Set up deep learning environment.

- [Setup Python Environment](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/0%20-%20Preparation/Seup%20Machine%20Learning%20Environment.py)

- [Python IDE](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/0%20-%20Preparation/Python%20IDE%20(Emacs%20%2B%20Org%20%2B%20Ob-iPython))
  ![alt text](https://github.com/3dimaging/Accessory/blob/master/destopshot.png)
  
- [Package Installation for Color Normalization](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/0%20-%20Preparation/Staintools)
- [Singularity for reproducibility](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/0%20-%20Preparation/Singularity)
        


### 0.2 ASAP installation and image display

- [ASAP Installation](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/0%20-%20Preparation/ASAP%20installation%20(Ubuntu%2016.04))
- [OpenSlide Installation](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/0%20-%20Preparation/OpenSlide%20Installation)


Compare the ASAP with OpenSlide: ASAP doesn’t have detailed manual to describe its commands; OpenSlide has a much better document for its commands. ASAP has a GUI; OpenSlide doesn’t   
   
### 0.3 Mask file generation

- Mask file is the ground truth for model training. Mask file has the exact same dimensions as its corresponding WSI image.  Mask file is a binary file with normal tissue coded as ‘0’ and tumor tissue coded as ‘1’ for each corresponding pixel of WSI image. 
- [Mask file generation](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/1%20-%20WSI%20Visualization%20with%20Annotation/Mask%20Generation)
- However, the code provided by the organizer is misleading. The mask file generated by the code from their paper is all ‘0’; then another piece of code suggested generated mask file that can be open only by ASAP GUI, not by its command line and OpenSlide.
- Time consuming

WSI and Mask file: tumor_026

![alt text](https://github.com/3dimaging/Accessory/blob/master/389e2c0cd56cf3ea1356fd03a2af35cc-94048Vj0.png)

WSI and Mask file: tumor_005

![alt tex](https://github.com/3dimaging/Accessory/blob/master/49285w6c.png)

## 1 - WSI Visulization with Annotation
### Annotation Visulization

- [Annotation Visulization Over Image Base on xml file](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/1%20-%20WSI%20Visualization%20with%20Annotation/Display%20annotation%20over%20image_Based%20on%20xml%20file)

 
 <img src="https://github.com/3dimaging/Accessory/blob/master/annotation%20over%20wsi.png" width="250">

- [Annotation Visulization Over Image Base on mask file](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/1%20-%20WSI%20Visualization%20with%20Annotation/Display%20annotation%20over%20Image-Based%20on%20Mask%20file)

 
 <img src="https://github.com/3dimaging/Accessory/blob/master/annotation2.png" width="250">



## 2 - Image Preprocess

### 2.1 Image Segmentation

To reduce computation, the blank regions (no tissue) on slide will be excluded.

- Color space switch to HSV
- Tissue region segmentation (Otsu’s method of foreground segmentation)


### 2.2 Patch Extraction


#### Step 1 : Randomly extract patches (256 x 256) on the tissue region at the level of 40x
               
		Tumor slide : 1K positive and 1K negative from each slide

		Normal slide: 1K negative from each slide
            

#### Step 2 : Crop 224x224 patches and conduct image augmentation

- stain normalization (Method II)
	   
  The color variety among patches
	   
 <img src="https://github.com/3dimaging/Accessory/blob/master/color%20variety.png" width="350">

	   
  The patches before and after stain normalization
	   
 <img src="https://github.com/3dimaging/Accessory/blob/master/stain%20normalization.png" width="350">

	   
- flip
- adding color noise (Method II)
	   
 <img src="https://github.com/3dimaging/Accessory/blob/master/patch%20flip%20noise.png" width="350">

	   
	   
	    
	    

		
#### Step 3 : Image Generator

 Patches:
 ![alt text](https://github.com/3dimaging/Accessory/blob/master/patches.png)
 
 Ground Truth:
 ![alt text](https://github.com/3dimaging/Accessory/blob/master/patche%20ground%20truth.png)

## 3 - Training Neural Network
	
### 3.1	FCN

	Lambda, Normalize input (x / 255.0 - 0.5), outputs 256x256x3 
0. Convolution1, 5 x 5 kernel, stride 2, outputs 128x128x100 
1. Maxpooling1, 2 x 2 window, stride 2, outputs 64x64x100 
2. Convolution2, 5 x 5 kernel, stride 2, outputs 32x32x200 
3. Maxpooling2, 2 x 2 window, stride 2, outputs 16x16x200 
4. Convolution3, 3 x 3 kernel, stride 1, outputs 16x16x300 
5. Convolution4, 3 x 3 kernel, stride 1, outputs 16x16x300 
6. Dropout, 0.1 rate 
7. Convolution5, 1x1 kernel, stride 1, outputs 16x16x2 
8. Deconvolution, 31 x 31 kernel, stride 16, outputs 256x256x2 

-[FCN training](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/3%20-%20Training%20Neural%20Network/Model%20training%20code%20for%20fully%20convolutional%20neural%20network)

FCN prediction

<img src="https://github.com/3dimaging/Accessory/blob/master/fcn-predicion%20tumor.png" width="250">

<img src="https://github.com/3dimaging/Accessory/blob/master/fcn-prediction%202.png" width="250">

### 3.2 U-net
- [U-net training](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/3%20-%20Training%20Neural%20Network/Model%20training%20for%20unet)

Learning Curve
![alt text](https://github.com/3dimaging/Accessory/blob/master/u-net%20training%20score.png)

Prediction by trained U-net:
![alt text](https://github.com/3dimaging/Accessory/blob/master/u-net%20prediction.png)


### 3.3 GoogleNet

-- step 1: Model Training

[Training GoogleNet](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/3%20-%20Training%20Neural%20Network/Model%20training%20for%20googelnet)

- Optimization method: Stochastic gradient descent

- Weight initialization: Random sampling from a Gaussian distribution

- Batch size: 32

- Batch normalization: No

- Regularization: L2-regularization (0.0005) and 50% dropout

- Learning rate: 0.01, multiplied by 0.5 every 50,000 iterations (0.01, multiplied by 0.1 per epoch)

- Activation function: ReLu

- Loss function: Cross-entropy

- Number of training epochs/iterations: 300,000 iterations

-- step 2: Negative Mining


Extract additional training patches from false positive regions

## 4 - Prediction and Evaluation
### 4.1 Make predictions and construct heatmaps

Test images were divided into non-overlapping small patches; each patch will get a predicted image for each pixel assigned by probability.
Heatmap is a way to display the probability 

 - [Code for prediction - googlenet](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/4%20-%20Prediction%20and%20Evaluation/Prediction_googlenet.py)
 - [Code for prediction - FCN and U-net](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/4%20-%20Prediction%20and%20Evaluation/Prediction_fcn_unet.py)


Put all the patches together and get prediction for the whole slide ([code for heatmap generation based on predicted values](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/4%20-%20Prediction%20and%20Evaluation/Heatmap_generation.py)). 

#### Heatmap Examples
Heatmap for tumor_026:

![alt text](https://github.com/3dimaging/Accessory/blob/master/99262ES.png)

The overview of heatmap for tumor_026:

![alt text](https://github.com/3dimaging/Accessory/blob/master/9926DPY.png)

Comparison of predicted with ground truth for tumor_005:

![alt text](https://github.com/3dimaging/Accessory/blob/master/49285XZv.png)

#### Hard Negative Mining
 - [Code for generate index of false positive patches](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/4%20-%20Prediction%20and%20Evaluation/Generate_Index_For_False_Positive_Patches.py)
 - [Code for generate patches and its mask files](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/4%20-%20Prediction%20and%20Evaluation/Generate_False_Positive_Patches.py)
    
### 4.2a Slide-based Classification


Extracting Features for whole-slide image classification task

#### Global Features Extraction

1. The ratio between the area of metastatic regions and the tissue area.
2. The sum of all cancer metastases probailities detected in the metastasis identification task, divided by the tissue area. 
caculate them at 5 different thresholds (0.5, 0.6, 0.7, 0.8, 0.9), so the total 10 global features

#### Local Features Extraction 

Based on 2 largest metastatic candidate regions (select them based on a threshold of 0.5).

9 features were extracted from the 2 largest regions:

1. Area: the area of connected region
2. Eccentricity: The eccentricity of the ellipse that has the same second-moments as the region
3. Extend: The ratio of region area over the total bounding box area
4. Bounding box area
5. Major axis length: the length of the major axis of the ellipse that has the same normalized second central moments as the region
6. Max/mean/min intensity: The max/mean/minimum probability value in the region
7. Aspect ratio of the bounding box
8. Solidity: Ratio of region area over the surrounding convex area

#### [code for Global and Local Features Extraction](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/4%20-%20Prediction%20and%20Evaluation/Evaluation/Feature_extraction_for_random_forest.py)

### 4.2b Lesion-based Detection
   	 
- [Extract Patches near Tumor Regions](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/4%20-%20Prediction%20and%20Evaluation/Evaluation/Patches_Near_Tumor.py)

- Training model again (model-2)

- Combine Model-1 and Model-2, do prediction


### 4.3 ROC and FROC Generation

 - [Code for ROC](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/4%20-%20Prediction%20and%20Evaluation/Evaluation/Random_Forest_Training_and_ROC.py)
 - [Code for FROC](https://github.com/DIDSR/DeepLearningCamelyon/blob/master/4%20-%20Prediction%20and%20Evaluation/Evaluation/FROC_from_organizer.py)

# Teams using GoogleNet
HMS&MIT, HMS&MGH(model I), Smart Imaging(model II), Osaka University, CAMP-TUM(model II), Minsk Team, DeepCare

# References
-  Wang, D., et al.: Deep learning for identifying metastatic breast cancer
https://arxiv.org/abs/1606.05718

-  Diagnostic Assessment of Deep Learning Algorithms for Detection of Lymph Node Metastases in Women With Breast Cancer.
https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5820737/?report=reader#!po=59.4340

