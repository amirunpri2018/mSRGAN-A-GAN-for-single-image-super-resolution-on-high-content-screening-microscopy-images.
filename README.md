# mSRGAN - A Generative Adversarial Network for single image super-resolution in high content screening microscopy images.


This is the first work (to the best of my knowledge), utilizing GAN's for upscaling (4x) high content screening microscopy images and optimized for perceptual quality. Inspired by Christian et al.'s [SRGAN], a generative adversarial network, mSRGAN, is proposed for super-resolution with a perceptual loss function consisting of the weighted sum of adversarial loss, mean squared error and content loss. The objective of this implementation is to learn an end to end mapping between the low/ high-resolution microscopy images and optimize the upscaled image for quantitative metrics as well as perceptual quality.

This project was presented as a part of my masters thesis at KTH and performed at The BioImage Informatics group in [Scilifelab], Sweden.



<p align="center"><img src="https://github.com/Saurabh23/Single-Image-Super-resolution-for-high-content-screening-images-using-Deep-Learning/blob/master/thesis_scripts/prelim_results/gif22.gif" height="200" width="342" /></p>

# Abstract

Image Super resolution is a widely-studied problem in computer vision, where the objective is to convert a low-resolution image to a high resolution image. Conventional methods for achieving super-resolution such as image priors, interpolation, sparse coding require a lot of pre/post processing and optimization. Recently, deep learning methods such as convolutional neural networks and generative adversarial networks are being used to perform super-resolution with results competitive to the state of the art but none of them have been used on microscopy images. In this thesis, a generative adversarial network, mSRGAN, is proposed for super resolution with a perceptual loss function consisting of a adversarial loss, mean squared error and content loss. The objective of our implementation is to learn an end to end mapping between the low / high resolution images and optimize the upscaled image for quantitative metrics as well as perceptual quality. We  then compare our results with the current state of the art methods in super resolution, conduct a proof of concept segmentation study to show that super resolved images can be used as a effective pre processing step before segmentation and validate the findings statistically.

# Motivation:

  - **High content screening image acquisition errors** : Apart from suffering the usual challenges in image acquisition (optical distortions, motion blur, noise etc.), H.C.S images are also prone to a host of domain specific challenges (Photobleaching, Cross talk, Phototoxicity, Uneven illumination, Color/Contrast errors etc.) which might further degrade the quality of the images acquired. 

<p align="center"><img src="https://github.com/Saurabh23/mSRGAN-A-GAN-for-single-image-super-resolution-on-high-content-screening-microscopy-images./blob/master/thesis_scripts/images/micro.PNG" height="290" width="450" /></p>


 
  - **Inefficiency of the traditional pixel wise Mean squared error (MSE)**: M.S.E has its share of flaws for generating images, and images produced by MSE do not correlate well with the human visual system. M.S.E overly penalizes larger errors, while is more forgiving to the small errors ignoring the underlying structure of the image. M.S.E tends to have more local minima which make it challenging to reach convergence towards a better local minimum. Consequently, the most common metric to quantitatively measure the image quality, p.s.n.r, corresponds poorly to a human's perception of an image quality.
  
  - **Feature transferability between distant source and target domains in CNN's**: Transferability of features from convnets is inversely proportional to the distance between source and target tasks, ([Bengio et al.], [Azizpour et al.]). Hence, directly using [SRGAN] (which minimises feature representations from a VGG19 trained on Imagenet database of natural images for the content loss) on microscopic images is not a good idea. I instead train a miniVGG19 from scratch to classify protein subcellular localisations in microscopy images and use the corresponding features stored to minimise the content loss.

<p align="center"><img src="https://github.com/Saurabh23/mSRGAN-A-GAN-for-single-image-super-resolution-on-high-content-screening-microscopy-images./blob/master/thesis_scripts/images/dist.JPG" height="300" width="750" /></p>



# Solution:

I use the same architecture as SRGAN with the exception that 

**1]** Instead of using feature representations from a pre-trained VGG19 model on Imagenet for minimising the content loss, I train a miniVGG19 from scratch on microscopic images to classify protein sub cellular localizations across 13 classes, and minimise the subsequent feature representations stored .

**2]** Perceptual loss function used is the weighted sum of MSE, content loss and adversarial loss. Weight parameters (alpha, beta) to adjust the importance given to MSE and content loss respectively.

<p align="center"><img src="https://github.com/Saurabh23/mSRGAN-A-GAN-for-single-image-super-resolution-on-high-content-screening-microscopy-images./blob/master/thesis_scripts/images/loss.JPG" height="350" width="750" /></p>



## mSRGAN Architecture

<p align="center"><img src="https://github.com/Saurabh23/mSRGAN-A-GAN-for-single-image-super-resolution-on-high-content-screening-microscopy-images./blob/master/thesis_scripts/images/srgan.jpeg" height="580" width="750" /></p>

# Dataset info

