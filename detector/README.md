## Traffic Light Detector and Classifier

This folder is a Part of the CarND Capstone project.

* First part of the project is to Detect the traffic light on the incoming image.
* Second part fo the project is to Classify the traffic light color on the detected traffic light.

## Traffic Light Detector
We used the UNet model
![UNet](https://raw.githubusercontent.com/zhixuhao/unet/master/img/u-net-architecture.png)
And the impletementation of this can be found in folder tf_detector
### Training and predicting
We labled some our own dataset and using the Bosch dataset as well.
![Data](https://raw.githubusercontent.com/Aitical/CarND-Capstone/master/imgs/data.jpg)
Label
![Label](https://raw.githubusercontent.com/Aitical/CarND-Capstone/master/imgs/data_mask.jpg)
## Traffic light Classifier
We designed a simple Network structor with two convolutional layer.
![Net](https://raw.githubusercontent.com/Aitical/CarND-Capstone/master/imgs/cls.png)
### Training and predicting
We cut some traffice lights to train our own classifier

The predicting images
sim
![Red](https://raw.githubusercontent.com/Aitical/CarND-Capstone/master/imgs/Red.png)
carla
![Greed](https://raw.githubusercontent.com/Aitical/CarND-Capstone/master/imgs/Green.png)
