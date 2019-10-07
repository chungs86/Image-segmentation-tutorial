#  Foundations Atlas Tutorial
<p align="center">
  <img src='images/atlas_logo.png' width=70%>
</p>


# Start Guide

**Prerequisites**

1. Docker version >18.09 (Docker installation: <a target="_blank" href="https://docs.docker.com/docker-for-mac/install/"> Mac</a>
 | <a target="_blank" href="https://docs.docker.com/docker-for-windows/install/"> Windows</a>)
2. Python >3.6 (<a target="_blank" href="https://www.anaconda.com/distribution/">Anaconda installation</a>)
3. \>5GB of free machine storage
4. The atlas_ce_installer.py file (Download after signup <a target="_blank" href="https://www.atlas.dessa.com/">here</a>)


**Steps**

See <a target="_blank" href="https://dessa-atlas-community-docs.readthedocs-hosted.com/en/latest/ce-quickstart-guide/">Atlas documentation</a>
  

<details>
  <summary>FAQ: How to upgrade an older version of Atlas?</summary>
<br>

1. Stop atlas server using `atlas-server stop`
2. Remove docker images related to Atlas in your terminal `docker images | grep atlas-ce | awk '{print $3}' | xargs docker rmi -f`
3. Remove the environment where you installed the Atlas or pip uninstall the Atlas `conda env remove -n your_env_name`

-------------------------------------------------------------------------------------------------------------------------
</details>

# Image Segmentation

This tutorial demonstrates how to make use of the features of Foundations Atlas. Note that **any machine learning job can be run in Atlas without modification.** However, with minimal changes to the code we can take advantage of Atlas features that will enable us to:

* view artifacts such as plots and tensorboard logs, alongside model performance metrics
* launch many training jobs at once
* organize model experiments more systematically


## Data and Problem

The dataset that will be used for this tutorial is the <a target="_blank" href="https://www.robots.ox.ac.uk/~vgg/data/pets/">Oxford-IIIT Pet Dataset</a>, created by Parkhi *et al*. The dataset consists of images, their corresponding labels, and pixel-wise masks. The masks are basically labels for each pixel. Each pixel is given one of three categories :

* Class 1 : Pixel belonging to the pet.
* Class 2 : Pixel bordering the pet.
* Class 3 : None of the above/ Surrounding pixel.