I used the fluorescence microscopy images from the Human Protein atlas database which are a part of The CYTO 2017 image analysis challenge. The image data provided in this challenge were generated by the Cell Atlas part of the Human Protein Atlas database. The images visualize immunostaining of human protein, and the goal of the challenge was to identify subcellular protein localisations to major organelles. The images were acquired using Leica SP5 confocal microscopes with 63x/1.2 NA oil objective and Nyquist sampling rate in 4 fluorescence channels. Each field of view consists of 4 images, which are as follows 
- DAPI staining of the nucleus - Blue
- Antibody based staining of microtubules - Red
- Endoplasmic reticulum - Yellow
- Protein localizations - Green
- The major13 dataset  part of sub-challenge 2 of the CYTO 2017 competition which consists of 20,000 fields of view containing multi-label data for 13 protein localization's is used for training and Images from hold out test sets consisting of 1216  fields of view are chosen for performing validation.


## Data processing

The dataset consists of separate .tiff images for individual stains, and I merge these images into a single RGB image with the channels assigned as:
- R - Antibody-based staining of microtubules
- G - Protein localizations
- B - DAPI staining of the nucleus
- The yellow channel representing Endoplasmic reticulum stained images is discarded.

The resulting merged RGB images are then converted to .png which ensures lossless compression and finally resized to 96x96x3.
The 96x96x3 images are then downscaled using bicubic interpolation by a factor of 4, and the resulting 24x24x3 images are used as the input to the GAN.


# Dependency

- Python 3
- Tensorflow 1.1
- OpenCV

# Usage

## 1. Train the miniVGG19 from scratch to classify protein sub cellular localisations across 13 classes 

Download the CYTO2017 dataset images [here]


Train the miniVGG19 by:

```
$ python mSRGAN/vgg19/train.py
```

The trained miniVGG19 model will be stored in "vgg19/backup". You can download my trained [minivgg19 model] 

## II. Train the mSRGAN model.

Use the same CYTO2017 dataset 

Train with:

```
$ python mSRGAN/srgan/train.py
```

The results will be stored in "src/result" and the model will be stored in "src/backup". You can download my trained [mSRGAN model]

# Results

<p align="center"><img src="https://github.com/Saurabh23/mSRGAN-A-GAN-for-single-image-super-resolution-on-high-content-screening-microscopy-images./blob/master/thesis_scripts/images/result1.JPG" height="650" width="680" /></p>

<p align="center"><img src="https://github.com/Saurabh23/mSRGAN-A-GAN-for-single-image-super-resolution-on-high-content-screening-microscopy-images./blob/master/thesis_scripts/images/Capture2.JPG" height="650" width="650" /></p>
<p align="center"><img src="https://github.com/Saurabh23/mSRGAN-A-GAN-for-single-image-super-resolution-on-high-content-screening-microscopy-images./blob/master/thesis_scripts/images/18.JPG" height="250" width="600" /></p>
<p align="center"><img src="https://github.com/Saurabh23/mSRGAN-A-GAN-for-single-image-super-resolution-on-high-content-screening-microscopy-images./blob/master/thesis_scripts/images/16.JPG" height="250" width="590" /></p>
<p align="center"><img src="https://github.com/Saurabh23/mSRGAN-A-GAN-for-single-image-super-resolution-on-high-content-screening-microscopy-images./blob/master/thesis_scripts/images/17.JPG" height="250" width="600" /></p>

# Losses Visualised

The losses for the generator and discriminator (After numerous attempts and hacks :D) 

<p align="center"><img src="https://github.com/Saurabh23/mSRGAN-A-GAN-for-single-image-super-resolution-on-high-content-screening-microscopy-images./blob/master/thesis_scripts/images/g1.JPG" height="550" width="750" /></p>

# Sources:

1. [Photo-Realistic Single Image Super-Resolution Using a Generative Adversarial Network]
2. [Perceptual Losses for Real-Time Style Transfer and Super-Resolution]
3. [Generative Adversarial Networks: An Overview]
4. [github.com/tadax/srgan]
5. [https://github.com/sjchoi86/advanced-tensorflow]

  [Photo-Realistic Single Image Super-Resolution Using a Generative Adversarial Network]: <https://arxiv.org/abs/1609.04802>
  [Perceptual Losses for Real-Time Style Transfer and Super-Resolution]: <https://arxiv.org/abs/1603.08155>
  [Generative Adversarial Networks: An Overview]: <https://arxiv.org/abs/1710.07035>
  [github.com/tadax/srgan]: <https://github.com/tadax/srgan>
  [github.com/sjchoi86/advanced-tensorflow]: <https://github.com/sjchoi86/advanced-tensorflow>
  [https://github.com/ruiann/SRGAN]: <[https://github.com/ruiann/SRGAN]>
  [SRGAN]: <https://arxiv.org/abs/1609.04802>
  [Bengio et al.]: <https://arxiv.org/abs/1411.1792>
  [Azizpour et al.]: <https://arxiv.org/abs/1406.5774>
  [Scilifelab]: <https://www.scilifelab.se/>
  [here]: <https://www.dropbox.com/sh/5yx2wnb42grikb1/AACj99tJKW6pqmpK_elJXKlxa?dl=0>
  [minivgg19 model]: <https://www.dropbox.com/sh/pntavt13gnkiihb/AADDSFgbEFthrfmXnOxM-E4ga?dl=0>
  [mSRGAN model]: <https://www.dropbox.com/sh/nxka4zc0debz87i/AACbRoaFfz5auL83SdSYUYzSa?dl=0>
