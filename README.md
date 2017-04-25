# Image-Colorization
This repository tries to generate vehicle images with response to user sketch via Generative Adversarial networks(GAN). It is based on https://github.com/affinelayer/pix2pix-tensorflow and https://phillipi.github.io/pix2pix. \
It is known that the training of GAN is difficult and requires a lot of tuning on hyper-parameters. Recently, it is suggested that by introducing a different loss function to train GAN based on Wasserstein distance(WGAN), the training will become much easier. The theory behind WGAN is rather difficult while the implementation is relatively simple. WGAN is attempted based on this repository(https://github.com/affinelayer/pix2pix-tensorflow) with only few lines of code modified in pix2pix.py.\
Usage:
```
python pix2pix.py --mode=train --input_dir=data/testc --output_dir=test_model --l1_weight=10 --wgan=1
```
Only little data is provided for testing purpose.
