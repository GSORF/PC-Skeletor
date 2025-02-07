# PC Skeletor - Point Cloud Skeletonization <img align="right" height="250" src="img/PCSkeletor.png">

<a href="https://img.shields.io/pypi/pyversions/pc_skeletor"><img alt="PyPI - Python Version" src="https://img.shields.io/pypi/pyversions/pc_skeletor"></a>
<a href="https://github.com/meyerls/PC-Skeletor/actions"><img alt="GitHub Workflow Status" src="https://img.shields.io/github/workflow/status/meyerls/PC-Skeletor/Python%20package"></a>
<a href="https://github.com/meyerls/PC-Skeletor/blob/main/LICENSE"><img alt="license" src="https://img.shields.io/github/license/meyerls/PC-Skeletor"></a>

## About

**PC Skeletor** is a Python library for extracting a 1d skeleton from 3d point clouds using the algorithm from
[Laplacian-Based Contraction](https://taiya.github.io/pubs/cao2010cloudcontr.pdf) or
[L1-Medial Skeleton](https://www.cs.sfu.ca/~haoz/pubs/huang_sig13_l1skel.pdf) (Not yet implemented!).


<p align="center">
    <img width="70%" src="img/tree_sceleton_small.gif">
</p>

## ⚡️ Quick Start

### Installation

First install [Python](https://www.python.org/downloads/) Version 3.7 or higher. The python package can be installed via
from [PyPi](https://pypi.org/project/pc-skeletor/) using pip.

 ````bash
pip install pc-skeletor
 ````

### Basic Usage

Below is the code to execute the skeletonization algorithm with a downloaded example point cloud. Additionally to the
extraction an animation with the original point cloud and the skeleton is created and exported as a gif.

````python
import pc_skeletor
from pc_skeletor import skeletor
from pc_skeletor.download import Dataset
import numpy as np

# Download test tree dataset
downloader = Dataset()
downloader.download_tree_dataset()

# Init tree skeletonizer
skeletor = skeletor.Skeletonizer(point_cloud=downloader.file_path,
                                 down_sample=0.01,
                                 debug=False)
laplacian_config = {"MAX_LAPLACE_CONTRACTION_WEIGHT": 1024,
                    "MAX_POSITIONAL_WEIGHT": 1024,
                    "INIT_LAPLACIAN_SCALE": 100}
skeleton, graph, skeleton_graph = skeletor.extract(method='Laplacian', config=laplacian_config)
# save results
skeletor.save(result_folder='./data/')
# Make animation of original point cloud and skeleton
skeletor.animate(init_rot=np.asarray([[1, 0, 0],
                                      [0, 0, 1],
                                      [0, 1, 0]]), steps=200, out='./data/')
# Interactive visualization
skeletor.visualize()
````

## Ω Parametrization

### Laplacian-Based Contraction

Laplacian-Based Contraction is a method based on contraction of point clouds to extract curve skeletons by iteratively
contracting the point cloud. This method is robust to missing data and noise. Additionally no prior knowledge on the
topology of the object has to be made.

The contraction is computed by iteratively solving the linear system

```math
\begin{bmatrix}
\mathbf{W_L} \mathbf{L}\\
\mathbf{W_H}
\end{bmatrix} \mathbf{P}^{'} =
\begin{bmatrix}
\mathbf{0}\\
\mathbf{W_H} \mathbf{P}
\end{bmatrix}
```

obtained from [Kin-Chung Au et al.](http://graphics.csie.ncku.edu.tw/Skeleton/skeleton-paperfinal.pdf)
$\mathbf{L}$ is a $n \times n$
[Laplacian Matrix](http://rodolphe-vaillant.fr/entry/101/definition-laplacian-matrix-for-triangle-meshes)
with cotangent weights. The Laplacian of a point cloud (Laplace-Beltrami Operator) can be used to compute the [mean
curvature Vector](http://www.cs.cmu.edu/~kmcrane/Projects/DDG/paper.pdf)(p. 88 & p. 100). $\mathbf{P}$ is the original
point cloud, $\mathbf{P}^{'}$ a contracted point cloud and $\mathbf{W_L}$ and $\mathbf{W_H}$ are diagonal weight
matrices balancing the contraction and attraction forces. During the contraction the point clouds get thinner and
thinner until the solution converges. Afterwards the contracted point cloud aka. skeleton is sampled using
farthest-point method.

To archive good contraction result and avoid over- and under-contraction it is necessary to initialize and update the
weights $\mathbf{W_L}$ and $\mathbf{W_H}$. Therefore the initial values and the maximum values for both diagonal
weighting matrices have to adjusted to archive good results.

#### Hyperparameters

* **MAX_LAPLACE_CONTRACTION_WEIGHT** [default: 1024]: indicates the maximum contraction factor. If the skeleton is not
  shrunk to a line and has a net-like structure, this factor should be increased.
* **MAX_POSITIONAL_WEIGHT** [default: 1024]: indicates the maximum positional or attraction factor. If the skeleton has
  lost its shape, this factor can be increased to maintain the shape.
* **INIT_LAPLACIAN_SCALE** [default: 100]: this parameter gives the initial amplification of the laplace weights. At the
  beginning the laplace weights are calculated over $\frac{1}{\alpha \sum_i m_{ii}}$. $\alpha$ is the
  amplification factor and $m_{ii}$ are the diagonal elements of the
  computed [Mass Matrix](http://rodolphe-vaillant.fr/entry/101/definition-laplacian-matrix-for-triangle-meshes)
  $\mathbf{M}$. A large $\alpha$ might lead to a fast convergence with a loss of the original shape. This value should
  be adjusted regarding the size of the original point cloud.

### L1-Medial Skeleton

tbd

## 📖 Literature and Code used for implementation

#### Laplacian based contraction

Our implementation
of [Point Cloud Skeletons via Laplacian-Based Contraction](https://taiya.github.io/pubs/cao2010cloudcontr.pdf) is a
python reimplementation of the original [Matlab code](https://github.com/taiya/cloudcontr).

#### L1-Medial Skeleton of Point Cloud (NOT YET IMPLEMENTED!)

Paper: https://www.cs.sfu.ca/~haoz/pubs/huang_sig13_l1skel.pdf

Source Code: https://github.com/HongqiangWei/L1-Skeleton

````bash
@ARTICLE{Huang2013,
title = {L1-Medial Skeleton of Point Cloud},
author = H. Huang and S. Wu and D. Cohen-Or and M. Gong and H. Zhang and G. Li and B.Chen},
journal = {ACM Transactions on Graphics},
volume = {32},
issue = {4},
pages = {65:1--65:8},
year = {2013}
}
````

#### Robust Laplacian for Point Clouds

Computation of the discrete laplacian operator
via [Nonmanifold Laplace](http://www.cs.cmu.edu/~kmcrane/Projects/NonmanifoldLaplace/NonmanifoldLaplace.pdf) can be
found in the [robust-laplacians-py](https://github.com/nmwsharp/robust-laplacians-py) repository.

## Limitation / Improvements

- [ ] Implement L1-Medial skeleton of point cloud
- [ ] Add graph extraction
- [ ] Test code