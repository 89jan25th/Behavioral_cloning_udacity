## **Behavioral Cloning** 
---
**Behavioral Cloning Project**

The goals / steps of this project are the following:
* Use the simulator to collect data of good driving behavior
* Build, a convolution neural network in Keras that predicts steering angles from images
* Train and validate the model with a training and validation set
* Test that the model successfully drives around track one without leaving the road
* Summarize the results with a written report


[//]: # (Image References)

[image1]: ./examples/flowchart.png "Flow chart"
[image2]: ./examples/example0.jpg "Resize example"
[image3]: ./examples/falling.png "About to fall"
[image4]: https://devblogs.nvidia.com/parallelforall/wp-content/uploads/2016/08/cnn-architecture-624x890.png "model" 

---
### Files Submitted & Code Quality

#### 1. Submission includes all required files and can be used to run the simulator in autonomous mode

My project has the following files:
* model.py containing the script to create and train the model
* drive.py for driving the car in autonomous mode -- mostly the same from the given package and I just added the resize feature.
* model.h5 containing a trained convolution neural network
* readme.md
* vidoe.mp4 is the result clip from my final codes
* preprocess.py for resizing loaded images

#### 2. Submission includes functional code
Using the Udacity provided simulator and my drive.py file, the car can be driven autonomously around the track by executing 
```sh
python drive.py model.h5  
```  
  
My code is fully functional, it makes the simulator car drive the basic track endlessly without intervention.  
[video.mp4, youtube link, please open it in a new tab](https://youtu.be/DMREgenZhPQ)  
I used the generator as instructed, I only added a resize function from preprocess.py I created.  
The function starts at line 46 in model.py

I resized the image with open cv's resize function. I crop it to 50~155 in y-axis ad didn't touch x-axis.  
The code is in preprocess.py  
![alt text][image2]  
*Cropped and resized image*  

#### 3. Submission code is usable and readable

My model's flow chart is presented below. 

![alt text][image1] 



### Model Architecture and Training Strategy

#### 1. An appropriate model architecture has been employed

I used Nvidia end-to-end CNN model and it works. In order to use the exact same architecture, I had to resize the camera images to 70 x 204 which is not 66 x 200(This is what nvidia team used). It was because MaxPooling2D() loses a pixel from width or height if they are an odd number. Thus I did some trial-and-error tasks to find the exact image size.

#### 2. Attempts to reduce overfitting in the model

I used two dropout layers with factor = 0.5. The layer's detailed position is at the below table. (refer to 2. Final Model Architecture)
I tried 0 ~ 0.5, and two 0.5 layers produced the best result.


#### 3. Model parameter tuning

I used the adam optimiser.  
Batch size was set to 64 after trying 32, 64, and 128(the results are not different much between them.)

#### 4. Appropriate training data

I made three training set (driving_log.csv, driving_log2.csv, and driving_log3.csv)  
- driving_log.csv : 2 laps clockwise, 2 laps counter-clockwise, midium quality of driving.
- driving_log2.csv : 1 lap clockwise, 1 lap counter-clockwise, poor quality of driving. (stay in track though)
- driving_log3.csv : 2 laps clockwise, 2 lap counter-clockwise + additional cornering samples, good quality of driving.

I made dataset with different quality of driving to see what makes good dataset.  
After 23 runs, I concluded that the poor quality set doesn't help at all. it was just garbage in garbage out.  
Also I found because it is 'behavioral cloning,' the most important is to make a GOOD training set to clone.  
I only used log and log3 for my report.

### Model Architecture and Training Strategy

#### 1. Solution Design Approach

The overall architecture is the same as NVIDIA's. I wanted to implement NVIDIA's architecture.  
  
The data is normalized with keras Lambda layer.  
~~~~
model.add(Lambda(lambda x: x / 127.5 - 1.0, input_shape = (70, 204, 3)))
~~~~

**Dropout was one of the most important factor.** Without this, the car frequently failed to pass the corner.  
![alt text][image3]  
*About to fall!*  

I did about 20 runs where the car veer off from the corner, and tried different combinations of  
- training set (log1, log2, log3)
- multicamera correction factor (0 ~ 1.0)
- Dropout (0 ~ 0.5)   

Then I finally found my model work when  
- log 1 + log 3 + no multicamera correction + two dropout layers (0.5)


#### 2. Final Model Architecture
I used Nvidia's architecture  
![alt text][image4]  
*source: https://devblogs.nvidia.com/parallelforall/deep-learning-self-driving-cars/*

My architecture and pramaters are  

| Layer (type)                     | Output Shape         | Param #    | Connected to |
|:---------------------:|:---------------------:|:-------------:|:-----------:| 
| lambda_1 (Lambda)                | (None, 70, 204, 3)   | 0          | lambda_input_1[0][0] |
| convolution2d_1 (Convolution2D)  | (None, 66, 200, 24)  | 1824       | lambda_1[0][0] |
| maxpooling2d_1 (MaxPooling2D)    | (None, 33, 100, 24)  | 0          | convolution2d_1[0][0] |
| activation_1 (Activation)        | (None, 33, 100, 24)  | 0          | maxpooling2d_1[0][0] |
| convolution2d_2 (Convolution2D)  | (None, 29, 96, 36)   | 21636      | activation_1[0][0] |
| maxpooling2d_2 (MaxPooling2D)    | (None, 14, 48, 36)   | 0          | convolution2d_2[0][0] |
| activation_2 (Activation)        | (None, 14, 48, 36)   | 0          | maxpooling2d_2[0][0] |
| convolution2d_3 (Convolution2D)  | (None, 10, 44, 48)   | 43248      | activation_2[0][0] |
| maxpooling2d_3 (MaxPooling2D)    | (None, 5, 22, 48)    | 0          | convolution2d_3[0][0] |
| activation_3 (Activation)        | (None, 5, 22, 48)    | 0          | maxpooling2d_3[0][0] |
| convolution2d_4 (Convolution2D)  | (None, 3, 20, 64)    | 27712      | activation_3[0][0] |
| activation_4 (Activation)        | (None, 3, 20, 64)    | 0          | convolution2d_4[0][0] |
| convolution2d_5 (Convolution2D)  | (None, 1, 18, 64)    | 36928      | activation_4[0][0] |
| activation_5 (Activation)        | (None, 1, 18, 64)    | 0          | convolution2d_5[0][0] |
| dropout_1 (Dropout)              | (None, 1, 18, 64)    | 0          | activation_5[0][0] |
| flatten_1 (Flatten)              | (None, 1152)         | 0          | dropout_1[0][0] |
| dense_1 (Dense)                  | (None, 1164)         | 1342092    | flatten_1[0][0] |
| dense_2 (Dense)                  | (None, 100)          | 116500     | dense_1[0][0] |
| dense_3 (Dense)                  | (None, 50)           | 5050       | dense_2[0][0] |
| dense_4 (Dense)                  | (None, 10)           | 510        | dense_3[0][0] |
| dense_5 (Dense)                  | (None, 1)            | 11         | dense_4[0][0] |

Total params: 1,595,511  
Trainable params: 1,595,511  
Non-trainable params: 0  


#### 3. Creation of the Training Set & Training Process

Most of the points are already made above, but I'd like to make a summary.

To create good behavior to clone, I tried to...  
- drive the center lane.
- drive slightly off from the center but towards the inside just before the corner to take a smooth turn.
- make equivalent laps for each direction.
- make at least 20k images (I used 22,769 samples for training.)
  
To create a good training set, I tried to...
- collect three different qualities of driving, poor, normal, and good.
- augment flipped images, and it works as a significant factor.
- normalize images.

To create a good trainig process, I tried...
- to run with or without multi camera correction assist and different factors. To my case the multi camera isn't so helpful.
- to run with or without dropout and different factors. To my case I found two layers with 0.5 factor is good enough to avoid overfitting.