Download the processed data [here](https://dl-shareable.s3.amazonaws.com/train_data.npz).

<img src='images/data.png' width=70%>

## Clone the Tutorial

Clone this repository by running:
```bash
git clone https://github.com/dessa-public/Image-segmentation-tutorial.git
```
and then type `cd Image-segmentation-tutorial` in the terminal to make this your current directory.

## Start Atlas

Activate the conda environment in which Foundations Atlas is installed (by running `conda activte your_env` inside terminal). Then run `atlas-server start` in a new tab terminal. Validate that the GUI has been started by accessing it at <a target="_blank" href="http://localhost:5555/projects">http://localhost:5555/projects</a>.

## Running a job

Activate the environment in which you have foundations installed, then from inside the project directory (Image-segmentation-tutorial) run the following command:
```python
foundations submit scheduler . main.py
```
Notice that you didn't need to install any other packages to run your job because Foundations already take care of it.

#####Now you already have code reproducbility:
You can check the logs of your job by clicking the expand button on the right end of the job row in the GUI where you can check the performance of this job by checking the logs.
You can reproduce your code and results at any time later in the future. In order to recover the code corresponding to any foundations job_id, just `cd ~/.foundations/job_data/archive/your_job_id_here/artifacts` where you can find the code corresponding to a job-id in order to reproduce your results. 

Congrats! Your code is now tracked by Foundations Atlas! Let's move on to explore the magic of Atlas. 
## Full Atlas Features
The full Atlas features include: 
1. Automatic environment creation for jobs
1. Use of Custom docker images to avoid replicating download of packages
1. Experiment tracking via GUI to monitor various jobs
1. Code Reproducibility
1. Job scheduling
1. Hyperparameter search to create better ML models
1. Track job performance via job parameters and metrics
1. Save any objects such as images, audio, video corresponding to any job and view inside GUI
1. Tensorboard integration to analyze deep ML models


## How to Enable Full Atlas Features

You are provided with the following python scripts:

* main.py: a main script which prepares data, trains an U-net model, then evaluates the model on the test set.

To enable Atlas features, we only to need to make a few changes. Firstly add the following line to the line 19 of main.py:

```python
import foundations
```

## Logging Metrics and Parameters

The last line of main.py outputs the training and validation accuracy. After these statements, we will call to the function foundations.log_metric().This function takes two arguments, a key and a value. Once a job successfully completes, logged metrics for each job will be visible from the Foundations GUI. Copy the following line and replace the print statement with it.

Line 234 in main.py:

```python
foundations.log_metric('train_accuracy', float(train_acc))

foundations.log_metric('val_accuracy', float(val_acc))

```

## Saving Artifacts

We have created graphs for test samples. With Atlas, we can save any artifact to the GUI with just one line. Add the following lines to send the locally saved plot to the Atlas GUI.

Line 108 in main.py:
```python
foundations.save_artifact(f"sample_{name}.png", key=f"sample_{name}")
```
Moreover, you can save trained model by adding below

```python
foundations.save_artifact('trained_model.h5', key='trained_model')
```
to line 236 in main.py.

## TensorBoard Integration 


<a target="_blank" href="https://www.tensorflow.org/tensorboard/r1/summaries">TensorBoard</a> is a super powerful data visualization tool that makes visualizing your training extremely easy. Foundations Atlas has full TensorBoard integration. To access TensorBoard directly from the Atlas GUI, add the following line of code to start of driver.py.

Line 210 in main.py:
```python
foundations.set_tensorboard_logdir('tflogs')
```

## (Optional) Build Docker Image

The motivation of building customized image is to avoid reinstalling packages listed in the requirements.txt repeatedly. Run the following command in the terminal:
```bash
cd custom_docker_image
docker build . --tag image_seg:atlas
```
By doing this, you have created a docker image named `image_seg:atlas` on your local computer that conatins the python environment required to run this job.


## Configuration

Now, create a file in the project directory named "job.config.yaml", and copy the text from below into the file. 

You can specify project name, docker images in this configuration. 

Benefit from last `Build Docker Image` option, you have already build `image_seg:atlas`. 


```python
# Project config #
project_name: 'Image-segmentation-tutorial'
log_level: INFO

# Worker config #
# Additional definition for the worker can be found here: https://docker-py.readthedocs.io/en/stable/containers.html

num_gpus: 0

worker:
  image: image_seg:atlas # name of your customized images
  volumes:
    /local/path/to/folder/containing/data:
      bind: /data/
      mode: rw
```

Note: If you don't want to use the custom docker image, you can just comment out or just delete the whole `image` line inside `worker` section of this config file shown above. In this case, foundations will use a default docker image and will automatically create the required enviornment for the each job seperately (this may take relatively longer time if your job needs a lot of packages to be installed).

Under the `volumes` section, you will need to replace `/local/path/to/folder/containing/data` with your local absolute path of data folder so that your data can be accessed within the foundations docker container.

## Running a Hyperparameter Search

Atlas makes running multiple experiments and tracking the results of a set of hyperparameters easy. Create a new file called 'hyperparameter_search.py' and paste in the following code:

```python
import os
import numpy as np
import foundations

NUM_JOBS = 10

def generate_params():

    hyper_params = {'batch_size': int(np.random.choice([8, 16, 32, 64])),
                    'epochs': int(np.random.choice([10, 20, 30, 50])),
                    'learning_rate': np.random.choice([0.1, 0.01, 0.001, 0.0001]),
                    'decoder_neurons': [np.random.randint(16, 512), np.random.randint(16, 512),
                                        np.random.randint(16, 512), np.random.randint(16, 512)],
                    }
    return hyper_params


for job_ in range(NUM_JOBS):
    print(f"packaging job {job_}")
    hyper_params = generate_params()
    foundations.submit(scheduler_config='scheduler', job_dir='.', command='main.py', params=hyper_params,
                       stream_job_logs=False)
```

This script samples hyperparameters uniformly from pre-defined ranges, then submits jobs using those hyperparameters. For a script that exerts more control over the hyperparameter sampling, check the end of the tutorial. The job execution code is still coming from main.py; i.e. each experiment is submitted to and ran with the script.

In order to get this to work, a small modification needs to be made to main.py. In the code block where the hyperparameters are defined (indicated by the comment 'define hyperparameters'), we'll load the sampled hyperparameters instead of defining a fixed set of hyperparameters explictely.

Replace that block (line 27 - 31) with the following:
```python
hyper_params = foundations.load_parameters()
```

Now, to run the hyperparameter search, from the project directory (Image-segmentation-tutorial) simply run:
```bash
python hyperparameter_search.py
```

## Congrats!
That's it! You've completed the Foundations Atlas Tutorial. Now, you should be able to go to the <a target="_blank" href="http://localhost:5555/projects">GUI</a> and see your running and completed jobs, compare model hyperparameters and performance, as well as view artifacts and training visualizations on TensorBoard.

Do you have any thoughts or feedback for Foundations Atlas? Join the Dessa Slack community! tiny.cc/dessa
