# be223c - Mammogram Classification


# Introduction

Please put your code into the structure

* Remember to document your code in Docstring manner

* Remember to update your code running instruction here in your section


# Data Preprocessing - Yannan

1. Data Cleaning
	- Functions and methods
		- load_data: to load Athena data for this project
		- crop_image: to crop the background of an image
		- image_preprocessing: to preprocess images
		- load_implant_list: to load implant list from csv
		- create_train_data: to generate data for training
	- Description of the algorithms
		- A list of names of images with implants have been pre-identified and will be used to exclude images with implants. When matching the images with the labels, images with implants will be removed from the image list.
		- After removing images with implants, the image list needs to be further processed by removing low quality images, which have also been pre-identified and saved to a list. 
		- All functions have been saved to a library called utils_athena. 
	- Expected output
		- The expected output of this algorithm is a list of preprocessed images (no images with implants and no low quality images) that are ready for using as training data for this project. 
	- Running instructions
		- First, open run_athena.py saved in be223c/Model_Training/Data_Preprocess/. 
		- Second, import utils_athena and glob.
		- Third, set paths for importing files and saving results.
		- Fourth, run load data, load image list, and create train data blocks subsequently.
		- Finally, remove the low quality images in the low_quality_list.

2. Pectoral Muscle Removal
	- Functions and methods
		- create_train_data: to create traning data
		- rotate: to rotate an image
		- rotate_images: to rotate two lists of images
		- flip: to flip two lists of images
		- unet: to build u-net model
		- plot_acc_los: to plot loss and accuracy during training and validation
 		- read_test_images: to read in testing images
		- test_model: to test the model
		- print_test_result: to print out an example of test result
		- read_test_results: to read in test results	
		- mean_iou: to calculate Jaccard index
		- post_processing: to post-process result images
		- plot_post_process_results: to plot the results after post-processing
	- Description of the algorithms
		- This approach trains a u-net model to help segment pectoral muscles on digital mammograms.
		- It has four components: training, testing, evaluation, and post-processing. A classification model is trained and saved to the specified path. Fifty images are used as testing images and the predicted masks are generated for them. Jaccard index is used to evluate the performance of the model by comparing the similarity of the predicted mask and the groud truth (pre-defined). Highly unstatisfactory masks will undergo the post-processing procedure to improve the result. 
		- All functions have been saved to a library called utils_library. 
	- Expected output
		- The best model (model.h5) during training should be saved to the specified directory and used to generate predicted masks. 
	- Running instructions
		- First, open run_muscle.py saved in be223c/Model_Training/Data_Preprocess/. 
		- Second, import utils_muscle, glob, numpy, os, tensorflow, keras, and sklearn.
		- Third, set paths for importing files and saving results.
		- Fourth, run training, testing, evaluation, and post-processing blocks subsequently.

# Model Training - Zichen

## Requirements
* Python >= 3.5
* PyTorch >= 1.0.0
* tensorboard >= 1.7.0
* tensorboardX >= 1.2

## Code structure
    Model_Training/Training_Zichen/
    ├── agent  # Information flow controller
    │   ├── base_trainer.py  # model trainer for INBreast dataset
    │   ├── base_trainer_UCLA.py # model trainer for UCLA dataset
    │   ├── __init__.py
    │   ├── one_epoch_trainer.py
    │   ├── one_epoch_trainer_UCLA.py
    │   ├── run.sh 
    │   ├── run_UCLA.sh
    │   ├── train.py  # main script to start training
    │   └── train_UCLA.py
    ├── config  # json config file support for convenient parameter tuning.
    │   ├── config.json  # holds configuration for training
    │   └── config_UCLA.json
    ├── dataloader  # data loading
    │   ├── base_data_loaders.py
    │   ├── data_loader.py
    │   └── __init__.py
    ├── graph  # models and evaluation metrics
    │   ├── metric
    │   │   ├── __init__.py
    │   │   └── metric.py
    │   └── model
    │       ├── base_model.py
    │       ├── __init__.py
    │       └── model.py
    └── utils  # utility functions
        ├── __init__.py
        ├── logger.py
        ├── prepare_device.py
        ├── preprocess.py
        ├── util.py
        └── visualization.py
        
## Usage
Try `python train.py -c config.json` or `sh run.sh` to run code.

### Using config files
Modify the configurations in `.json` config files, then run:

  ```
  python train.py --config config.json
  ```
  
### Using Multiple GPU
Enable multi-GPU training by setting `n_gpu` argument of the config file to larger number.
First n devices will be used by default if configured to use smaller number of gpu than available.
You can specify indices of available GPUs by cuda environmental variable.
  ```
  python train.py --device 2,3 -c config.json
  ```
  This is equivalent to
  ```
  CUDA_VISIBLE_DEVICES=2,3 python train.py -c config.py
  ```


# Model Training - David

1. Dataset
	- Pytorch dataset class
		- data: data dictionary for patches and bag label by patient, where patches are stored in img key. In the form: data['imgs'], data['label'].
		- patients: patients to be used.
		- transform: the transform for the data.
		- datasetType: the type of the data, train or test.
	- Show images function
		- images: List of np.arrays compatible with plt.imshow (good for viewing patches from single image).
		- cols (Default = 1): Number of columns in figure (number of rows is set to np.ceil(n_images/float(cols))).
		- titles: List of titles corresponding to each patch. Must have the same length as titles.
	- Train test split function
		- folds: number of folds.
		- fold_index: index for folds. 
		- data: data dictionary for patches and bag label by patient, where patches are stored in img key. In the form: data['imgs'], data['label'].
		- dataset_transform: transforms train and test set.

