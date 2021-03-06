DiracNets
=========

PyTorch code and models for *DiracNets: Training Very Deep Neural Networks Without Skip-Connections*

<https://arxiv.org/abs/1706.00388>

Networks with skip-connections like ResNet show excellent performance in image recognition benchmarks, but do not benefit from increased depth, we are thus still interested in learning __actually__ deep representations, and the benefits they could bring. We propose a simple weight parameterization, which improves training of deep plain (without skip-connections) networks, and allows training plain networks with hundreds of layers. Accuracy of our proposed DiracNets is close to Wide ResNet (although DiracNets need more parameters to achieve it), and we are able to outperform ResNet-1000 with plain DiracNet with only 34 layers. Also, the proposed Dirac weight parameterization can be folded into one filter for inference, leading to easily interpretable VGG-like network.

<img src=http://imagine.enpc.fr/~zagoruys/delta-circles.svg>


## TL;DR

In a nutshell, Dirac parameterization is a sum of filters and scaled Dirac delta function:

```
conv2d(x, alpha * delta + W)
```

Here is simplified PyTorch-like pseudocode for the function we use to train plain DiracNets (with weight normalization):

```python
def dirac_conv2d(input, W, alpha, beta)
    return F.conv2d(input, alpha * dirac(W) + beta * normalize(W))
```

where `alpha` and `beta` are scaling scalars, and `normalize` does l_2 normalization over each feature plane.

We also use NCReLU (negative CReLU) nonlinearity:

```python
def ncrelu(x):
    return torch.cat([x.clamp(min=0), x.clamp(max=0)], dim=1)
```


## Code

Code structure:

├── [README.md](README.md)          # this file<br>
├── [diracconv.py](diracconv.py)    # modular DiracConv definitions<br>
├── [test.py](test.py)              # unit tests<br>
├── [diracnet-export.ipynb](diracnet-export.ipynb) # ImageNet pretrained models<br>
├── [diracnet.py](diracnet.py)      # functional model definitions<br>
└── [train.py](train.py)            # CIFAR and ImageNet training code<br>

### Requirements

First install [PyTorch](https://pytorch.org), then install [torchnet](https://github.com/pytorch/tnt):

```
pip install git+https://github.com/pytorch/tnt.git@master
```

Install [OpenCV](https://opencv.org) with Python bindings (e.g. `conda install -c menpo opencv3`), and `torchvision`
with OpenCV transforms:

```
pip install git+https://github.com/szagoruyko/vision.git@opencv
```

Finally, install other Python packages:

```
pip install -r requirements.txt
```

To train DiracNet-34-2 on CIFAR do:

```
python train.py --save ./logs/diracnets_$RANDOM$RANDOM --depth 34 --width 2
```

To train DiracNet-18 on ImageNet do:

```bash
python train.py --dataroot ~/ILSVRC2012/ --dataset ImageNet --depth 18 --save ./logs/diracnet_$RANDOM$RANDOM \
                --batchSize 256 --epoch_step [30,60,90] --epochs 100 --weightDecay 0.0001 --lr_decay_ratio 0.1
```


### nn.Module code

We provide `DiracConv1d`, `DiracConv2d`, `DiracConv3d`, which work like `nn.Conv1d`, `nn.Conv2d`, `nn.Conv3d`, but have Dirac-parametrization inside (our training code doesn't use these modules though).


### Pretrained models

We fold batch normalization and Dirac parameterization into `F.conv2d` `weight` and `bias` tensors for simplicity. Resulting models are as simple as VGG or AlexNet, having only nonlinearity+conv2d as a basic block.

See [diracnets.ipynb](diracnets.ipynb) for functional and modular model definitions.

We provide printout of DiracNet-18-0.75 sequential model for reference:

```
Sequential (
  (conv): Conv2d(3, 48, kernel_size=(7, 7), stride=(2, 2), padding=(3, 3), bias=False)
  (max_pool0): MaxPool2d (size=(3, 3), stride=(2, 2), padding=(1, 1), dilation=(1, 1))
  (group0.block0.bn): Affine(48)
  (group0.block0.ncrelu): NCReLU()
  (group0.block0.conv): Conv2d(96, 48, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
  (group0.block1.ncrelu): NCReLU()
  (group0.block1.conv): Conv2d(96, 48, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
  (group0.block2.ncrelu): NCReLU()
  (group0.block2.conv): Conv2d(96, 48, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
  (group0.block3.ncrelu): NCReLU()
  (group0.block3.conv): Conv2d(96, 48, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1), bias=False)
  (max_pool1): MaxPool2d (size=(2, 2), stride=(2, 2), dilation=(1, 1))
  (group1.block0.bn): Affine(48)
  (group1.block0.ncrelu): NCReLU()
  (group1.block0.conv): Conv2d(96, 96, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
  (group1.block1.ncrelu): NCReLU()
  (group1.block1.conv): Conv2d(192, 96, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
  (group1.block2.ncrelu): NCReLU()
  (group1.block2.conv): Conv2d(192, 96, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
  (group1.block3.ncrelu): NCReLU()
  (group1.block3.conv): Conv2d(192, 96, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1), bias=False)
  (max_pool2): MaxPool2d (size=(2, 2), stride=(2, 2), dilation=(1, 1))
  (group2.block0.bn): Affine(96)
  (group2.block0.ncrelu): NCReLU()
  (group2.block0.conv): Conv2d(192, 192, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
  (group2.block1.ncrelu): NCReLU()
  (group2.block1.conv): Conv2d(384, 192, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
  (group2.block2.ncrelu): NCReLU()
  (group2.block2.conv): Conv2d(384, 192, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
  (group2.block3.ncrelu): NCReLU()
  (group2.block3.conv): Conv2d(384, 192, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1), bias=False)
  (max_pool3): MaxPool2d (size=(2, 2), stride=(2, 2), dilation=(1, 1))
  (group3.block0.bn): Affine(192)
  (group3.block0.ncrelu): NCReLU()
  (group3.block0.conv): Conv2d(384, 384, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
  (group3.block1.ncrelu): NCReLU()
  (group3.block1.conv): Conv2d(768, 384, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
  (group3.block2.ncrelu): NCReLU()
  (group3.block2.conv): Conv2d(768, 384, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
  (group3.block3.ncrelu): NCReLU()
  (group3.block3.conv): Conv2d(768, 384, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
  (relu): ReLU ()
  (avg_pool): AvgPool2d ()
  (view): Flatten()
  (fc): Linear (384 -> 1000)
)
```

Pretrained weights for this model: <https://www.dropbox.com/s/0z8ko3aqbuk2hov/diracnet-18-0.75-export.hkl?dl=0>

We plan to add more pretrained models later.

## Bibtex

```
@inproceedings{Zagoruyko2017diracnets,
    author = {Sergey Zagoruyko and Nikos Komodakis},
    title = {DiracNets: Training Very Deep Neural Networks Without Skip-Connections},
    url = {https://arxiv.org/abs/1706.00388},
    year = {2017}}
```
