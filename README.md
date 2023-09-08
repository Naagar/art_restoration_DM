# Art restoration in camera capute images of distorted/degraded Art.
Image restoration and super resolution using pretrained and finetuned Diffusion Model.

### [COMPETITIONS @ ICETCI 2023](https://ietcint.com/user/competitions)  
### AI Art Restoration - Reviving Cultural Heritage

## Update
- [x] For Art Restoration fine-tuned model weights contact `rnfrbdsandeep@gmail.com`.
- [ ] 

## <font color="red" >   </font>

### Demo on real-world SR


For more evaluation, please refer to our [paper](https://arxiv.org/abs/2305.07015) for details.

### Demo on 4K Results

- StableSR is capable of achieving arbitrary upscaling in theory, below is a 8x example with a result beyond 4K (5120x3680).
The example image is taken from [here](https://github.com/Mikubill/sd-webui-controlnet/blob/main/tests/images/ski.jpg).


- We further directly test StableSR on AIGC and compared with several diffusion-based upscalers following the suggestions.
A 4K demo is [here](https://imgsli.com/MTc4MDg3), which is a 4x SR on the image from [here](https://github.com/pkuliyi2015/multidiffusion-upscaler-for-automatic1111).
More comparisons can be found [here](https://github.com/IceClear/StableSR/issues/2).

### Dependencies and Installation
- Pytorch == 1.12.1
- CUDA == 11.7
- pytorch-lightning==1.4.2
- xformers == 0.0.16 (Optional)
- Other required packages in `environment.yaml`
```
# git clone this repository
git clone https://github.com/IceClear/StableSR.git
cd StableSR

# Create a conda environment and activate it
conda env create --file environment.yaml
conda activate stablesr

# Install xformers
conda install xformers -c xformers/label/dev

# Install taming & clip
pip install -e git+https://github.com/CompVis/taming-transformers.git@master#egg=taming-transformers
pip install -e git+https://github.com/openai/CLIP.git@main#egg=clip
pip install -e .
```

### Running Examples

#### Train
Download the pretrained Stable Diffusion models from [[HuggingFace](https://huggingface.co/stabilityai/stable-diffusion-2-1-base)]

- Train Time-aware encoder with SFT: set the ckpt_path in config files ([Line 22](https://github.com/IceClear/StableSR/blob/main/configs/stableSRNew/v2-finetune_text_T_512.yaml#L22) and [Line 55](https://github.com/IceClear/StableSR/blob/main/configs/stableSRNew/v2-finetune_text_T_512.yaml#L55))
```
python main.py --train --base configs/stableSRNew/v2-finetune_text_T_512.yaml --gpus GPU_ID, --name NAME --scale_lr False
```

- Train CFW: set the ckpt_path in config files ([Line 6](https://github.com/IceClear/StableSR/blob/main/configs/autoencoder/autoencoder_kl_64x64x4_resi.yaml#L6)).

You need to first generate training data using the finetuned diffusion model in the first stage. The data folder should be like this:
```
CFW_trainingdata/
    └── inputs
          └── 00000001.png # LQ images, (512, 512, 3) (resize to 512x512)
          └── ...
    └── gts
          └── 00000001.png # GT images, (512, 512, 3) (512x512)
          └── ...
    └── latents
          └── 00000001.npy # Latent codes (N, 4, 64, 64) of HR images generated by the diffusion U-net, saved in .npy format.
          └── ...
    └── samples
          └── 00000001.png # The HR images generated from latent codes, just to make sure the generated latents are correct.
          └── ...
```

Then you can train CFW:
```
python main.py --train --base configs/autoencoder/autoencoder_kl_64x64x4_resi.yaml --gpus GPU_ID, --name NAME --scale_lr False
```

#### Resume

```
python main.py --train --base configs/stableSRNew/v2-finetune_text_T_512.yaml --gpus GPU_ID, --resume RESUME_PATH --scale_lr False
```

#### Test directly

Download the Diffusion and autoencoder pretrained models from [[HuggingFace](https://huggingface.co/Iceclear/StableSR/blob/main/README.md) | [Google Drive](https://drive.google.com/drive/folders/1FBkW9FtTBssM_42kOycMPE0o9U5biYCl?usp=sharing) | [OneDrive](https://entuedu-my.sharepoint.com/:f:/g/personal/jianyi001_e_ntu_edu_sg/Et5HPkgRyyxNk269f5xYCacBpZq-bggFRCDbL9imSQ5QDQ) | [OpenXLab](https://openxlab.org.cn/models/detail/Iceclear/StableSR)].
We use the same color correction scheme introduced in paper by default.
You may change ```--colorfix_type wavelet``` for better color correction.
You may also disable color correction by ```--colorfix_type nofix```

- Test on 128 --> 512: You need at least 10G GPU memory to run this script (batchsize 2 by default)
```
python scripts/sr_val_ddpm_text_T_vqganfin_old.py --config configs/stableSRNew/v2-finetune_text_T_512.yaml --ckpt CKPT_PATH --vqgan_ckpt VQGANCKPT_PATH --init-img INPUT_PATH --outdir OUT_DIR --ddpm_steps 200 --dec_w 0.5 --colorfix_type adain
```
- Test on arbitrary size w/o chop for autoencoder (for results beyond 512): The memory cost depends on your image size, but is usually above 10G.
```
python scripts/sr_val_ddpm_text_T_vqganfin_oldcanvas.py --config configs/stableSRNew/v2-finetune_text_T_512.yaml --ckpt CKPT_PATH --vqgan_ckpt VQGANCKPT_PATH --init-img INPUT_PATH --outdir OUT_DIR --ddpm_steps 200 --dec_w 0.5 --colorfix_type adain
```

- Test on arbitrary size w/ chop for autoencoder: Current default setting needs at least 18G to run, you may reduce the autoencoder tile size by setting ```--vqgantile_size``` and ```--vqgantile_stride```.
Note the min tile size is 512 and the stride should be smaller than the tile size. A smaller size may introduce more border artifacts.
```
python scripts/sr_val_ddpm_text_T_vqganfin_oldcanvas_tile.py --config configs/stableSRNew/v2-finetune_text_T_512.yaml --ckpt CKPT_PATH --vqgan_ckpt VQGANCKPT_PATH --init-img INPUT_PATH --outdir OUT_DIR --ddpm_steps 200 --dec_w 0.5 --colorfix_type adain
```

- For test on 768 model, you need to set ```--config configs/stableSRNew/v2-finetune_text_T_768v.yaml```, ```--input_size 768``` and ```--ckpt```. You can also adjust ```--tile_overlap```, ```--vqgantile_size``` and ```--vqgantile_stride``` accordingly. We did not finetune CFW.

#### Test using Replicate API
```
import replicate
model = replicate.models.get(<model_name>)
model.predict(input_image=...)
```
You may see [here](https://replicate.com/cjwbw/stablesr/api) for more information.

### Citation
If our work is useful for your research, please consider citing:

    @inproceedings{wang2023exploiting,
        author = {Wang, Jianyi and Yue, Zongsheng and Zhou, Shangchen and Chan, Kelvin CK and Loy, Chen Change},
        title = {Exploiting Diffusion Prior for Real-World Image Super-Resolution},
        booktitle = {arXiv preprint arXiv:2305.07015},
        year = {2023}
    }

### License

This project is licensed under <a rel="license" href="https://github.com/IceClear/StableSR/blob/main/LICENSE.txt">NTU S-Lab License 1.0</a>. Redistribution and use should follow this license.

### Acknowledgement

This project is based on [StableSR](https://github.com/IceClear/StableSR), [stablediffusion](https://github.com/Stability-AI/stablediffusion), [latent-diffusion](https://github.com/CompVis/latent-diffusion), [SPADE](https://github.com/NVlabs/SPADE), [mixture-of-diffusers](https://github.com/albarji/mixture-of-diffusers) and [BasicSR](https://github.com/XPixelGroup/BasicSR). Thanks for their awesome work.

### Contact
If you have any questions, please feel free to reach me out at `rnfrbdsandeep@gmail.com`.
