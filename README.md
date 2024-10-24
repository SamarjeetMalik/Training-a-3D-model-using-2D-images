# Training a 3D Diffusion Model using 2D Images

### Method

![Diagram](https://geometry.cs.ucl.ac.uk/group_website/projects/2023/holodiffusion/webpage/static/figures/pipeline_small.png)

----------------------------------------------------------------------------------------------------

### Dependencies
The fastest way to get up and running is to create a new conda environment using the `environment.yaml` file provided here. 

```
<user>@<machine>:<project-dir>/holo_diffusion$ conda env create -f environment.yml
<user>@<machine>:<project-dir>/holo_diffusion$ conda activate holo_diffusion_release
(holo_diffusion_release) <user>@<machine>:<project-dir>/holo_diffusion$  
```
```
# New conda environment
conda create -n holo_diffusion_release python==3.9.15
conda activate holo_diffusion_release

# Pytorch3D:
conda install -c pytorch -c nvidia pytorch=1.13.1 torchvision pytorch-cuda=11.6
conda install -c fvcore -c conda-forge fvcore iopath
conda install -c bottler nvidiacub
conda install -c pytorch3d pytorch3d

# Configuration management:
conda install -c conda-forge hydra-core

# Miscellaneous:
conda install -c conda-forge imageio
conda install -c conda-forge accelerate
conda install -c conda-forge matplotlib plotly visdom
```

### Training
Running the training scripts involves a few of steps. They are described as follows.

#### Data
We use the [Co3Dv2](https://github.com/facebookresearch/co3d) for training the models. Please follow the steps on the Co3Dv2 repository to download the dataset. The full dataset is large and occupies **5.5TB** of space, so feel free to download only the classes that you are interested in. 
Once, downloaded the dataset should have the following directory structure
```
CO3DV2_DATASET_ROOT
    ├── <category_0>
    │   ├── <sequence_name_0>
    │   │   ├── depth_masks
    │   │   ├── depths
    │   │   ├── images
    │   │   ├── masks
    │   │   └── pointcloud.ply
    │   ├── <sequence_name_1>
    │   │   ├── depth_masks
    │   │   ├── depths
    │   │   ├── images
    │   │   ├── masks
    │   │   └── pointcloud.ply
    │   ├── ...
    │   ├── <sequence_name_N>
    │   ├── set_lists
    │       ├── set_lists_<subset_name_0>.json
    │       ├── set_lists_<subset_name_1>.json
    │       ├── ...
    │       ├── set_lists_<subset_name_M>.json
    │   ├── eval_batches
    │   │   ├── eval_batches_<subset_name_0>.json
    │   │   ├── eval_batches_<subset_name_1>.json
    │   │   ├── ...
    │   │   ├── eval_batches_<subset_name_M>.json
    │   ├── frame_annotations.jgz
    │   ├── sequence_annotations.jgz
    ├── <category_1>
    ├── ...
    ├── <category_K>
```

#### Training models
Once, the data is downloaded and properly setup, the training scripts can now be run as follows. set an environment variable named `CO3DV2_DATASET_ROOT` which points to the root directory of the Co3Dv2 dataset. 
```
(holo_diffusion_release) <user>@<machine>:<project-dir>/holo_diffusion$ export CO3DV2_DATASET_ROOT=<co3d_dataset_path>
```
With the environment variable set, use the `experiment.py` script to start the training. 
```
(holo_diffusion_release) <user>@<machine>:<project-dir>/holo_diffusion$ python experiment.py --config-name base.yaml
```
Some notes about the training pipeline:

1. The base path where the configs are stored is `configs/`, but can be overridden while running the `experiment.py` script.
2. The `base.yaml` config specifies config used for training holo_diffusion models. Feel free to create copies from it to:

    - Change the category for training.
    - Change the model size.
    - Experiment with the hyperparamters of diffusion
    - Experiment with hyperparameters of the holo-diffusion training

3. ***Always remeber** to set the `exp_dir` in the config to point to your intended output directory for the experiment. 
4. (Optional) The config `configs/unet_with_no_diffusion.yaml` can be used to run the baseline which trains a 3D Unet to do few-view reconstruction **without diffusion**. 
5. (Optional) You can also modify the `configs/unet_with_no_diffusion.yaml` configuration, by disabling the `3D_unet`, to train a few-view reconstruction model which only uses the RenderMLP to go from pooled voxel features to rendered views. 
6. **The baselines of 4. and 5. are a useful indicator to guide the expected performance from the holo-diffusion models in case you are training on a dataset other than Co3D. The visual quality of the models should be such that 5. > 4. and the quality of samples for the diffusion models is somewhere between the two. Note that the noising and denoising process for the generative modelling losses some visual quality compared to non-stochastic few-view reconstruction process.

#### Visualization
```
(holo_diffusion_release) <user>@<machine>:<project-dir>/holo_diffusion$ python -m visdom.server --hostname <visdom_server> --port <visdom_port>
```

### Sampling
Once the models are trained using the training (experiment) script, run the sample-generating script as follows to generate samples (or visualize reconstructions). 
 
#### Generating samples
Once the experiments are complete and the `exp_dirs` setup, you can generate samples using the `generate_samples.py` script. An example invocation is as follows.
```
(holo_diffusion_release) <user>@<machine>:<project-dir>/holo_diffusion$ python generate_samples.py \
    exp_dir=<experiment_path> \
    render_size=[512,512] \
    video_size=[512,512] \
    num_samples=15 \
    save_voxel_features=True
``` 
To generate the sampling animation (similar to the ones shown at the top), use the `progressive_sampling_steps_per_render` option of the `generate_samples.py` script. It controls, how many sampling steps to take before rendering a view on the circular camera trajectory. 
```
(holo_diffusion_release) <user>@<machine>:<project-dir>/holo_diffusion$ python generate_samples.py \
    exp_dir=<experiment_path> \
    render_size=[512,512] \
    video_size=[512,512] \
    num_samples=15 \
    save_voxel_features=False \
    progressive_sampling_steps_per_render=4
``` 
*(Optional)* <br>
Please note that you can similarly use the `visualize_reconstruction.py` script for visualizing the few-view reconstructions from the baseline unet model which is trained without diffusion. 

----------------------------------------------------------------------------------------------------
