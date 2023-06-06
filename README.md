# defect-detection-using-semantic-segmentation

This Github Repository adds support for **Defect Detection** Using [EDGE-AI-Model-Maker](https://github.com/TexasInstruments/edgeai-modelmaker) tool for TI Embedded Processor.

## Defect Detection in Casting Product Image Data
Dataset is of casting manufacturing product.\
Casting is a manufacturing process in which a liquid material is usually poured into a mould, which contains a hollow cavity of the desired shape, and then allowed to solidify.\
The Dataset is taken from the **Kaggle**. [link to dataset](https://www.kaggle.com/datasets/ravirajsinh45/real-life-industrial-dataset-of-casting-product)

This project Uses **Semantic segmentation** for detecting the defect.\
The pixel corresponding to Defective area will be colored. 


## Setting Up Model Maker

The [edgeai-modelmaker](https://github.com/TexasInstruments/edgeai-modelmaker) is an end-to-end model development tool that contains dataset handling, model training and compilation.\
**This is a commandline tool and requires a Linux PC.**\
Go to [this repository](https://github.com/TexasInstruments/edgeai-modelmaker) to know the Edgeai-modelmaker in detail and install it on Linux PC.

:o: Note: 
1. I tried to run the model in the Virtual Box, it didn't worked for me. This needs a CUDA enabled Linux PC to run.
2. I installed the model from the github and tried to run to run but it didn't worked for me. So i installed it from TI's BigBucket Page. [edgeai-model-maker Bitbucket](https://bitbucket.itg.ti.com/projects/EDGEAI-ALGO/repos/edgeai-modelmaker/browse). If you see the `setup_all.sh` file you will get to know that another 4 repository are cloned to run the model-maker.  
a. edgeai-torchvision\
b. edgeai-edgeai-mmdetection\
c. edgeai-benchmark\
d. edgeai-tidl-tools

3. I used some specific branch of all 4 repository above because some branches was not working for me. The repo and their branches which i cloned are listed below. Just change these branches in `setup_all.sh` script.\
a. edgeai-torchvision -> \
b. edgeai-edgeai-mmdetection\
c. edgeai-benchmark\
d. edgeai-tidl-tools

Follow each instructions in the Readme.md file to set up the Model Maker.

## Annotating Data
The annotation file must be in **COCO JSON** format.

**If you are using Label Studio take note of following:**  

- For Semantic Segmentation we can export data in COCO-JSON format only if we use polygon tool to annotate data.  
- :o: Note: We can't export data in COCO JSON format if we use brush tool for semantic segmentation annotation in Label Studio. 

**How to use label studio**

- Make an account in label-studio by signing up.
- Create a new project in Label Studio and give it a name in the "Project Name" tab.
- In the Data Import tab upload your images. (You can upload multiple times if your images are located in various folders in the source location).
- Go to Setting at top right.
- In the tab named "Labelling Setup ->" click on "Browse Template". choose "Semantic Segmentation with Mask" .
- Remove the existing "Choices" and add your Label Choices (Object Types) that you would like to annotate. Clip on Save.
- Now the "project page" is shown with list of images and their previews.
- Now click on an image listed to go to the "Labelling" page.
- Do not forget to click "Submit" before moving on to the next image. The annotations done for an image is saved only when "Submit" is clicked.
- After annotating the required images, go back to the "project page", by clicking ont he project name displayed on top. From this page we can export the annotation.
- Export the annotation in COCO-JSON. 

## Semantic segmentation Dataset Format
- The annotated json file and images must be under a suitable folder with the dataset name.
- Under the folder with dataset name, the following folders must exist:
1. there must be an "images" folder containing the images
2. there must be an annotations folder containing the annotation json file with the name given below.

```
edgeai-modelmaker/data/downloads/datasets/dataset_name
                             |
                             |--images
                             |     |-- the image files should be here
                             |
                             |--annotations
                                   |--instances.json
```

Once data have been annotated , exported in COCO-JSON format , and been placed the data in above format, its time to start the training and compile the model.

## Training and Compilation

Make sure you have activated the python virtual environment. By typing  `pyenv activate py36` .\
\
**Setting up configuration file.**
- Go the `edgeai-modelmaker/config_segmentation.yaml` to set up the configuration file.
- In yaml file under `common` change the `target_device` name according to your device.
- Under `dataset` change the `annotation_perfix` with the name of annotation file in `edgeai-modelmaker/data/downloads/datasets/dataset_name/annotations`. For example if the name of your annotation file is "abcd.json". Then update `annotation_prefix:'abcd'`
- `dataset_name` : You can give any name of your choice.
- `input_data_path: ` Here give path to the dataset. `./data/downloads/datasets/dataset_name`

- Under `training` tune the parameters.
- `num_gpu` is the number's of GPU you will be using for the training.

- Then finally under `compilation` tune parameter according to your need. You can add `calibration_frames` and `calibration_frames` also.
```
compilation:
    # enable/disable compilation
    enable: True #False
    tensor_bits: 8 #16 #32
    calibration_frames: 10
    calibration_iterations: 10
```

After Setting up the configuration file go to `edgeai-modelmaker` directory in terminal and enter the following command. and hit Enter.
```
./run_modelmaker.sh <target_device> config_segmentation.yaml
```

The training and compilation will take a good amount of time.

The Compiled model will be saved to `edgeai-modelmaker/data/projects/dataset_name`

## Deployment on the Board
