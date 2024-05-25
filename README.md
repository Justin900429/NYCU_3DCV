# README 📖

Welcome to the final project repository for the "3D Computer Vision" course at NYCU! This project utilizes techniques from [GaussianEditor](https://arxiv.org/abs/2311.14521) to experiment with 3D scene editing.

## A. Installation 🛠️

This project is tested on **Ubuntu 22.04** with **CUDA 11.8**. Follow these steps to set up your environment:

```shell
# Set up the environment with conda
conda env create -f environment.yaml
conda activate 3dcv_final

# Download weights for Wonder3D and DPT
cd GaussianEditor
bash download_wonder3d.sh
mkdir -p .cache/dpt
gdown 1Jrh-bRnJEjyMCS7f-WsaFlccfPjJPPHI -O ./cache/dpt
```

## B. Preparing Dataset 🎥

### B-1. Download Videos

Access our dataset via Google Drive:
- [Link 1](https://drive.google.com/file/d/11rxX2uWAcH4LKjtvq2zpUIfofpWybNYe/view?usp=drive_link)
- [Link 2](https://drive.google.com/file/d/1v1g1wYLPhPfpK3pXnwT7FQRD1UAE-zbX/view?usp=drive_link)

Or download directly using:

```shell
gdown 11rxX2uWAcH4LKjtvq2zpUIfofpWybNYe
gdown 1v1g1wYLPhPfpK3pXnwT7FQRD1UAE-zbX
```

### B-2. Videos to Frames

Extract frames from videos at the specified frame rates using `ffmpeg`:

```shell
# For the first video at 15 FPS
mkdir -p desk/input
ffmpeg -i IMG_1991.MOV -vf fps=15 desk/input/img%5d.png

# For the second video at 30 FPS
mkdir -p human/input
ffmpeg -i IMG_1992.MOV -vf fps=30 human/input/img%5d.png
```

## C. Scene Reconstruction 🏗️

### C-1. Obtaining Point Clouds

Ensure you have [colmap](https://colmap.github.io/) and [ImageMagick](https://imagemagick.org/script/download.php) installed. Execute the following:

```shell
# Resize images in the 'desk' directory to conserve resources
python convert.py -s ../desk --resize

# Convert images in the 'human' directory
python Converter -s ../human
```

The following structure will be created in the `desk` and `human` folders:

```plaintext
📦 desk or human
┣ 📂 images
┃ ┗ 📜 *.png
┣ 📂 sparse/0
┃ ┣ 📜 cameras.bin
┃ ┣ 📜 images.bin
┃ ┗ 📜 points3D.bin
┗ 📂 (other auxiliary files)
```

### C-2. Constructing Gaussian Splatting

Run the Gaussian splatting training script:

```shell
cd gaussian_splatting
OAR_JOB_ID=desk python train.py -s desk -r 2  # Downsample to save GPU resources
OAR_JOB_ID=human python train.py -s human
```

Find the training results in `output/desk` and `output/human`.

## D. Gaussian Editor 🖌️

Edit 3D scenes after generating point clouds:

```shell
cd GaussianEditor

# Desk scene
python webui.py --gs ../gaussian-splatting/output/desk/point_cloud/iteration_30000/point_cloud.ply --colmap_dir ../gaussian-splatting/desk

# Human scene
python webui.py --gs ../gaussian-splatting/output/human/point_cloud/iteration_30000/point_cloud.ply --colmap_dir ../gaussian-splatting/human
```

For detailed guidance on using the webUI, refer to [this guide](https://github.com/buaacyw/GaussianEditor/blob/master/docs/webui.md).

### D-1. Object Deleting

Use specific prompts to delete objects:

- For the remote control, use the prompt `remote control`.
- For the human, use `man with orange t-shirt`.

### D-2. Object Adding

Add objects by generating and refining inpainting results:

- **Apple**: Use `a red apple on the table` followed by `make it a realistic

 apple`.
- **Teddy Bear**: Use `a teddy bear on the ground` followed by `make it a realistic teddy bear`.

### D-3. Object Editing

Edit objects using Gaussian tracing:

- **Remote Control**: Enable SAM and add SAM points to the object. Start tracing with the prompt `Make it a Miro painting`.
- **Human Clothes**: Similarly, enable SAM for the clothes and use the prompt `Turn his shirt out of leather`.
