한국어 버전은 여기로 → [README-ko.md](README-ko.md)
# How to use
## Clone this repository
```sh
git clone https://github.com/ahnhs2k/vits.git
cd vits
```
This repository is a fork of ouor/vits, modified to support CUDA 12.8 (cu128)
and PyTorch ≥ 2.x, required for RTX 50xx (Blackwell) GPUs.

## Choose cleaners

- Fill "text_cleaners" in config.json
- Initialy "text_cleaners" is set to 'korean_cleaners'. To use alternative cleaners, revise with following step.
- Edit text/symbols.py
- Remove unnecessary imports from text/cleaners.py
## Create virtual environment

Windows

```sh
python -m venv .venv
.\.venv\Scripts\activate
```

Linux / WSL

```sh
uv venv --python 3.10
source .venv/bin/activate
```

## Install pytorch
```sh
pip install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cu128
```
## Install requirements
```sh
pip install -r requirements.txt
```

`requirements.txt` does NOT include PyTorch.
Make sure PyTorch is installed before running this command.

If error occurs while install requirements, Install [visual studio build tools](https://visualstudio.microsoft.com/downloads/?q=build+tools) and try again.
## Build monotonic alignment search
```sh
cd monotonic_align
mkdir monotonic_align
python setup.py build_ext --inplace
cd ..
```

Windows native Python may fail to build this module.
WSL2 or Linux environment is strongly recommended.

## Create datasets

All wav files must match sampling_rate in config.json
(Recommended: 22050Hz / mono / PCM_16)

### Single speaker
"n_speakers" should be 0 in config.json
```
path/to/XXX.wav|transcript
```
- Example
```
dataset/001.wav|こんにちは。
```
### Mutiple speakers
Speaker id should start from 0 
```
path/to/XXX.wav|speaker id|transcript
```
- Example
```
dataset/001.wav|0|こんにちは。
```
## Preprocess

This step is OPTIONAL.
- If your text is already normalized
- And "cleaned_text": true is set in config.json
You can skip preprocess.py

If you need random pick from full filelist..
```sh
python random_pick.py --filelist path/to/filelist.txt
```
```sh
# Single speaker
python preprocess.py --text_index 1 --filelists path/to/filelist_train.txt path/to/filelist_val.txt --text_cleaners 'korean_cleaners'

# Mutiple speakers
python preprocess.py --text_index 2 --filelists path/to/filelist_train.txt path/to/filelist_val.txt --text_cleaners 'korean_cleaners'
```
If you have done this, set "cleaned_text" to true in config.json
## Small Tips
- recommand to use pretrained model (you can get pretrained model from huggingface.co)
- If your vram is not enough (less than 40GB)
- do not train with 44100Hz. 22050Hz is good enough.
- make each dataset audio length short. (recommand to use maximum 4 seconds per audio)
## Train
```sh
# Single speaker
python train.py -c <config> -m <folder>

# Mutiple speakers
python train_ms.py -c <config> -m <folder>
```
If you want to train from pretrained model, Place 'G_0.pth' and 'D_0.pth' in destination folder before enter train command.
## Tensorboard
```sh
tensorboard --logdir checkpoints/<folder> --port 6006
```
## Inference
### Jupyter notebook
[infer.ipynb](infer.ipynb)
### Gradio web app
```sh
python server.py --config_path path/to/config.json --model_path path/to/model.pth
```

# Running in Docker

```sh
docker run -itd --gpus all --name "Container name" -e NVIDIA_DRIVER_CAPABILITIES=compute,utility -e NVIDIA_VISIBLE_DEVICES=all "Image name"
```