2. Form Bags
	- Set path to images, clinical dataframe, home folder.
	- Balance clinical dataset.
	- Preprocess images.
	- Extract patches.
	- Data dictionary.
	- Data transform.
	- Folds.
	- Form bags
		- Pytorch dataset object that loads balanced mammogram dataset in bag form.
		- implementation influenced by Ilse et al. (2018).
		- target number: The desired bag label number. In this project, use 1, as bags have a positive label (1) if they contain at least 1 cancerous patch or a negative label (0) if they do not contain any cancerous patches. The aim of our model is to predict this target number (the bag label).
		- number of patches: The desired number of patches.  In this project, we extracted 50 patches of size 128 x 128 from the image after it was segmented.  These number of patches (50) are contained within one bag.The form bag function we adapted does not generally support more than 10 instances per bag; however, it works for 50.
		- variance of number patches: the desired variance for the number of patches.  In this project, we set to 0 because we have a fixed number of patches extracted from the image which are contained within one bag.
		- number of bags: The number of bags in total model, which is the combined number of training bags and testing bags.
		- seed: Set random seed.
		- train: Train.
		- num_bags_train: This contains the number of bags in the training model, which can also be interpreted as the number of images in your training set before having extracted patches.  For balanced athena dataset set to 522.
		- num_bags_test: This contains the number of bags in the test model, which can also be interpreted as the number of images in your test set before having extracted patches.  For balanced athena dataset set to 58.

3. Attention-based Deep Multiple Instance Learning Model 
	- Pytorch neural network module.
	- modified LeNet-5 model.
	- Implementation influenced by Ilse et al. (2018).
	- self.L: embedding size for bag.
	- self.D: hidden embedding size for bag features.
	- self.K: number of classes, 1 for breast cancer detection.

4. Main
	- epochs: number of epochs to train model.
	- learning rate: learning rate to train model.
	- weight decay: weight decay to train model.
	- target number: the desired bag label number. In this project, use 1, as bags have a positive label (1) if they contain at least 1 cancerous patch or a negative label (0) if they do not contain any cancerous patches. The aim of our model is to predict this target number (the bag label).
	- number_of_patches: the desired number of patches.  In this project, we extracted 50 patches of size 128 x 128 from the image after it was segmented.  These number of patches (50) are contained within one bag.
	- variance_of_number_patches: the desired variance for the number of patches.  In this project, we set to 0 because we have a fixed number of patches extracted from the image which are contained within one bag.
	- num_bags_train: This contains the number of bags in the training model, which can also be interpreted as the number of images in your training set before having extracted patches.  In this project, we set to 522.
	- num_bags_test: This contains the number of bags in the test model, which can also be interpreted as the number of images in your test set before having extracted patches.  In this project, we set to 58.
	- seed: set random seed.
	- no-cuda: disables CUDA training.
		
5. How to Use
	- Run the following commands in jupyter notebook:
		- %run -i 'get_dataset.py'
		- %run -i 'model_attn_mil.py'
		- %run -i 'mg_bag_loader.py'
		- %run -i 'main.py'


# Model Deployment - Harry

## 1. Tools
   Backend:Flask, Pytorch
   
   Frontend: uikit, Vue.js, JQuery.js
   
   Deployment: Gunicorn, Docker

## 2. Code structure
   ```
   flask-app/
   ├── server.py   # contains major code of flask app
   ├── wsgi.py     # main run
   ├── Dockerfile  # use this to generate docker image
   ├── model.py    # pytorch model prediction functions
   ├── requirements.txt # all required libraries
   ├── config.py   # Flask.app config
   ├── code_zichen/ # all Zichen's model source code
   ├── templates/  # all html pages
       ├── front_page.html # introduction page
       ├── index.html # initial loading page
       ├── team_member.html # list of team members
       ├── upload.html # all upload and prediction, display functions
   ├── uploads/ # for preload saved models
   ├── venv/ # virtual env for development
   ├── static/ # all the javascript library and static files, css, uikit
   
   ```
   
## 3. Build from source (Recommended)

This is recommended because the base image is from Nvidia
and it is very big (7Gb), it's faster to directly build
than transfer the image.

pull the base image from nvcr.io/nvidia/pytorch:19.01-py3

Run the cmd from terminal in flask-app folder
    
    sudo docker build -t flask-app .
    
The Dockerfile not only contains a Ubuntu system with CUDA and pytorch but also
include the source code for the app.
Use the generated Docker image to run the flask app

## 4. Run the Flask app with Docker image

4.1 Download or pull image from Docker Hub.

go to hub.docker.com
search for: zhanghaoyue/flask-app-mammogram


4.2 Run the following code
    
    # load the docker image if not built from source
    sudo docker load -i flask-app.tar
    # run the docker image
    sudo docker run --runtime=nvidia -p 5000:5000 flask-app


## 5. How to use the webpage

There are 3 sub pages of the website. You can nagivate 
using the navbar function.

Introduction contains model introduction

Model Prediction contains the major function:

Upload: select a input Mamogram image
Predict: click this button for result

after click the result, three images will show up and a 
prediction result will be shown at the bottom.

- The first image is the original input image.

- The second image is the heatmap in grayscale

- the third image is the original image overlay with heatmap

The probability values and the label shows Cancer/No Cancer result

    
