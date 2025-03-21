## this is a reproduction of learn seg from traj
this original code is Guess what move
## reproduct report:

this have been test in pytorch 2.6, python 3.12 and cuda 12 
```bash
python3.12 -m venv venv
source venv/bin/activate 
```
```bash
pip install torch torchvision torchaudio
pip install kornia jupyter tensorboard timm einops scikit-learn scikit-image openexr-python tqdm fontconfig
pip install cvbase opencv-python wandb
```

to install detectron2 from source, we need to use this command, don`t use commands list 
```bash
pip install --no-build-isolation 'git+https://github.com/facebookresearch/detectron2.git'
```

#### Data Preparation

Datasets should be placed under `data/<dataset_name>`, e.g. `data/DAVIS2016`.

* For video segmentation we follow the dataset preparation steps of [MotionGrouping](https://github.com/charigyang/motiongrouping).
* For image segmentation we follow the dataset preparation steps of [unsupervised-image-segmentation](https://github.com/lukemelas/unsupervised-image-segmentation).

#### code for generate cotracker
```python
import os
import torch
import numpy as np
import cv2
from base64 import b64encode
from IPython.display import HTML
from cotracker.predictor import CoTrackerPredictor
from tqdm import tqdm
def read_images_from_folder(folder_path):
    images = []
    for filename in sorted(os.listdir(folder_path)):
        img_path = os.path.join(folder_path, filename)
        img = cv2.imread(img_path)
        if img is not None:
            images.append(img)
    return np.array(images)

def show_video(video_path):
    video_file = open(video_path, "r+b").read()
    video_url = f"data:video/mp4;base64,{b64encode(video_file).decode()}"
    return HTML(f"""<video width="640" height="480" autoplay loop controls><source src="{video_url}"></video>""")

def process_videos(input_dir, output_dir, model):
    if not os.path.exists(output_dir):
        os.makedirs(output_dir)

    for subdir in tqdm(os.listdir(input_dir)):
        subdir_path = os.path.join(input_dir, subdir)
        if os.path.isdir(subdir_path):
            video = read_images_from_folder(subdir_path)
            video = torch.from_numpy(video).permute(0, 3, 1, 2)[None].float()

            if torch.cuda.is_available():
                model = model.cuda()
                video = video.cuda()

            pred_tracks, pred_visibility = model(video, grid_size=30)
            print(f"Processed {subdir}: {pred_tracks.shape}, {pred_visibility.shape}")

            output_path_tracks = os.path.join(output_dir, f"{subdir}_tracks.npy")
            np.save(output_path_tracks, pred_tracks.cpu().numpy())
            output_path_visibility = os.path.join(output_dir, f"{subdir}_visibility.npy")
            np.save(output_path_visibility, pred_visibility.cpu().numpy())

model = CoTrackerPredictor(
    checkpoint=os.path.join(
        '/workspace/co-tracker/checkpoints/scaled_offline.pth'
    )
)

input_dir = '/workspace/guess-what-moves/DAVIS2016/JPEGImages/480p/'
output_dir = '/workspace/guess-what-moves/DAVIS2016/Traj/480p/'

process_videos(input_dir, output_dir, model)

```

### Running

#### Training

Experiments are controlled through a mix of config files and command line arguments. See config files and [`src/config.py`](src/config.py) for a list of all available options.

```bash
python main.py
```
Run the above commands in [`src`](src) folder.

#### Evaluation

Evaluation scripts are provided as [`eval-vid_segmentation.ipynb`](src/eval-vid_segmentation.ipynb) and [`eval-img_segmentation.ipynb`](src/eval-img_segmentation.ipynb) notebooks.


# above is deprecated, don`t run code above 

#### [Subhabrata Choudhury*](https://subhabratachoudhury.com/), [Laurynas Karazija*](https://karazijal.github.io), [Iro Laina](http://campar.in.tum.de/Main/IroLaina), [Andrea Vedaldi](https://www.robots.ox.ac.uk/~vedaldi/), [Christian Rupprecht](https://chrirupp.github.io/)
### [![ProjectPage](https://img.shields.io/badge/-Project%20Page-magenta.svg?style=for-the-badge&color=white&labelColor=magenta)](https://www.robots.ox.ac.uk/~vgg/research/gwm/) [![Conference](https://img.shields.io/badge/BMVC%20Spotlight-2022-purple.svg?style=for-the-badge&color=f1e3ff&labelColor=purple)](https://bmvc2022.org/programme/papers/#554-guess-what-moves-unsupervised-video-and-image-segmentation-by-anticipating-motion)    [![arXiv](https://img.shields.io/badge/arXiv-2205.07844-b31b1b.svg?style=for-the-badge&logo=arXiv)](https://arxiv.org/abs/2205.07844)



### Abstract:
<sup> Motion, measured via optical flow, provides a powerful cue to discover and learn objects in images and videos. However, compared to using appearance, it has some blind spots, such as the fact that objects become invisible if they do not move. In this work, we propose an approach that combines the strengths of motion-based and appearance-based segmentation. We propose to supervise an image segmentation network with the pretext task of predicting regions that are likely to contain simple motion patterns, and thus likely to correspond to objects. As the model only uses a single image as input, we can apply it in two settings: unsupervised video segmentation, and unsupervised image segmentation. We achieve state-of-the-art results for videos, and demonstrate the viability of our approach on still images containing novel objects. Additionally we experiment with different motion models and optical flow backbones and find the method to be robust to these change. </sup>



### Online Demo


[<img alt="Gradio Demo" width="280px" src="https://huggingface.co/spaces/subhc/Guess-What-Moves/resolve/main/gwm_gradio.png" />](https://huggingface.co/spaces/subhc/Guess-What-Moves)

Try out the web demo at [![Hugging Face Spaces](https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Spaces-blue)](https://huggingface.co/spaces/subhc/Guess-What-Moves)


### Getting Started

#### Requirements

Create and name a conda environment of your choosing, e.g. `gwm`:
```bash
conda create -n gwm python=3.8
conda activate gwm
```
then install the requirements using this one liner:
```bash
conda install -y pytorch==1.12.0 torchvision==0.13.0 torchaudio==0.12.0 cudatoolkit=11.3 -c pytorch && \
conda install -y kornia jupyter tensorboard timm einops scikit-learn scikit-image openexr-python tqdm gcc_linux-64=11 gxx_linux-64=11 fontconfig -c conda-forge && \
yes | pip install cvbase opencv-python wandb && \
yes | python -m pip install 'git+https://github.com/facebookresearch/detectron2.git'
```






### Checkpoints
See [`checkpoints`](checkpoints) folder for available checkpoints.


### Acknowledgements

This repository builds on [MaskFormer](https://github.com/facebookresearch/MaskFormer), [MotionGrouping](https://github.com/charigyang/motiongrouping), [unsupervised-image-segmentation](https://github.com/lukemelas/unsupervised-image-segmentation), and [dino-vit-features](https://github.com/ShirAmir/dino-vit-features).

### Citation   
```
@inproceedings{choudhury+karazija22gwm, 
    author    = {Choudhury, Subhabrata and Karazija, Laurynas and Laina, Iro and Vedaldi, Andrea and Rupprecht, Christian}, 
    booktitle = {British Machine Vision Conference (BMVC)}, 
    title     = {{G}uess {W}hat {M}oves: {U}nsupervised {V}ideo and {I}mage {S}egmentation by {A}nticipating {M}otion}, 
    year      = {2022}, 
}
```   
